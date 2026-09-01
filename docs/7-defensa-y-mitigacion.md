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

- He abierto el archivo de configuración de Wazuh Manager (``/var/ossec/etc/ossec.conf``) en el Ubuntu Server y he revisado que exista este bloque para indicar
  que herramienta usar en la respuesta activa. En este caso, es la herramienta **netsh.exe** , la cuál viene preinstalada y se encarga de añadir
  reglas de bloqueo automáticas en el Firewall de Windows:
  ```
  <command>
    <name>netsh</name>
    <executable>netsh.exe</executable>
    <timeout_allowed>yes</timeout_allowed>
  </command>

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
  - rules_id: Indica que cuando se produzcan alertas con esos IDs de reglas (en este caso alertas de Suricata, conexiones remotas o inicios de sesión
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
