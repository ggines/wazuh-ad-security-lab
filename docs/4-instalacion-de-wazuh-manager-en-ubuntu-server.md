# Instalación de Wazuh Manager en Ubuntu Server

Ahora que el controlador de dominio está creado y ya está el cliente unido al dominio, el siguiente paso es instalar Wazuh,
comenzando por la instalación de Wazuh Manager en Ubuntu Server.

He seguido estos pasos:

He ejecutado el siguiente comando para iniciar el asistente de instalación de Wazuh:
``curl -sO https://packages.wazuh.com/4.14/wazuh-install.sh && sudo bash ./wazuh-install.sh -a``

![Comando](/img/comando-wazuh-1.png)

Al finalizar la instalación, aparecerán las credenciales para acceder a la interfaz web:

![Credenciales de Wazuh](/img/credenciales-wazuh.png)

En este caso, se accede a la interfaz web con el usuario ``admin`` y la contraseña ``AUna?Cspe5qiAu*GCM4k?PXxNGW2Gs.j``

Los usuarios utilizados por Wazuh se almacenan en el archivo ``wazuh-passwords.txt`` dentro de ``wazuh-install-files.tar``  
Para listarlos, se ejecuta el comando ``sudo tar -O -xvf wazuh-install-files.tar wazuh-install-files/wazuh-passwords.txt``

![Comando](/img/comando-wazuh-2.png)

Por último, he desactivado las actualizaciones de Wazuh para evitar problemas entre versiones.  
La versión de Wazuh Agent en los agentes no debe ser superior a la de Wazuh Manager.  
Para ello he ejecutado los comandos ``sed -i "s/^deb /#deb /" /etc/apt/sources.list.d/wazuh.list`` y ``apt update``

![Comando](/img/comando-wazuh-3.png)

Después de estos pasos, Wazuh Manager ya está instalado.

Para acceder a la interfaz web, he indicado la IP del adaptador de Red NAT del Ubuntu Server en el navegador (https://192.168.56.105):  

![Inicio de sesión de Wazuh](/img/wazuh-login.png)

Después de introducir las credenciales, ya tenemos acceso al dashboard:

![Wazuh Dashboard](/img/wazuh-dashboard.png)

![Siguiente: Instalación de Wazuh Agent en Windows Server y Windows 10](5-instalacion-de-wazuh-agent-en-windows-server-y-windows10.md)
