# Integración de sistema completo

## Resumen Ejecutivo

La integración de plataformas de monitoreo y seguridad como Wazuh y Zabbix con tecnologías modernas de desarrollo como Node.js, Django, PostgreSQL y Docker permite crear arquitecturas robustas para la gestión de infraestructura y seguridad. Este documento presenta una guía detallada sobre cómo implementar esta integración utilizando contenedores Docker y APIs RESTful.

---

## 1. Introducción

### 1.1 Contexto Tecnológico

Wazuh es una plataforma de seguridad de código abierto que proporciona capacidades de gestión de eventos e información de seguridad (SIEM), detección de intrusiones y respuesta a incidentes [1]. La arquitectura de Wazuh está diseñada para operar en entornos distribuidos, permitiendo el monitoreo centralizado de múltiples endpoints a través de agentes ligeros que recopilan datos de seguridad y los transmiten al servidor central para su análisis [1].

Por su parte, Zabbix es una solución empresarial de monitoreo de infraestructura que permite supervisar el rendimiento y disponibilidad de servidores, redes y aplicaciones mediante un sistema de agentes y checks activos/pasivos [2].

### 1.2 Arquitectura de Microservicios

La arquitectura de microservicios estructura una aplicación como una colección de servicios pequeños y débilmente acoplados, donde cada servicio es responsable de una funcionalidad específica del negocio [3]. Esta aproximación permite escalar componentes individuales, facilita el mantenimiento y mejora la resiliencia del sistema mediante el uso de contenedores Docker que encapsulan cada servicio con sus dependencias [4].

---

## 2. Componentes de la Arquitectura y Puertos de Red

### 2.1 Wazuh

#### 2.1.1 Componentes Principales y Puertos

Wazuh utiliza diversos puertos para la comunicación entre sus componentes [5]:

| Puerto | Protocolo | Dirección | Descripción |
|--------|-----------|-----------|-------------|
| 1514 | TCP/UDP | Entrante | Puerto de comunicación de agentes. Los agentes establecen conexión con el servicio del servidor Wazuh en este puerto para enviar datos de eventos de seguridad [6] |
| 1515 | TCP | Entrante | Puerto de registro remoto. Permite que el gestor Wazuh acepte conexiones de nuevos agentes usando cifrado TLS [7] |
| 55000 | TCP | Entrante | API RESTful del servidor Wazuh. Puerto para todas las operaciones de gestión y consulta [8] |
| 9200 | TCP | Interna | Puerto de Wazuh Indexer (basado en OpenSearch) para indexación y almacenamiento de datos [9] |
| 5601 | TCP | Entrante | Dashboard de Wazuh para visualización web [9] |

La comunicación del agente con el gestor Wazuh requiere conectividad saliente desde el agente hacia el gestor, utilizando el puerto 1514/TCP por defecto [6].

#### 2.1.2 API RESTful de Wazuh

La API del servidor Wazuh es una API RESTful de código abierto que permite la interacción con el gestor Wazuh desde un navegador web, herramientas de línea de comandos como cURL, o cualquier script o programa capaz de realizar solicitudes web [8]. Las capacidades de esta API incluyen:

- Gestión de agentes Wazuh
- Control y supervisión del gestor Wazuh
- Control y supervisión de clústeres
- Control y búsqueda de monitoreo de integridad de archivos (FIM)
- Información del conjunto de reglas
- Gestión de usuarios y control de acceso basado en roles (RBAC)

### 2.2 Zabbix

#### 2.2.1 Puertos y Comunicación

Zabbix utiliza dos puertos principales asignados por IANA para sus operaciones de monitoreo [10]:

| Puerto | Protocolo | Dirección | Descripción |
|--------|-----------|-----------|-------------|
| 10050 | TCP | Entrante | Puerto del agente Zabbix. Utilizado para checks pasivos donde el servidor contacta al agente [11] |
| 10051 | TCP | Entrante | Puerto del servidor/proxy Zabbix. Para checks activos donde el agente se conecta al servidor [11] |
| 8080 | TCP | Entrante | Interfaz web de Zabbix (puerto configurable) |
| 162 | UDP | Entrante | Recepción de SNMP traps (opcional) |

