# Instalación de Active Directory en Windows Server

Antes de realizar la instalación del AD, he configurado la red del Windows Server de esta manera:
- **IP estática:** 10.0.1.10
- **Máscara de subred:** 255.255.255.0
- **Puerta de enlace:** 10.0.1.1
- **Servidor DNS preferido:** 10.0.1.10 (Para asegurar la resolución de nombres local)

<p>
  <img src="/img/propiedades-red-nat.png" alt="Propiedades del Adaptador Red NAT" width="40%">
</p>

<p>
  <img src="/img/ipconfig-windows.png" alt="Configuración IP de Windows" width="60%">
</p>

El siguiente paso tras configurar la red, ha sido cambiar el nombre del servidor.  
Desde el **Administrador del servidor > Servidor local > Propiedades > Nombre de equipo**
![Cambio del nombre del equipo](/img/nombre-de-equipo.png)

Después de aplicar el nombre **‘SRV-LAB’**, he reiniciado el servidor para reflejar los cambios:
![Cambio del nombre del equipo](/img/propiedades-srv-lab.png)

Después de estos ajustes en el servidor, he iniciado el proceso de instalación del AD.  
Desde el panel del administrador del servidor, he seleccionado **Agregar roles y características**.

En el tipo de instalación he seleccionado **Instalación basada en características o en roles**, ya que los roles se instalan en este servidor:
![Tipo de instalación](/img/tipo-instalacion.png)

En el servidor de destino, he seleccionado el propio servidor:
![Servidor de destino](/img/servidor-destino.png)

En los roles del servidor, he seleccionado los servicios de dominio de Active Directory con todas sus características por defecto:
![Roles de servidor](/img/roles-servidor.png)

![Características](/img/caracteristicas.png)

Por último, he permitido el reinicio automático del servidor y he confirmado la instalación:
![Confirmar selecciones de instalación](/img/confirmar-instalacion.png)

Una vez finalizada la instalación, he seleccionado **Promover este servidor a controlador de dominio** para iniciar su configuración:
![Promover este servidor a DC](/img/promover-dc.png)

He agregado un nuevo bosque indicando el nombre de dominio **server.local**
![Nuevo bosque](/img/nuevo-bosque.png)

He indicado una contraseña para el modo de restauración de servicios de directorio (DSRM), la cual permitirá recuperar, restaurar o reparar la base de datos de AD sin que los servicios del dominio estén activos, iniciando el servidor en Modo seguro de Active Directory:
![Opciones del DC](/img/opciones-dc.png)

El nombre de dominio NetBIOS es **SERVER:**
![Nombre de dominio NetBIOS](/img/netbios.png)

Después de la comprobación de los requisitos, he iniciado la instalación de los servicios de dominio de AD:
![Comprobación de requisitos previos](/img/comprobacion-requisitos-previos.png)

Después de finalizar la instalación y reiniciar el servidor, el nombre de dominio ya aparece:
<p>
  <img src="/img/server-login.png" alt="Inicio de sesión del servidor" width="60%">
</p>

En el panel aparece el AD y el servidor DNS instalados:
![Administrador del servidor](/img/panel-servidor.png)

![Siguiente: Reenviadores para el acceso a Internet](2.1-reenviadores-para-el-acceso-a-internet.md)
