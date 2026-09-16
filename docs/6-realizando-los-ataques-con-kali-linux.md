# Realizando los ataques con Kali Linux

Una vez está el entorno listo, el siguiente paso es iniciar los ataques hacia el Windows Server y el Windows 10 cliente e identificarlos en el Wazuh Dashboard.

### Reconocimiento

Escaneo de puertos e identificación de servicios con nmap (``nmap -sV -O IP``) hacia Windows Server
  
![nmap](/img/reconocimiento.png)

Este comando nos permite descubrir los puertos abiertos de la máquina objetivo, incluyendo los servicios que les corresponde y su versión. En este caso ha
detectado servicios como Kerberos, RPC, el dominio de AD server.local, etc…

- **Eventos en Wazuh**: Suricata detectó el escaneo de Nmap y la detección del SO (Firmas ET SCAN Nmap Scripting Engine User-Agent Detected (Nmap Scripting Engine) y
  ET SCAN NMAP OS Detection Probe)
  
  ![Eventos](/img/eventos-reconocimiento.png)
  ![Eventos](/img/eventos-reconocimiento-2.png)

  Entre los detalles de las alertas, aparece la IP del Kali:
  
  ![Detalles del evento](/img/detalles-reconocimiento.png)

### Ataque de inundación ICMP (Ping Flood / DoS)

Consiste en saturar el servidor enviando miles de paquetes con un tamaño inusualmente grande (``sudo ping -f -s 1400 IP``)
  
![Comando](/img/ping-flood.png)

- **Eventos en Wazuh:** Suricata detectó los pings ICMP (Firma GPL ICMP PING *NIX)
  
  ![Eventos](/img/eventos-ping-flood.png)

### Ataque de Password Spraying

Consiste en intentar acceder o validar la contraseña correcta de una cuenta de usuario probando contraseñas. En este caso, he hecho un intento de acceso 
remoto hacia el usuario **soporte.it** del servidor. He probado una vez indicando una contraseña incorrecta y luego indicando la correcta, 
con el comando ``netexec smb IP -u "usuario" -p 'contraseña/diccionario'``
  
![Comando](/img/password-spraying.png)

- **Eventos en Wazuh:** El canal de seguridad ha detectado estos intentos de conexiones remotas, y los identifica como un posible ataque de pass-the-hash
  
  ![Eventos](/img/eventos-password-spraying.png)

- **Detalles del evento del fallo de inicio de sesión:**
  <p>
    <img src="/img/detalles-password-spraying-1.png" alt="Detalles" width="60%"> 
  </p>

- **Detalles del evento del inicio de sesión exitoso:**
  <p>
    <img src="/img/detalles-password-spraying-2.png" alt="Detalles" width="60%"> 
  </p>

### Enumeración de usuarios mediante Kerberos

