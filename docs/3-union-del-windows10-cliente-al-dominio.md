# Unión del Windows 10 cliente al dominio
He configurado la red del Windows 10 cliente de esta manera:
- **IP estática:** 10.0.1.20
- **Máscara de subred:** 255.255.255.0
- **Puerta de enlace:** 10.0.1.1
- **Servidor DNS preferido:** 10.0.1.10 (La IP del Windows Server para que pueda resolver el dominio local)

![Configuración de la red](/img/configuracion-red.png)

Después de aplicar los cambios, he hecho un ping al servidor para comprobar que el cliente lo detecta:
![Ping servidor](/img/ping-servidor.png)

Para que el servidor tenga conectividad con el cliente, he desactivado el Firewall de Windows Defender para las redes privadas en el cliente:  
![Firewall de Windows](/img/windows-firewall.png)

Ahora ya hay conectividad:  
![Ping 10.0.1.20](/img/conectividad.png)

Para unir el cliente al dominio, he ido a **Panel de control > Sistema y seguridad > Sistema > Configuración avanzada del sistema > Nombre de equipo**  
He cambiado el nombre del equipo a WIN10-CLIENT y he indicado el dominio **server.local**
![Cambio de nombre de equipo](/img/nombre-de-equipo-cliente.png)

Al unir el dominio, hay que poner la contraseña del administrador del dominio:
![Contraseña del administrador](/img/contraseña-admin.png)

Una vez unido, el equipo se reiniciará y ya podremos iniciar sesión con los usuarios creados anteriormente en el controlador de dominio, como por ejemplo **soporte.it**:  
![Inicio de sesión con el usuario soporte.it](/img/login-soporteit.png)

![whoami](/img/soporteit-whoami.png)


![Siguiente: Instalación de Wazuh Manager en Ubuntu Server](4-instalacion-de-wazuh-manager-en-ubuntu-server.md)
