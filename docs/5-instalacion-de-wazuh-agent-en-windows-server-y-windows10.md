# Instalación de Wazuh Agent en Windows Server y Windows 10
El siguiente paso es instalar Wazuh Agent en los agentes Windows Server y Windows 10 para recopilar los logs y los eventos de seguridad y enviarlos al Wazuh Manager del
Ubuntu Server.

La implementación de los agentes se puede hacer de diferentes maneras, pero esta vez la haré mediante la **interfaz del Wazuh Dashboard**, siguiendo estos pasos:

Desde el Wazuh Dashboard (accediendo indicando la IP del adaptador Host-Only del Ubuntu Server) he seleccionado **Deploy new agent:**  
![Agents summary](/img/agents-summary.png)

Para la implementación del agente Windows Server, he indicado los siguientes valores:
- **Sistema operativo:** Windows
  ![Deploy new agent - Sistema operativo](/img/deploy-new-agent.png)

- **Dirección IP del Wazuh Manager:** 10.0.1.11
  ![Deploy new agent - Dirección del servidor](/img/deploy-new-agent-2.png)

- **Nombre del agente:** WIN-SERVER
- **Grupo:** Por defecto
  ![Deploy new agent - Parámetros opciones](/img/deploy-new-agent-3.png)

Después de introducir los valores, se generará el comando para instalar el agente.
En este caso el comando es: ``Invoke-WebRequest -Uri
https://packages.wazuh.com/4.x/windows/wazuh-agent-4.14.6-1.msi -OutFile
$env:tmp\wazuh-agent; msiexec.exe /i $env:tmp\wazuh-agent /q
WAZUH_MANAGER='10.0.1.11' WAZUH_AGENT_NAME='WIN-SERVER'``

He ejecutado el comando en el Windows Server mediante PowerShell y he iniciado el agente con el comando ``NET START Wazuh``
![Comando](/img/wazuh-agent-command.png)

Después he hecho lo mismo pero en el Windows 10 cliente, indicando el comando
``Invoke-WebRequest -Uri
https://packages.wazuh.com/4.x/windows/wazuh-agent-4.14.6-1.msi -OutFile
$env:tmp\wazuh-agent; msiexec.exe /i $env:tmp\wazuh-agent /q
WAZUH_MANAGER='10.0.1.11' WAZUH_AGENT_NAME='WIN-CLIENT'``
![Comando](/img/wazuh-agent-command-2.png)

De esta manera, aparecerán los 2 agentes en el Wazuh Dashboard:
![Agentes en el Wazuh Dashboard](/img/wazuh-agents-dashboard.png)

![Siguiente: Configuración de los agentes para leer Sysmon y Suricata](configuracion-de-los-agentes-para-leer-sysmon-y-suricata.md)
