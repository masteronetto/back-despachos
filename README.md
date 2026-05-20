# Backend Despachos - Innovatech Chile

Microservicio REST desarrollado con Spring Boot 3.4.4 y Java 17 que gestiona las órdenes de despacho de Innovatech Chile.

## Tecnologías
- Java 17
- Spring Boot 3.4.4
- MySQL 8.0
- Docker (multi-stage build)
- GitHub Actions (CI/CD)

## Estructura del proyecto
back-Despachos_SpringBoot/
├── Springboot-API-REST-DESPACHO/
│   ├── src/
│   ├── Dockerfile
│   ├── docker-compose.yml
│   └── pom.xml
└── .github/
└── workflows/
└── deploy.yml

## Variables de entorno requeridas

| Variable | Descripción | Ejemplo |
|---|---|---|
| `SPRING_DATASOURCE_URL` | URL de conexión a MySQL | `jdbc:mysql://mysql-despachos:3306/despachos_db` |
| `SPRING_DATASOURCE_USERNAME` | Usuario de la BD | `despachos_user` |
| `SPRING_DATASOURCE_PASSWORD` | Contraseña de la BD | `despachos_pass` |

## Endpoints disponibles

| Método | Endpoint | Descripción |
|---|---|---|
| GET | `/api/v1/despachos` | Lista todos los despachos |
| POST | `/api/v1/despachos` | Crea una nueva orden de despacho |
| PUT | `/api/v1/despachos/{id}` | Modifica/cierra una orden de despacho |

## Levantar con Docker Compose

```bash
# Crear archivo .env con las variables
cp .env.example .env

# Levantar servicios
docker-compose up -d

# Verificar
docker ps
```

## Dockerfile (multi-stage)

El Dockerfile usa multi-stage build:
- **Etapa 1 (builder):** Compila con Maven y Java 17
- **Etapa 2 (producción):** Imagen mínima JRE Alpine, usuario no root `appuser`, puerto 8081

## Persistencia de datos

Se usa **named volume** (`mysql_despachos_data`) para MySQL. Los named volumes garantizan que los datos persisten aunque el contenedor se reinicie o actualice, lo cual es crítico para mantener el historial de despachos de Innovatech.

## Pipeline CI/CD

Se activa con `push` a la rama `deploy`:

1. **Build:** Imagen Docker desde Dockerfile multi-stage
2. **Push:** Publica en Docker Hub (`daniel0netto/back-despachos:latest`)
3. **Deploy:** Actualiza el contenedor en EC2 via SSH

### Secrets requeridos en GitHub

| Secret | Descripción |
|---|---|
| `DOCKER_USERNAME` | Usuario de Docker Hub |
| `DOCKER_TOKEN` | Token de acceso de Docker Hub |
| `EC2_HOST` | IP de la instancia EC2 frontend (salto SSH al backend) |
| `EC2_SSH_KEY` | Clave privada SSH (.pem) |
| `EC2_FRONTEND_HOST` | IP pública del frontend para salto SSH |

## Despliegue en AWS EC2

El backend corre en subred privada (`10.0.129.176`). El acceso está restringido únicamente desde el Security Group del Frontend, cumpliendo el principio de mínimo privilegio.
