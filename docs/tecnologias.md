# Tecnologías utilizadas

A continuación, se presenta un resumen de las tecnologías listadas, sus usos principales, una tabla de puertos (según la información disponible en las fuentes) y cómo integrarlas utilizando Docker.

## Resumen de Tecnologías y Usos

| Tecnología | Uso Principal |
|:-----------|:--------------|
| **Node.js (Runtime)** | Es un entorno de ejecución de JavaScript de código abierto, basado en el motor V8. Es el campeón de la eficiencia para operaciones de I/O y está diseñado para construir **aplicaciones de red escalables** sin bloqueo. Ideal para APIs de alto rendimiento, microservicios, funciones *serverless* y aplicaciones en tiempo real (chat o *streaming*). |
| **Django (Framework)** | Un *framework* web de Python de alto nivel que promueve el desarrollo rápido, limpio y seguro. Es la opción preferida para **aplicaciones web complejas y robustas**, automatización, y servicios *backend* que requieren algoritmos complejos o integración con modelos de IA, ciencia de datos y *machine learning*. |
| **PostgreSQL (Base de Datos)** | Potente sistema de gestión de bases de datos objeto-relacional de código abierto, con más de 35 años de desarrollo. Reconocido por su **fiabilidad, robustez y rendimiento**. |
| **Express.js (Framework)** | El *framework* web ligero más popular en el ecosistema de Node.js, conocido por su simplicidad. Es minimalista y flexible, con un ecosistema masivo de *middleware*, y sigue siendo relevante en 2025.
| **Docker (Plataforma)** | Plataforma de contenedores confiable para construir, asegurar, compartir y ejecutar agentes y aplicaciones. Asegura que el código "funcione en todas partes", proporcionando un entorno consistente. |
| **Docker Desktop** | Se utiliza para contenerizar aplicaciones, definir aplicaciones multi-servicio y realizar pruebas de extremo a extremo localmente. También facilita la conexión a servidores MCP (*Managed Component Provider*) contenedorizados en segundos. |
| **Docker Hardened Images** | Imágenes de contenedores seguras y listas para empresas, construidas para mantener un número casi nulo de vulnerabilidades conocidas (CVEs), lo que reduce la superficie de ataque hasta en un 97%. |

## Tabla de Puertos

Los ejemplos de código y los detalles de las fuentes sugieren el uso del puerto **3000** para el desarrollo en Node.js, aunque las fuentes no especifican puertos por defecto para todos los componentes de la pila.

| Tecnología | Puerto(s) común(es) utilizado(s) |
|:-----------|:----------------------------------|
| **Node.js (Servidores Express/Fastify/Koa)** | 3000 (Comúnmente utilizado para desarrollo de aplicaciones web o API) [1-3, 9, 16] |
| **PostgreSQL** | 5432 (Puerto por defecto estándar) |
| **Django** | 8000 (Puerto por defecto para desarrollo) [5] |
| **Docker (Componentes)** | Varía según configuración |

## Integración de Tecnologías con Docker

Docker unifica toda la pila tecnológica mediante contenedores aislados que se comunican entre sí a través de redes definidas. Cada servicio (Node.js, Django, PostgreSQL) se ejecuta en su propio contenedor con sus dependencias específicas, eliminando conflictos y garantizando reproducibilidad en cualquier entorno [1].

### Ejemplo de Integración con Docker Compose

El siguiente ejemplo muestra cómo orquestar los tres servicios principales de la pila:

```yaml
version: '3.8'

services:
  # Base de datos PostgreSQL
  postgres:
    image: postgres:16-alpine
    container_name: db_postgres
    environment:
      POSTGRES_DB: myapp_db
      POSTGRES_USER: admin
      POSTGRES_PASSWORD: secure_password
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
    networks:
      - app_network

  # Servicio Django (Backend Python)
  django:
    build:
      context: ./django-service
      dockerfile: Dockerfile
    container_name: django_backend
    command: python manage.py runserver 0.0.0.0:8000
    environment:
      DATABASE_URL: postgresql://admin:secure_password@postgres:5432/myapp_db
    ports:
      - "8000:8000"
    depends_on:
      - postgres
    volumes:
      - ./django-service:/app
    networks:
      - app_network

  # Servicio Node.js (API/Microservicio)
  nodejs:
    build:
      context: ./nodejs-service
      dockerfile: Dockerfile
    container_name: nodejs_api
    environment:
      DATABASE_URL: postgresql://admin:secure_password@postgres:5432/myapp_db
      DJANGO_API_URL: http://django:8000
    ports:
      - "3000:3000"
    depends_on:
      - postgres
      - django
    volumes:
      - ./nodejs-service:/app
      - /app/node_modules
    networks:
      - app_network

volumes:
  postgres_data:

networks:
  app_network:
    driver: bridge
```

