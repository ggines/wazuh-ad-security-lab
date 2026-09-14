# Defensa y mitigación

Una vez realizados los ataques anteriores, el último paso es aplicar medidas de defensa para proteger el controlador de dominio y su cliente.

### Bloqueo de la IP del atacante mediante la Respuesta Activa de Wazuh

Wazuh es capaz de bloquear direcciones IP de manera automática mediante su función de Active Response. 
En este caso, he creado una regla en el Wazuh Manager para que cuando se activen las alertas anteriores, se bloquee la IP del atacante.
Al configurar esta regla, cuando se dan las condiciones, el Wazuh Manager envía una orden automática al agente para ejecutar un script 
que añade la IP de Kali al Firewall de Windows Defender.

Para configurarlo, he seguidos estos pasos:
- He abierto el archivo de configuración de Wazuh (``C:\Program Files (x86)\ossec-agent\ossec.conf``) en los agentes y he revisado que en el bloque
  de ``active-response`` las respuestas estén activas:
  ```
  <!-- Active response -->
    <active-response>
        <disabled>no</disabled>
        <ca_store>wpk_root.pem</ca_store>
        <ca_verification>yes</ca_verification>
  </active-response>
  ```
- He abierto el archivo de configuración de Wazuh Manager (``/var/ossec/etc/ossec.conf``) en el Ubuntu Server y he revisado que exista este bloque para indicar
  que herramienta usar en la respuesta activa. En este caso, es la herramienta **netsh.exe** , la cuál viene preinstalada y se encarga de añadir
  reglas de bloqueo automáticas en el Firewall de Windows:
  ```
  <command>
    <name>netsh</name>
    <executable>netsh.exe</executable>
    <timeout_allowed>yes</timeout_allowed>
  </command>
  ````
- En el mismo archivo de configuración anterior de Wazuh Manager (``/var/ossec/etc/ossec.conf``) he definido la siguiente Respuesta Activa:
  ```
  <active-response>
    <command>netsh</command>
    <location>local</location>
    <rules_id>86601,92652,60122</rules_id>
    <timeout>600</timeout>
  </active-response>
  ```
  Las directivas de este bloquea indican:
  
  - **command:** El comando que se ejecutará en la Respuesta Activa. En este caso es el comando netsh, y se especifica con el mismo nombre indicado en el bloque anterior.
  - **location:** Indica que el bloqueo debe ejecutarse en la misma máquina que generó la alerta.
  - **rules_id:** Indica que cuando se produzcan alertas con esos IDs de reglas (en este caso alertas de Suricata, conexiones remotas o inicios de sesión
    fallidos), se bloquee la IP del atacante (*srcip*).
  - **timeout:** Indica la duración del bloqueo. En este caso 600 segundos (10 minutos).
 
- Después de guardar el archivo de configuración de Wazuh Manager (``/var/ossec/etc/ossec.conf``) he reiniciado el servicio con el comando
  ``sudo systemctl restart wazuh-manager``
  
  ![Comando](/img/restart-wazuh-manager.png)

Para comprobar que la respuesta activa funciona, he vuelto a lanzar los ataques que generan alertas con los IDs de reglas anteriores.

He lanzado el ataque de Password Spraying (Regla con ID 92652) con netexec hacia el Windows Server para probar credenciales válidas en un usuario. Como este
comando ha producido varios intentos de inicio de sesión erróneos, la alerta se generó y se activó la respuesta activa:

![Ataque de Password Spraying](/img/password-spraying-defensa.png)

En este momento Kali Linux no tiene conectividad con el Windows Server, ya que Wazuh dió la orden para ejecutar el comando **netsh** y bloquear la IP atacante:
<p>
  <img src="/img/kali-sin-ping.png" alt="Ping" width="60%">
</p>

> En este caso el bloqueo solo durará 10 minutos.

### Directiva de bloqueo de cuenta

Esta directiva es útil para dificultar ataques de Password Spraying que intentan adivinar contraseñas de usuarios.
Permite bloquear temporalmente una cuenta cuando se supera un número determinado de intentos fallidos de inicio de sesión.

Para configurarla, he ido a ``Configuración del equipo > Directivas > Configuración de Windows > Configuración de seguridad > Directivas de cuenta > Directiva de
bloqueo de cuenta``

![Directiva de bloqueo de cuenta](/img/directiva-de-bloqueo-de-cuenta.png)

- **Duración del bloqueo de cuenta:** La cuenta estará 5 minutos bloqueada. En este tiempo el usuario no podrá autenticarse ni aunque introduzca la contraseña correcta.
- **Restablecer el bloqueo de cuenta después de:** Indica el tiempo que debe pasar sin nuevos intentos fallidos para que el contador vuelva a 0.
  Por ejemplo, si tras 4 intentos fallidos el usuario espera más de 5 minutos sin volver a fallar, el contador vuelve a 0 y se dispone otra vez de 5 intentos.
- **Umbral de bloqueo de cuenta:** Tras 5 intentos de inicio de sesión fallidos se bloqueará la cuenta.

Después de aplicar las directivas, he aplicado los cambios con ``gpupdate /force``

![Comando](/img/gpupdate-force.png)

Si ahora vuelvo a ejecutar el ataque de Password Spraying para adivinar contraseñas, la cuenta de usuario que estoy atacando se bloqueará y el comando se cortará:

![Comando](/img/netexec-account-locked.png)

> Aparece el mensaje STATUS_ACCOUNT_LOCKED_OUT

Si intento acceder con el usuario afectado (maria.rrhh en este caso) desde el Windows 10 cliente, no podré acceder hasta que pasen los 5 minutos del bloqueo:
<p>
  <img src="/img/cuenta-bloqueada.png" alt="Cuenta bloqueada" width="45%">
</p>

Desde PowerShell, pueden verse las cuentas bloqueadas con el comando ``Search-ADAccount -LockedOut``

![Comando](/img/powershell-locked-out.png)

> Para esta medida de defensa, es importante indicar un umbral de bloqueo equilibrado, como de 5 a 10 intentos. Si el umbral es demasiado bajo, el atacante podría provocar una denegación de servicio al bloquear deliberadamente cuentas de usuario.

### Forzar el cifrado AES en las cuentas de usuario

Por defecto, Active Directory puede emitir tickets Kerberos utilizando un cifrado antiguo llamado RC4-HMAC.
El algoritmo de este cifrado es matemáticamente débil y muy rápido de procesar, por lo que si un atacante intercepta un ticket cifrado en RC4, las herramientas de crackeo pueden probar millones de contraseñas por segundo y descubrir claves débiles en cuestión de minutos.

Por este motivo, es importante asegurarnos que las cuentas de usuario utilicen tipos de cifrado más modernos como AES de 128 o 256 bits para Kerberos y así 
obligar a que se emitan los tickets de servicio (TGS) con un algoritmo más robusto.

Para habilitar los cifrados AES y desactivar el cifrado RC4 en la cuenta de servicio, he seguido estos pasos:

En la ventana de Usuarios y equipos de Active Directory he ido a ``Ver > Características avanzadas`` para mostrar las características avanzadas.

En las propiedades de la cuenta ``sql.finanzas`` he ido a la pestaña Editor de atributos y he seleccionado el atributo ``msDS-SupportedEncryptionTypes`` para editarlo. 
Este atributo sirve para indicar los tipos de cifrado que soporta la cuenta:
<p>
  <img src="/img/SupportedEncryptionTypes.png" alt="Atributo msDS-SupportedEncryptionTypes" width="50%">
</p>

He introducido el valor decimal 24. Este valor activa exclusivamente los cifrados AES128 y AES256, eliminando el cifrado RC4 que es inseguro. 
El funcionamiento de estos valores en el atributo msDS-SupportedEncryptionTypes es el siguiente:
| **Valor decimal** | **Valor hexadecimal** | **Tipos de cifrado habilitados** |
|:-----------------:|:---------------------:|:--------------------------------:|
|         24        |          0x18         |          AES128 + AES256         |
|         28        |          0x1C         |       RC4 + AES128 + AES256      |
|         4         |          0x4          |          Únicamente RC4          |

> En este caso, el valor 24 activa los cifrados AES128 y AES256 porque el cifrado AES
128 vale 8 bits (0x8) y el cifrado AES 256 vale 16 bits (0x10), por lo que la suma es 24.

Después de aplicar los cambios en la cuenta, he definido a nivel dominio los tipos de cifrados permitidos en Kerberos para no estar desactivando el cifrado RC4 cuenta por cuenta.

En la GPO predeterminada, he ido a ``Configuración del equipo > Directivas > Configuración de Windows > Configuración de seguridad > Directivas locales >
Opciones de seguridad > Seguridad de red: configurar tipos de cifrado permitidos para Kerberos``

<p>
  <img src="/img/directiva-cifrado-kerberos.png" alt="Directiva de tipos de cifrado permitidos para Kerberos" width="70%">
</p>

> En esta directiva he marcado únicamente las casillas AES128_HMAC_SHA1 y AES256_HMAC_SHA1 para obligar a todo el dominio a comunicarse utilizando
únicamente Kerberos AES.

Para aplicar los cambios de la GPO, he ejecutado el comando ``gpupdate /force``

![Comando](/img/gpupdate-directiva-usuario.png)

Por último, he vuelto a lanzar el comando ``impacket-GetUserSPNs server.local/maria.rrhh:Password1234 -dc-ip 10.0.1.10 -request`` para solicitar un ticket de servicio:

![Comando](/img/impacket-aes.png)

Ahora podemos comprobar que el ticket que ha devuelto empieza por ``$krb5tgs$𝟭𝟴$``
El número 18 indica que el ticket usa el cifrado AES 256.

Antes de aplicar estos cambios, el ticket empezaba por ``$krb5tgs$𝟮𝟯$``
El número 23 significa que estaba usando el cifrado RC4.

De esta manera, la cuenta sql.finanzas sigue siendo vulnerable a un ataque de Kerberoasting, pero ahora el descifrado offline requerirá mucha más potencia de
cómputo.

Aún así, es muy importante que la contraseña siga siendo lo suficientemente robusta, ya que aunque se use un cifrado más moderno, **si la contraseña es débil seguirá
siendo fácil de descifrar.**

Además, **la cuenta siempre debe tener habilitada la autenticación Kerberos previa.**


### Desactivar LLMNR y NetBIOS frente ataques Man-in-the-Middle

El objetivo de esta defensa es protegerse frente ataques MitM. Herramientas como *Responder* se usan para interceptar protocolos heredados de resolución de nombres (LLMNR, NBT-NS y mDNS) o abusar de firmas SMB no requeridas.

Para desactivar LLMNR (Link Local Multicast Name Resolution / Resolución de nombres de multidifusión local de enlace), he creado la GPO ‘Protocolos’ y he
configurado las siguientes directivas:

Desde ``Configuración del equipo > Directivas > Plantillas administrativas > Red > Cliente DNS`` he habilitado la directiva **"Desactivar resolución de nombres de
multidifusión"**

<p>
  <img src="/img/desactivar-resolucion-de-nombres-multidifusion.png" alt="Directiva" width="80%"
</p>
<p>
  <img src="/img/desactivar-resolucion-de-nombres-multidifusion-2.png" alt="Directiva" width="80%"
</p>

Para desactivar NetBIOS, desde la misma GPO, he ido a ``Configuración del equipo > Directivas > Configuración de Windows > Scripts (inicio o apagado)``
He creado un script de PowerShell que incluye el siguiente comando:
```
$RegPath =
"HKLM:SYSTEM\CurrentControlSet\services\NetBT\Parameters\Interfaces"
Get-ChildItem $RegPath | ForEach-Object {
Set-ItemProperty -Path "$RegPath\$($_.PSChildName)" -Name
"NetbiosOptions" -Value 2
}
```
> Este script **desactiva NetBIOS sobre TCP/IP** en todas las interfaces de red que encuentre el registro. El valor **2** de NetbiosOptions desactiva NetBIOS.

Una vez guardado el script, he seleccionado **Inicio** y he ido a la pestaña de Scripts de PowerShell para agregar el script:
<p>
  <img src="/img/script-inicio.png" alt="Script de inicio" width="70%">
</p>

Después de añadir el script creado anteriormente, he seleccionado la opción **'Ejecutar los scripts de PowerShell al principio'** y he aplicado los cambios:
<p>
  <img src="/img/ejecutar-scripts-al-principio.png" alt="Ejecutar los scripts de PowerShell al principio" width="40%">
</p>

Para evitar que el sistema bloqueé la ejecución de scripts al inicio, podemos permitir su ejecución mediante una directiva. 

En la misma GPO, he ido a ``Configuración del equipo > Directivas > Plantillas administrativas > Componentes de Windows > Windows PowerShell``

He habilitado la directiva **'Activar la ejecución de scripts'** y he seleccionado **'Permitir todos los scripts'**:
<p>
  <img src="/img/activar-ejecucion-de-scripts.png" alt="Activar la ejecución de scripts" width="80%">
</p>

Por último, he aplicado todos los cambios anteriores ejecutando ``gpupdate /force``

![Comando](/img/gpupdate-force.png)

Para comprobar la defensa, he vuelto a lanzar el ataque con Responder hacia el Windows 10 cliente para capturar el hash NTLM del usuario:

![Comando](/img/responder-defensa.png)

En el momento de intentar acceder a una ruta de red que no existe, en la pantalla del Kali, **ya no aparecen los protocolos LLMNR ni NetBIOS**, pero se sigue pudiendo
capturar el hash mediante el protocolo **mDNS**:
<p>
  <img src="/img/responder-defensa-2.png" alt="Listening for events" width="70%">
</p>

> El protocolo mDNS permite resolver nombres sin necesidad de un servidor DNS convencional, pero no tiene el mismo nivel de riesgo que NetBIOS o LLMNR, por lo que
habría que desactivarlo únicamente si no existe dependencia con otras aplicaciones o dispositivos del entorno.

### Forzar la firma SMB (SMB Signing)

Aún desactivando los protocolos NetBIOS y LLMNR, el atacante podría interceptar el tráfico de archivos (SMB) si no requiere una firma digital, ya que por defecto, las máquinas cliente de Windows no exigen que el tráfico de archivos esté firmado digitalmente.

Para corregirlo, en la misma GPO anterior, he ido a ``Configuración del equipo > Directivas > Configuración de Windows > Configuración de seguridad >
Directivas locales > Opciones de seguridad`` y he habilitado la directiva **"Servidor de red de Microsoft: firmar digitalmente las comunicaciones (siempre)"**
<p>
  <img src="/img/directiva-firmar-digitalmente-las-comunicaciones-servidor.png" alt="Directiva" width="70%">
</p>

En la misma ruta, también he habilitado la directiva **"Cliente de redes de Microsoft: firmar digitalmente las comunicaciones (siempre)"**
<p>
  <img src="/img/directiva-firmar-digitalmente-las-comunicaciones-cliente.png" alt="Directiva" width="70%">
</p>

> De esta manera, se exigirá que el tráfico de archivos esté firmado digitalmente.

Por último, he aplicado los cambios anteriores ejecutando ``gpupdate /force``

![Comando](/img/gpupdate-force.png)

Para comprobar que la firma SMB esté realmente activada y que el atacante no pueda realizar un ataque de Relay, he ejecutado el comando ``netexec smb 10.0.1.10
10.0.1.20 --gen-relay-list objetivos.txt`` en Kali:
<p>
  <img src="/img/netexec-gen-relay-list.png" alt="Comando" width="75%">
</p>

> Este comando comprueba el protocolo SMB en los hosts Windows Server y Windows 10 y genera una lista de posibles destinos vulnerables a SMB relay. 
En este caso, los 2 hosts aparecen con la firma SMB habilitada (signing:True), y además no generó el archivo, por lo que estos hosts no los consideró vulnerables al ataque.

![Siguiente: Conclusiones](8-conclusiones.md)
