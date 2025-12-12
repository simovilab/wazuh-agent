# Integración de Wazuh

Wazuh es la **Plataforma de Seguridad de Código Abierto** más utilizada del mundo, que ofrece protección unificada de **Detección y Respuesta Extendidas (XDR)** y **Gestión de Información y Eventos de Seguridad (SIEM)** para *endpoints* y cargas de trabajo en la nube.
Wazuh es concretamente una **Plataforma de Seguridad de Código Abierto (Open Source Security Platform)** que unifica dos funciones críticas de seguridad en una **arquitectura de agente y plataforma única**: la **Detección y Respuesta Extendidas (XDR)** y la **Gestión de Información y Eventos de Seguridad (SIEM)**.



### 1. Arquitectura y Funcionalidad Específica

*   **Solución Unificada XDR y SIEM:** Proporciona protección para *endpoints* y cargas de trabajo en la nube, unificando funciones que históricamente estaban separadas.
*   **Función SIEM:** Ofrece una solución integral para el **monitoreo, detección y alerta** de eventos e incidentes de seguridad.
*   **Función XDR Activa:** Proporciona a los analistas de seguridad **correlación y contexto en tiempo real**. Sus respuestas son **granulares** e incluyen la **remediación en el dispositivo** para mantener los *endpoints* limpios y operativos.

### 2. Capacidades de Protección Detalladas

Wazuh proporciona protección para centros de datos *on-premises*, nubes públicas y nubes privadas. Sus capacidades de seguridad se extienden a varias áreas clave:

| Área de Seguridad | Funcionalidades Específicas |
| :--- | :--- |
| **Seguridad de *Endpoints*** | Evaluación de Configuración, Detección de *Malware*, Monitoreo de Integridad de Archivos. |
| **Inteligencia de Amenazas** | Caza de Amenazas (*Threat Hunting*), Análisis de Datos de Registro, Detección de Vulnerabilidades. |
| **Operaciones de Seguridad** | Respuesta a Incidentes, Cumplimiento Normativo, Higiene de TI. |
| **Seguridad en la Nube** | Seguridad de Contenedores, Gestión de Postura, Protección de Cargas de Trabajo. |

### 3. Código Abierto y Adopción

*   **Adopción Global:** Wazuh es la **solución de seguridad de código abierto más utilizada del mundo**.
*   **Disponibilidad y Flexibilidad:** Está disponible **sin costo** y adopta un enfoque de código abierto que asegura transparencia y flexibilidad. Los usuarios tienen la capacidad de **modificar el código fuente** para adaptarlo a sus necesidades específicas de seguridad.
*   **Integración:** Es compatible con **APIs y soluciones de terceros** como VirusTotal, TheHive y PagerDuty, permitiendo a la plataforma funcionar como fuente y receptor de datos de seguridad.

## Monitoreo de puertos

| Puerto | Protocolo | Dirección | Descripción |
|--------|-----------|-----------|-------------|
| 1514 | TCP/UDP | Entrante | Puerto de comunicación de agentes. Los agentes establecen conexión con el servicio del servidor Wazuh en este puerto para enviar datos de eventos de seguridad [6] |
| 1515 | TCP | Entrante | Puerto de registro remoto. Permite que el gestor Wazuh acepte conexiones de nuevos agentes usando cifrado TLS [7] |
| 55000 | TCP | Entrante | API RESTful del servidor Wazuh. Puerto para todas las operaciones de gestión y consulta [8] |
| 9200 | TCP | Interna | Puerto de Wazuh Indexer (basado en OpenSearch) para indexación y almacenamiento de datos [9] |
| 5601 | TCP | Entrante | Dashboard de Wazuh para visualización web [9] |

La comunicación del agente con el gestor Wazuh requiere conectividad saliente desde el agente hacia el gestor, utilizando el puerto 1514/TCP por defecto [6].