### Dockerfiles de Ejemplo

**Dockerfile para Django:**
```dockerfile
FROM python:3.11-slim

WORKDIR /app

# Instalar dependencias del sistema
RUN apt-get update && apt-get install -y \
    postgresql-client \
    && rm -rf /var/lib/apt/lists/*

# Copiar requirements e instalar dependencias Python
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copiar el código de la aplicación
COPY . .

# Exponer el puerto de Django
EXPOSE 8000

# Comando por defecto (se puede sobrescribir en docker-compose)
CMD ["python", "manage.py", "runserver", "0.0.0.0:8000"]
```

**Dockerfile para Node.js:**
```dockerfile
FROM node:20-alpine

WORKDIR /app

# Copiar package.json y package-lock.json
COPY package*.json ./

# Instalar dependencias
RUN npm ci --only=production

# Copiar el código de la aplicación
COPY . .

# Exponer el puerto de Node.js
EXPOSE 3000

# Comando por defecto
CMD ["node", "index.js"]
```

### Ventajas de esta Arquitectura

1. **Aislamiento**: Cada servicio tiene su propio entorno sin interferencias [1]
2. **Escalabilidad**: Los servicios pueden escalarse independientemente
3. **Comunicación**: Los contenedores se comunican por nombres de servicio (ej: `postgres`, `django`)
4. **Persistencia**: Los volúmenes garantizan que los datos de PostgreSQL persistan
5. **Lanzamiento Simple**: Todo arranca con `docker compose up -d` [1]

Esta configuración permite que Node.js y Django compartan la misma base de datos PostgreSQL, mientras Django puede exponer APIs que Node.js consume, creando un ecosistema integrado donde cada tecnología aporta sus fortalezas específicas.

## Bibliografía (IEEE)

[1] Docker, Inc., "Docker: Accelerated Container Application Development." [En línea]. Disponible en: https://www.docker.com

[2] The Node.js Project, "Node.js v25.0.0 (Current)." [En línea]. Disponible en: https://nodejs.org

[3] The Node.js Project, "Introduction to Node.js." [En línea]. Disponible en: https://nodejs.org/en/learn/getting-started/introduction-to-nodejs

[4] The PostgreSQL Global Development Group, "PostgreSQL: The World's Most Advanced Open Source Relational Database." [En línea]. Disponible en: https://www.postgresql.org

[5] Django Software Foundation, "The web framework for perfectionists with deadlines." [En línea]. Disponible en: https://www.djangoproject.com

[6] R. Gonzaga, "Node.js 21 is now available!," *The Node.js Project*, 17 de octubre de 2023.

[7] Z. Dev, "Cómo manejar errores en Node.js de manera efectiva," 3 de julio de 2024.

[8] A. Isaiah, "Express Alternatives for Modern Node.js Web Development," *Better Stack Community*, 28 de abril de 2025.

[9] R. Gonzaga, "Node.js v25.0.0 (Current)," *The Node.js Project*, 15 de octubre de 2025.

[10] H. Khan, "Rust vs Node.js vs Go: Performance Comparison for Backend Development 🏎️," *DEV Community*, 23 de septiembre de 2024.

[11] A. S. Makwana, "Node.js 25 Version Release: A Quick Overview," 10 de noviembre de 2025.

[12] C. Orrego Dev, "Top 5 patrones de diseño en Node.js." [En línea].

[13] H. Khan, "Node.js vs. Go – Settling the Debate Once and for All (2025 Edition)," *DEV Community*, 9 de abril de 2025.

[14] S. Gaurav, "Scaling Node.js Applications: Strategies and Best Practices," *DEV Community*, 21 de octubre de 2024.

[15] H. Khan, "Backend con Node.js y Python en 2025: ¿El Dúo Dinámico o Rivales Eternos?," *Reddit*. [En línea].

[16] The Node.js Project, "Sobre Node.js®." [En línea]. Disponible en: https://nodejs.org/es/about