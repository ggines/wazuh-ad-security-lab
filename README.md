# Bastionado de Active Directory y monitorización con Wazuh

En este proyecto se explica paso a paso la creación de un entorno basado en Active Directory para posteriormente auditar vulnerabilidades en el controlador de dominio
y aplicar medidas efectivas de defensa y mitigación.

<p align="center">
  <img src="/img/portada.png" alt="Portada">
</p>

### Tabla de contenidos

- ![Introducción y MVs](1-introduccion-y-mvs.md)
- ![Instalación de Active Directory en Windows Server](2-instalacion-de-ad-en-windows-server.md)
  - ![Reenviadores para el acceso a Internet](2.1-reenviadores-para-el-acceso-a-internet.md)
  - ![Configuración de usuarios, unidades organizativas y directivas de grupo](2.2-configuracion-de-usuarios-uo-gpo.md)
  - ![Instalación de Sysmon](2.3-instalacion-de-sysmon.md)
  - ![Instalación de Suricata](2.4-instalacion-de-suricata.md)
- ![Unión del Windows 10 cliente al dominio](3-union-del-windows10-cliente-al-dominio.md)
- ![Instalación de Wazuh Manager en Ubuntu Server](4-instalacion-de-wazuh-manager-en-ubuntu-server.md)
- ![Instalación de Wazuh Agent en Windows Server y Windows 10](5-instalacion-de-wazuh-agent-en-windows-server-y-windows10.md)
  - ![Configuración de los agentes para leer Sysmon y detectar Suricata](5.1-configuracion-de-los-agentes-para-leer-sysmon-y-suricata.md)
- ![Realizando los ataques con Kali Linux](6-realizando-los-ataques-con-kali-linux.md)
- ![Defensa y mitigación](7-defensa-y-mitigacion.md)
- ![Conclusiones](8-conclusiones.md)
- ![Webgrafía](9-webgrafia.md)
