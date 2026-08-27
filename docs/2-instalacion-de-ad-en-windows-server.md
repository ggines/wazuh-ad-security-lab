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