Cuando un servidor o proxy se conecta a un agente pasivo, el puerto TCP de destino predeterminado es 10050. Para otras conexiones de componentes Zabbix (conexiones al servidor o proxy), el puerto TCP de destino predeterminado es 10051 [10].

#### 2.2.2 Checks Activos vs Pasivos

- **Checks Pasivos**: El servidor Zabbix contacta al agente en el puerto 10050 y solicita un valor específico [12]
- **Checks Activos**: El agente se conecta al servidor en el puerto 10051 y solicita la lista de items a monitorear [12]

### 2.3 Django

Django es un framework web de alto nivel escrito en Python que promueve el desarrollo rápido y el diseño limpio [13]. En esta arquitectura, Django actúa como el backend principal para:

- Gestión de autenticación y autorización
- Exposición de APIs RESTful
- Procesamiento de datos de Wazuh y Zabbix
- Lógica de negocio centralizada

**Puertos predeterminados**:
- **8000**: Puerto de desarrollo de Django [14]
- **8000-8080**: Puertos comunes para producción con Gunicorn

### 2.4 Node.js

Node.js proporciona un entorno de ejecución para JavaScript del lado del servidor basado en el motor V8 de Chrome [15]. En esta arquitectura, Node.js puede utilizarse para:

- Microservicios de alto rendimiento
- Procesamiento de eventos en tiempo real
- WebSockets para notificaciones push
- API Gateway para agregación de servicios

**Puertos comunes**:
- **3000**: Puerto predeterminado de Express.js
- **3001-3999**: Puertos para microservicios adicionales

### 2.5 PostgreSQL

PostgreSQL es un sistema de gestión de bases de datos relacional de código abierto que ofrece conformidad ACID, soporte para tipos de datos avanzados incluyendo JSON, y capacidades de extensión [16].

**Puertos**:
- **5432**: Puerto predeterminado de PostgreSQL [17]

### 2.6 Docker

Docker permite empaquetar aplicaciones y sus dependencias en contenedores ligeros y portables que garantizan consistencia entre entornos de desarrollo, pruebas y producción [18]. Los contenedores proporcionan aislamiento de recursos y permiten el despliegue rápido de servicios complejos mediante Docker Compose [19].

---

## 3. Módulo de Integración de Wazuh

### 3.1 Configuración del Módulo Integrator

El módulo Wazuh Integrator permite a Wazuh conectarse con APIs externas y herramientas de alerta como Slack, PagerDuty, VirusTotal, Shuffle y Maltiverse, pudiendo también configurarse para conectar con otro software personalizado [20].

#### 3.1.1 Estructura de Configuración

La configuración se añade en el archivo `/var/ossec/etc/ossec.conf` del servidor Wazuh [20]:

```xml
<integration>
  <name>custom-integration</name>
  <hook_url>WEBHOOK_URL</hook_url>
  <api_key>API_KEY</api_key>
  <alert_format>json</alert_format>
  <level>10</level>
  <group>authentication_failures,web_attack</group>
  <options>{"data": "Custom data"}</options>
</integration>
```

#### 3.1.2 Filtros Opcionales

El módulo Integrator utiliza campos de filtros opcionales para determinar qué alertas deben enviarse a plataformas externas. Solo se envían las alertas que cumplen las condiciones del filtro [20]:

- **rule_id**: Filtra alertas por ID de regla (lista separada por comas)
- **level**: Filtra por nivel de alerta (0-16), enviando alertas con el nivel especificado o superior
- **group**: Filtra por grupo de reglas
- **event_location**: Filtra por origen del evento usando expresiones regulares (sregex)

### 3.2 Script de Integración Personalizada

Para crear un script de integración personalizada se deben seguir las siguientes recomendaciones [20]:

1. **Ubicación**: `/var/ossec/integrations/` en el servidor Wazuh
2. **Permisos**: 750 con propietario root:wazuh
3. **Intérprete**: Primera línea debe indicar el intérprete (ej: `#!/usr/bin/env python`)

#### 3.2.1 Parámetros del Script

El script recibe tres argumentos obligatorios [20]:
- **Argumento 1**: Ubicación del archivo de alerta (`/logs/alerts/alerts.json`)
- **Argumento 2**: API key definida en el bloque `<integration>`
- **Argumento 3**: Webhook URL definida en el bloque `<integration>`

