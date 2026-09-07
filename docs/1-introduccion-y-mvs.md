# Introducción y MVs

Este proyecto consiste en monitorear las alertas de seguridad de un Windows Server con Active Directory instalado utilizando el SIEM Wazuh tras lanzar ataques simulados
desde Kali Linux, para posteriormente aplicar medidas de protección en el controlador de dominio. Todo ello desde un entorno virtualizado con VirtualBox.

Los objetivos son:
- Desplegar un Windows Server con Active Directory, configurando el controlador de dominio.
- Configurar un Windows 10 cliente para que pertenezca al controlador de dominio.
- Instalar Wazuh Manager en Ubuntu Server para procesar los eventos de seguridad.
- Instalar Wazuh Agent en los agentes para enviar los eventos de seguridad al Wazuh Manager.
- Instalar Sysmon en Windows Server y el cliente y Suricata únicamente en Windows Server.
- Lanzar ataques simulados desde Kali Linux hacia el Windows Server y el cliente.
- Detectar y analizar los ataques en el Wazuh Manager.
- Aplicar medidas de defensa y mitigación en el controlador de dominio.

El entorno está compuesto por las siguientes MVs:
- **Windows Server 2022 Standard:** Actúa como el controlador de dominio.
  - **CPU:** 4
  - **Memoria:** 8000 MB
  - **Disco:** 80 GB
  - **Red:** Adaptador en Red NAT con la IP fija 10.0.1.10
 
- **Ubuntu Server 24.04:** Tiene instalado Wazuh Manager para recopilar los eventos de seguridad en tiempo real y Wazuh Dashboard para proporcionar la interfaz web.
  - **CPU:** 4
  - **Memoria:** 8000 MB
  - **Disco:** 80 GB
  - **Red:**
    - Adaptador en Red NAT con la IP fija 10.0.1.11
    - Adaptador en host-only con la IP 192.168.56.x
   
- **Windows 10 Pro:** Actúa como un cliente perteneciente al controlador de dominio.
  - **CPU:** 2
  - **Memoria:** 4096 MB
  - **Disco:** 50 GB
  - **Red:** Adaptador en Red NAT con la IP fija 10.0.1.20
 
- **Kali Linux:** Actúa como atacante para lanzar ataques hacia el Windows Server.
  - **CPU:** 2
  - **Memoria:** 4096 MB
  - **Disco:** 30 GB
  - **Red:** Adaptador en Red NAT con la IP fija 10.0.1.21

 ## Mapa de red
 
<p align="center">
  <img src="/img/red-ad.png" alt="Mapa de red" width="65%">
</p>

![Siguiente: Instalación de Active Directory en Windows Server](2-instalacion-de-ad-en-windows-server.md)