Consiste en identificar los usuarios existentes en el AD probando una lista.
En este caso he usado la herramienta [Kerbrute](https://github.com/ropnop/kerbrute) (``kerbrute userenum --dc IP -d dominio users.txt``)
  
![Comando](/img/kerbrute.png)

El archivo ``users.txt`` contiene una lista de usuarios erróneos y válidos, y de esa lista ha detectado como válidos los usuarios **david.rrhh** y **maria.rrhh**

### Ataque de Kerberoasting

Consiste en hacer que un usuario válido del dominio solicite un ticket Kerberos para una cuenta de servicio (como una cuenta que 
ejecuta SQL Server o IIS) con el fin de extraer el hash de la contraseña de esa cuenta de servicio y descifrarlo. 
En este caso, he utilizado Impacket con el script ``-GetUserSPNs`` para escanear todo el AD en busca de las cuentas con SPN y pedirles un ticket Kerberos. 
En el comando he indicado una cuenta de un usuario normal del AD para solicitar un ticket de servicio. 
(``impacket-GetUserSPNs server.local/maria.rrhh:Password1234 -dc-ip 10.0.1.10 -request``)
  
![Comando](/img/kerberoasting.png)

El comando devuelve el **ticket de servicio** (cifrado con el hash de la contraseña de la cuenta de servicio), y un atacante lo usaría para obtener la contraseña de
la cuenta de servicio (en este caso sql.finanzas) mediante ataques de fuerza bruta.

Para simular este ataque de fuerza bruta, he guardado el ticket en un archivo .txt y lo he descifrado usando un archivo con contraseñas, mediante el comando
``john --wordlist=passwds.txt ticket.txt``
  
![Comando](/img/john-ticket.png)

> Como es una contraseña fácil, lo ha descifrado en segundos. En este caso la contraseña de la cuenta de servicio **sql.finanzas** es **Pass1234**

  - **Eventos en Wazuh:** Suricata ha detectado el ataque desde el tráfico de red.
    
    ![Eventos](/img/eventos-kerberoasting.png)

  - **Detalles del evento:** En los detalles aparece la cuenta usada para obtener el ticket (maria.rrhh),
    el protocolo Kerberos y la cuenta de servicio sql.finanza
    <p>
     <img src="/img/detalles-kerberoasting.png" alt="Detalles" width="60%"> 
    </p>

### Ataque de AS-REP Roasting

Este ataque va dirigido a cuentas con preautenticación de Kerberos deshabilitada. Para realizarlo, he creado un usuario llamado 
**'cuenta.rrhh'** con una contraseña débil y con las opciones **'La contraseña nunca expira'** y **'No pedir la autenticación Kerberos previa'** marcadas:
<p>
  <img src="/img/cuentarrhh.png" alt="Propiedades de la cuenta" width="60%"> 
</p>

> La vulnerabilidad está en no pedir la autenticación Kerberos previa.
  
Después de aplicar los cambios, he lanzado el comando ``impacket-GetNPUsers server.local/maria.rrhh:Password1234 -dc-ip 10.0.1.10 -request -format hashcat``
  
![Comando](/img/comando-as-rep-roasting.png)

En este comando he indicado las credenciales de una cuenta supuestamente comprometida (por ejemplo, maria.rrhh) para autenticarme, y después, el script
ejecuta una consulta automática buscando un atributo específico de Windows llamado ``userAccountControl`` para buscar el flag numérico ``DONT_REQ_PREAUTH``
(que se activa al marcar la casilla de "No requerir preautenticación"). El servidor le responde con la lista de usuarios que cumplen
esa condición, mostrando en este caso el usuario **cuenta.rrhh**

Una vez más, el atacante podría intentar descifrar el hash anterior para obtener la contraseña de los usuarios afectos.

- **Evento en Wazuh:**
  
  ![Evento](/img/evento-as-rep-roasting.png)

- **Detalles del evento:**
  <p>
    <img src="/img/detalles-as-rep-roasting.png" alt="Detalles" width="60%"> 
  </p>

> Entre los detalles se encuentra el usuario objetivo, la IP de origen, el ID del evento (4768) y el mensaje de que se solicitó un vale de autenticación Kerberos TGT.
    
### Simulación de ejecución de malware

He creado un archivo de texto en el Escritorio del cliente con una cadena de carácteres que simula un virus. 
Al guardar el archivo, inmediatamente Windows Defender lo detecta como una amenaza y lo elimina.
<p>
  <img src="/img/malware.png" alt="Malware simulado" width="60%"> 
</p>

- **Evento en Wazuh:** El canal de Windows Defender ha detectado el posible malware.
  
  ![Evento](/img/evento-malware.png)

- **Detalles del evento:** Aparece la cuenta en que sucedió y la ruta y el proceso de la amenaza:
  <p>
    <img src="/img/detalles-malware.png" alt="Detalles" width="60%"> 
  </p>

### Ejecución de comandos sospechosos

He ejecutado un comando en el cliente con la herramienta **procdump** para simular un ataque de robo de credenciales en memoria. 
Este ataque consiste en leer la memoria del proceso lsass.exe para extraer contraseñas o hashes de usuarios. Este comando se utiliza para realizar 
un volcado completo del proceso lsass.exe y guardarlo en el archivo lsass.dmp
<p>
    <img src="/img/procdump.png" alt="Comando" width="60%"> 
  </p>

- **Evento en Wazuh:** Al tratarse de un comando que interactúa con el proceso lsass.exe, Windows Defender lo detecta.
  
  ![Evento](/img/evento-procdump.png)

- **Detalles del evento:**
  <p>
    <img src="/img/detalles-procdump.png" alt="Detalles" width="60%"> 
  </p>

### Ataque Man-In-The-Middle para la captura de credenciales de usuarios

Este ataque consiste en montar servidores falsos (HTTP, HTTPS, DNS, SMB, etc…) en el Kali para quedarse en escucha y esperar a que alguna víctima cometa un error de resolución de nombres. En este caso, desde el cliente he intentado acceder a una ruta de red que no existe ("servidorfalso") para que envíe un Broadcast a toda la red local preguntando por ese nombre. En este punto, el Kali se hace pasar por ese servidor y responde al cliente solicitando que se autentique para así obtener las credenciales del usuario en hash NTLMv2.

Para esto, he usado el comando ``sudo responder -I eth0 -dwv``
  
![Comando](/img/responder.png)

Inicia los servidores falsos:
<p>
  <img src="/img/servidores-falsos.png" alt="Inicio de servidores falsos con responder" width="40%"> 
</p>

Y se queda en escucha:
  
![responder en escucha](/img/responder-listening.png)

En este punto, he intentado acceder a una ruta de red que no existe ("servidorfalso") desde el explorador de archivos o desde la ventana de ejecución (Windows + R):

![Ruta de red inexistente en la ventana de ejecución](/img/servidorfalso.png)

Al intentar acceder, el Kali responde al broadcast pidiendo que el usuario se autentique para acceder a ese servidor falso:
<p>
  <img src="/img/poisoned-answers.png" alt="Respuestas maliciosas enviadas a la víctima" width="80%"> 
</p>
<p>
  <img src="/img/credenciales-de-red.png" alt="Credenciales de red" width="40%"> 
</p>

Al enviar las credenciales de red en el cliente, el Kali recibe el hash NTLMv2 del usuario indicado (soporte.it):
<p>
  <img src="/img/ntlm-hash.png" alt="Hash NTLMv2-SSP" width="80%"> 
</p>

![Siguiente: Defensa y mitigación](7-defensa-y-mitigacion.md)