---

## 4. Arquitectura de Integración Propuesta

### 4.1 Diagrama de Componentes

```
┌─────────────────────────────────────────────────────────────┐
│                    Infraestructura Monitoreada              │
│  (Servidores, Aplicaciones, Dispositivos de Red)           │
└──────────────┬────────────────────────┬─────────────────────┘
               │                        │
               ▼                        ▼
    ┌──────────────────┐    ┌──────────────────┐
    │  Wazuh Agents    │    │  Zabbix Agents   │
    │  Port: 1514      │    │  Port: 10050     │
    └────────┬─────────┘    └────────┬─────────┘
             │                       │
             ▼                       ▼
    ┌──────────────────┐    ┌──────────────────┐
    │  Wazuh Manager   │    │  Zabbix Server   │
    │  (Docker)        │    │  (Docker)        │
    │  Ports:          │    │  Ports:          │
    │  - 1514 (Agent)  │    │  - 10051 (Agent) │
    │  - 1515 (Enroll) │    │  - 8080 (Web)    │
    │  - 55000 (API)   │    │                  │
    └────────┬─────────┘    └────────┬─────────┘
             │                       │
             └───────────┬───────────┘
                         │
                         ▼
            ┌────────────────────────┐
            │   API Gateway          │
            │   (Node.js/Express)    │
            │   Port: 3000           │
            │   (Docker Container)   │
            └───────────┬────────────┘
                        │
        ┌───────────────┼───────────────┐
        │               │               │
        ▼               ▼               ▼
┌──────────────┐ ┌─────────────┐ ┌──────────────┐
│ Django       │ │ Node.js     │ │ PostgreSQL   │
│ REST API     │ │ Microservice│ │ Database     │
│ Port: 8000   │ │ Port: 3001  │ │ Port: 5432   │
│ (Container)  │ │ (Container) │ │ (Container)  │
└──────────────┘ └─────────────┘ └──────────────┘
        │               │               │
        └───────────────┼───────────────┘
                        │
                        ▼
            ┌────────────────────────┐
            │   Frontend Dashboard   │
            │   (React/Vue/Angular)  │
            │   Port: 80/443         │
            └────────────────────────┘
```

### 4.2 Tabla de Puertos de la Arquitectura

| Servicio | Puerto Interno | Puerto Externo | Protocolo | Descripción |
|----------|---------------|----------------|-----------|-------------|
| Wazuh Manager | 1514 | 1514 | TCP/UDP | Comunicación de agentes |
| Wazuh Manager | 1515 | 1515 | TCP | Registro de agentes (TLS) |
| Wazuh API | 55000 | 55000 | TCP | API RESTful |
| Wazuh Indexer | 9200 | - | TCP | OpenSearch (interno) |
| Wazuh Dashboard | 5601 | 443 | TCP | Interfaz web |
| Zabbix Agent | 10050 | 10050 | TCP | Checks pasivos |
| Zabbix Server | 10051 | 10051 | TCP | Checks activos |
| Zabbix Web | 8080 | 8080 | TCP | Interfaz web |
| PostgreSQL (Zabbix) | 5432 | - | TCP | Base de datos (interno) |
| PostgreSQL (Django) | 5432 | - | TCP | Base de datos (interno) |
| Django API | 8000 | 8000 | TCP | API REST |
| Node.js Gateway | 3000 | 3000 | TCP | API Gateway |
| Node.js Service | 3001 | - | TCP | Microservicio (interno) |
| Redis | 6379 | - | TCP | Cache/Queue (interno) |
| Nginx | 80/443 | 80/443 | TCP | Reverse Proxy |

### 4.3 Flujo de Datos

1. **Recolección**: Los agentes de Wazuh (puerto 1514) y Zabbix (puertos 10050/10051) recopilan datos de seguridad y métricas
2. **Procesamiento**: Los gestores procesan y almacenan los datos en sus respectivos backends
3. **Exposición**: Las APIs RESTful exponen los datos (Wazuh: 55000, Zabbix: 8080)
4. **Gateway**: API Gateway en Node.js (puerto 3000) agrega y normaliza las solicitudes
5. **Backend**: Django (puerto 8000) procesa la lógica de negocio compleja
6. **Almacenamiento**: PostgreSQL (puerto 5432) persiste datos históricos y configuraciones
7. **Presentación**: Dashboard frontend consume las APIs y visualiza información

