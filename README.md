# Bastionado de Active Directory y monitorización con Wazuh

En este proyecto se explica paso a paso la creación de un entorno basado en Active Directory para posteriormente auditar vulnerabilidades en el controlador de dominio
y aplicar medidas efectivas de defensa y mitigación.

<p align="center">
  <img src="/img/portada.png" alt="Portada">
</p>

### Tabla de contenidos

- ![Introducción y MVs](/docs/1-introduccion-y-mvs.md)
- ![Instalación de Active Directory en Windows Server](/docs/2-instalacion-de-ad-en-windows-server.md)
  - ![Reenviadores para el acceso a Internet](/docs/2.1-reenviadores-para-el-acceso-a-internet.md)
  - ![Configuración de usuarios, unidades organizativas y directivas de grupo](/docs/2.2-configuracion-de-usuarios-uo-gpo.md)
  - ![Instalación de Sysmon](/docs/2.3-instalacion-de-sysmon.md)
  - ![Instalación de Suricata](/docs/2.4-instalacion-de-suricata.md)
- ![Unión del Windows 10 cliente al dominio](/docs/3-union-del-windows10-cliente-al-dominio.md)
- ![Instalación de Wazuh Manager en Ubuntu Server](/docs/4-instalacion-de-wazuh-manager-en-ubuntu-server.md)
- ![Instalación de Wazuh Agent en Windows Server y Windows 10](/docs/5-instalacion-de-wazuh-agent-en-windows-server-y-windows10.md)
  - ![Configuración de los agentes para leer Sysmon y detectar Suricata](/docs/5.1-configuracion-de-los-agentes-para-leer-sysmon-y-suricata.md)
- ![Realizando los ataques con Kali Linux](/docs/6-realizando-los-ataques-con-kali-linux.md)
- ![Defensa y mitigación](/docs/7-defensa-y-mitigacion.md)
- ![Conclusiones](/docs/8-conclusiones.md)
- ![Webgrafía](/docs/9-webgrafia.md)

### Documentación en PDF 📄

La documentación también se puede descargar en ![formato PDF](/docs/Bastionado-Active-Directory-Monitorizacion-Wazuh-GuillermoGines.pdf)
