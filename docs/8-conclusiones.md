# ✔️ Conclusiones

Este proyecto me ha permitido recrear un entorno basado en Active Directory con el fin de auditar vulnerabilidades en el controlador de dominio, comprender su
funcionamiento y aplicar medidas efectivas de defensa y mitigación.

En el ámbito de la monitorización, he implementado el SIEM de código abierto **Wazuh**, complementado con **Sysmon** para la detección avanzada de eventos del sistema 
y con **Suricata** para la inspección de tráfico en red. Esta combinación híbrida (HIDS/NIDS) ha facilitado la correlación de logs y la captura de vectores de ataque 
que, de manera aislada, habrían pasado desapercibidos para la auditoría nativa de Windows.

Finalmente, en la fase de defensa y mitigación, he aprovechado la capacidad de la **Respuesta Activa de Wazuh** para automatizar el bloqueo de direcciones IP atacantes
mediante el Firewall de Windows Defender tras detectar determinados ataques. Conjuntamente con esta mitigación automatizada, he aplicado medidas de bastionado
como la desactivación de protocolos de red heredados (LLMNR, NetBIOS), la exigencia de la firma SMB obligatoria y la imposición del uso de algoritmos criptográficos 
robustos basados en AES-256 para el protocolo Kerberos.

**Como conclusión final**, este proyecto ha permitido comprender el funcionamiento estructural de un entorno basado en Active Directory, 
identificar los vectores de ataque y vulnerabilidades críticas presentes en sistemas por defecto, y validar las medidas de bastionado y mitigación automatizada 
que deben aplicarse para proteger tanto el controlador de dominio como los equipos clientes de la organización.

![Siguiente: Webgrafía](9-webgrafia.md)