---

## 5. Implementación con Docker

### 5.1 Docker Compose para Wazuh

```yaml
version: '3.8'

services:
  wazuh-manager:
    image: wazuh/wazuh-manager:latest
    hostname: wazuh-manager
    restart: always
    ports:
      - "1514:1514"      # Comunicación de agentes
      - "1515:1515"      # Registro de agentes (TLS)
      - "55000:55000"    # API RESTful
    environment:
      - INDEXER_URL=https://wazuh-indexer:9200
      - INDEXER_USERNAME=admin
      - INDEXER_PASSWORD=${INDEXER_PASSWORD}
      - FILEBEAT_SSL_VERIFICATION_MODE=none
    volumes:
      - wazuh_api_configuration:/var/ossec/api/configuration
      - wazuh_etc:/var/ossec/etc
      - wazuh_logs:/var/ossec/logs
      - wazuh_queue:/var/ossec/queue
      - wazuh_var_multigroups:/var/ossec/var/multigroups
      - wazuh_integrations:/var/ossec/integrations
    networks:
      - monitoring_network

  wazuh-indexer:
    image: wazuh/wazuh-indexer:latest
    hostname: wazuh-indexer
    restart: always
    ports:
      - "9200:9200"      # OpenSearch API
    environment:
      - "discovery.type=single-node"
      - "bootstrap.memory_lock=true"
      - "OPENSEARCH_JAVA_OPTS=-Xms1g -Xmx1g"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - wazuh_indexer_data:/var/lib/wazuh-indexer
    networks:
      - monitoring_network

  wazuh-dashboard:
    image: wazuh/wazuh-dashboard:latest
    hostname: wazuh-dashboard
    restart: always
    ports:
      - "443:5601"       # Dashboard web (HTTPS)
    environment:
      - INDEXER_USERNAME=admin
      - INDEXER_PASSWORD=${INDEXER_PASSWORD}
      - WAZUH_API_URL=https://wazuh-manager
      - API_PORT=55000
    depends_on:
      - wazuh-indexer
      - wazuh-manager
    networks:
      - monitoring_network

volumes:
  wazuh_api_configuration:
  wazuh_etc:
  wazuh_logs:
  wazuh_queue:
  wazuh_var_multigroups:
  wazuh_integrations:
  wazuh_indexer_data:

networks:
  monitoring_network:
    driver: bridge
```

### 5.2 Docker Compose para Zabbix con PostgreSQL

```yaml
version: '3.8'

services:
  postgres-zabbix:
    image: postgres:15-alpine
    restart: always
    ports:
      - "5432:5432"      # PostgreSQL (opcional exponer)
    environment:
      - POSTGRES_USER=zabbix
      - POSTGRES_PASSWORD=${POSTGRES_PASSWORD}
      - POSTGRES_DB=zabbix
    volumes:
      - postgres_zabbix_data:/var/lib/postgresql/data
    networks:
      - monitoring_network
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U zabbix"]
      interval: 10s
      timeout: 5s
      retries: 5

  zabbix-server:
    image: zabbix/zabbix-server-pgsql:latest
    restart: always
    ports:
      - "10051:10051"    # Servidor Zabbix
    environment:
      - DB_SERVER_HOST=postgres-zabbix
      - DB_SERVER_PORT=5432
      - POSTGRES_USER=zabbix
      - POSTGRES_PASSWORD=${POSTGRES_PASSWORD}
      - POSTGRES_DB=zabbix
      - ZBX_ENABLE_SNMP_TRAPS=true
    depends_on:
      postgres-zabbix:
        condition: service_healthy
    volumes:
      - zabbix_alertscripts:/usr/lib/zabbix/alertscripts
      - zabbix_externalscripts:/usr/lib/zabbix/externalscripts
      - zabbix_modules:/var/lib/zabbix/modules
    networks:
      - monitoring_network

  zabbix-web:
    image: zabbix/zabbix-web-nginx-pgsql:latest
    restart: always
    ports:
      - "8080:8080"      # Interfaz web Zabbix
      - "8443:8443"      # HTTPS alternativo
    environment:
      - ZBX_SERVER_HOST=zabbix-server
      - ZBX_SERVER_PORT=10051
      - DB_SERVER_HOST=postgres-zabbix
      - DB_SERVER_PORT=5432
      - POSTGRES_USER=zabbix
      - POSTGRES_PASSWORD=${POSTGRES_PASSWORD}
      - POSTGRES_DB=zabbix
      - PHP_TZ=America/Costa_Rica
    depends_on:
      - zabbix-server
      - postgres-zabbix
    networks:
      - monitoring_network

volumes:
  postgres_zabbix_data:
  zabbix_alertscripts:
  zabbix_externalscripts:
  zabbix_modules:

networks:
  monitoring_network:
    external: true
```

### 5.3 Django con PostgreSQL y Gunicorn

```yaml
version: '3.8'

services:
  postgres-django:
    image: postgres:15-alpine
    restart: always
    ports:
      - "5433:5432"      # Puerto alternativo para evitar conflictos
    environment:
      - POSTGRES_DB=monitoring_db
      - POSTGRES_USER=django_user
      - POSTGRES_PASSWORD=${DJANGO_DB_PASSWORD}
    volumes:
      - postgres_django_data:/var/lib/postgresql/data
    networks:
      - monitoring_network
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U django_user"]
      interval: 10s
      timeout: 5s
      retries: 5

  django-app:
    build:
      context: ./django-app
      dockerfile: Dockerfile
    restart: always
    command: gunicorn config.wsgi:application --bind 0.0.0.0:8000 --workers 4
    ports:
      - "8000:8000"      # API REST Django
    environment:
      - DEBUG=False
      - DATABASE_URL=postgresql://django_user:${DJANGO_DB_PASSWORD}@postgres-django:5432/monitoring_db
      - SECRET_KEY=${DJANGO_SECRET_KEY}
      - WAZUH_API_URL=https://wazuh-manager:55000
      - WAZUH_API_USER=admin
      - WAZUH_API_PASSWORD=${WAZUH_API_PASSWORD}
      - ZABBIX_API_URL=http://zabbix-web:8080/api_jsonrpc.php
      - ZABBIX_API_USER=Admin
      - ZABBIX_API_PASSWORD=${ZABBIX_API_PASSWORD}
      - CELERY_BROKER_URL=redis://redis:6379/0
    depends_on:
      postgres-django:
        condition: service_healthy
    volumes:
      - django_static:/app/staticfiles
      - django_media:/app/media
    networks:
      - monitoring_network

  django-celery-worker:
    build:
      context: ./django-app
      dockerfile: Dockerfile
    restart: always
    command: celery -A config worker -l info
    environment:
      - DATABASE_URL=postgresql://django_user:${DJANGO_DB_PASSWORD}@postgres-django:5432/monitoring_db
      - CELERY_BROKER_URL=redis://redis:6379/0
      - WAZUH_API_URL=https://wazuh-manager:55000
      - ZABBIX_API_URL=http://zabbix-web:8080/api_jsonrpc.php
    depends_on:
      - postgres-django
      - redis
    networks:
      - monitoring_network

  django-celery-beat:
    build:
      context: ./django-app
      dockerfile: Dockerfile
    restart: always
    command: celery -A config beat -l info
    environment:
      - DATABASE_URL=postgresql://django_user:${DJANGO_DB_PASSWORD}@postgres-django:5432/monitoring_db
      - CELERY_BROKER_URL=redis://redis:6379/0
    depends_on:
      - postgres-django
      - redis
    networks:
      - monitoring_network

  redis:
    image: redis:7-alpine
    restart: always
    ports:
      - "6379:6379"      # Redis (interno)
    networks:
      - monitoring_network

volumes:
  postgres_django_data:
  django_static:
  django_media:

networks:
  monitoring_network:
    external: true
```

### 5.4 API Gateway con Node.js

```yaml
version: '3.8'

services:
  api-gateway:
    build:
      context: ./api-gateway
      dockerfile: Dockerfile
    restart: always
    ports:
      - "3000:3000"      # API Gateway principal
    environment:
      - NODE_ENV=production
      - PORT=3000
      - WAZUH_API_URL=https://wazuh-manager:55000
      - WAZUH_API_USER=admin
      - WAZUH_API_PASSWORD=${WAZUH_API_PASSWORD}
      - ZABBIX_API_URL=http://zabbix-web:8080/api_jsonrpc.php
      - ZABBIX_API_USER=Admin
      - ZABBIX_API_PASSWORD=${ZABBIX_API_PASSWORD}
      - DJANGO_API_URL=http://django-app:8000/api
      - JWT_SECRET=${JWT_SECRET}
      - REDIS_URL=redis://redis:6379
    depends_on:
      - redis
    networks:
      - monitoring_network
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3

  nodejs-microservice:
    build:
      context: ./nodejs-service
      dockerfile: Dockerfile
    restart: always
    ports:
      - "3001:3001"      # Microservicio Node.js
    environment:
      - NODE_ENV=production
      - PORT=3001
      - DATABASE_URL=postgresql://django_user:${DJANGO_DB_PASSWORD}@postgres-django:5432/monitoring_db
      - REDIS_URL=redis://redis:6379
    networks:
      - monitoring_network

networks:
  monitoring_network:
    external: true
```

### 5.5 Nginx como Reverse Proxy

```yaml
version: '3.8'

services:
  nginx:
    image: nginx:alpine
    restart: always
    ports:
      - "80:80"          # HTTP
      - "443:443"        # HTTPS
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
      - ./nginx/ssl:/etc/nginx/ssl:ro
      - django_static:/usr/share/nginx/html/static:ro
    depends_on:
      - api-gateway
      - django-app
      - wazuh-dashboard
      - zabbix-web
    networks:
      - monitoring_network

volumes:
  django_static:
    external: true

networks:
  monitoring_network:
    external: true
```

**Configuración de Nginx** (`nginx/nginx.conf`):

```nginx
events {
    worker_connections 1024;
}

http {
    upstream api_gateway {
        server api-gateway:3000;
    }

    upstream django_api {
        server django-app:8000;
    }

    upstream wazuh_dashboard {
        server wazuh-dashboard:5601;
    }

    upstream zabbix_web {
        server zabbix-web:8080;
    }

    server {
        listen 80;
        server_name monitoring.example.com;

        # API Gateway
        location /api/v1/ {
            proxy_pass http://api_gateway/;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }

        # Django API
        location /api/django/ {
            proxy_pass http://django_api/api/;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
        }

        # Wazuh Dashboard
        location /wazuh/ {
            proxy_pass https://wazuh_dashboard/;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
        }

        # Zabbix Web
        location /zabbix/ {
            proxy_pass http://zabbix_web/;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
        }

        # Static files
        location /static/ {
            alias /usr/share/nginx/html/static/;
        }
    }
}
```

---

## 6. Implementación del Backend Django

### 6.1 Estructura del Proyecto Django

```
django-app/
├── config/
│   ├── __init__.py
│   ├── settings.py
│   ├── urls.py
│   ├── wsgi.py
│   └── celery.py
├── apps/
│   ├── wazuh_integration/
│   │   ├── __init__.py
│   │   ├── models.py
│   │   ├── views.py
│   │   ├── serializers.py
│   │   ├── services.py
│   │   ├── tasks.py
│   │   └── urls.py
│   ├── zabbix_integration/
│   │   ├── __init__.py
│   │   ├── models.py
│   │   ├── views.py
│   │   ├── serializers.py
│   │   ├── services.py
│   │   ├── tasks.py
│   │   └── urls.py
│   └── dashboard/
│       ├── __init__.py
│       ├── models.py
│       ├── views.py
│       └── urls.py
├── requirements.txt
├── Dockerfile
├── manage.py
└── docker-entrypoint.sh
```

### 6.2 Modelos de Django

```python
# apps/wazuh_integration/models.py
from django.db import models
from django.contrib.postgres.fields import JSONField

class WazuhAlert(models.Model):
    """Modelo para almacenar alertas de Wazuh"""
    alert_id = models.CharField(max_length=100, unique=True, db_index=True)
    timestamp = models.DateTimeField(db_index=True)
    rule_id = models.Int