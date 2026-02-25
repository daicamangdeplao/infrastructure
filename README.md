# Infrastructure Services

A Docker Compose setup for local development infrastructure including databases, artifact repositories, code quality
tools, and message brokers.

## Overview

This project provides a complete local development infrastructure stack. All services are containerized and can be run
individually or together based on your needs.

## Services

| Service              | Version          | Port(s)        | Description                                      |
|----------------------|------------------|----------------|--------------------------------------------------|
| **PostgresML 2.10**  | 2.10.0           | 5434, 8001     | PostgreSQL + ML extension with dashboard         |
| **PostgreSQL 16.1**  | 16.1             | 5432           | Primary PostgreSQL database (custom build)       |
| **PostgreSQL 13.16** | 13.16            | 5439           | Legacy PostgreSQL database                       |
| **MS SQL Server**    | 2019-latest      | 1433           | Microsoft SQL Server                             |
| **Nexus**            | 3 (latest)       | 8111           | Artifact repository manager                      |
| **SonarQube**        | 25.9.0.112764    | 9000           | Code quality and security scanner                |
| **RabbitMQ**         | 4.2.3-management | 5672, 15672    | Message broker with management UI                |

## Prerequisites

- Docker Engine 20.10+
- Docker Compose 1.29+
- At least 8GB RAM available for Docker (16GB recommended for all services)

## Environment Setup

Create a `.env` file in the root directory with the following variables:

```env
# PostgresML Configuration
POSTGRESML_USER=postgresml

# PostgreSQL Configuration
POSTGRES_USER=postgres
POSTGRES_PASSWORD=your_postgres_password

# MS SQL Server Configuration
ACCEPT_EULA=Y
MSSQL_SA_PASSWORD=YourStrong@Passw0rd
MSSQL_PID=Developer

# PgAdmin Configuration (if enabled)
# PGADMIN_DEFAULT_EMAIL=admin@example.com
# PGADMIN_DEFAULT_PASSWORD=admin
```

**Note:** MS SQL Server password must meet complexity requirements (uppercase, lowercase, numbers, special characters,
minimum 8 characters).

## Running Services

### Start All Services

To start all infrastructure services:

```bash
docker-compose up -d
```

To view logs:

```bash
docker-compose logs -f
```

To stop all services:

```bash
docker-compose down
```

To stop and remove volumes (⚠️ deletes all data):

```bash
docker-compose down -v
```

### Start Specific Services

Run only the services you need by specifying their names:

#### Database Only

```bash
# PostgresML 2.10 only
docker-compose up -d postgresml210

# PostgreSQL 16.1 only
docker-compose up -d postgresql161

# PostgreSQL 16.1 only with custom build
docker-compose up -d --build postgresql161

# PostgreSQL 13.16 only
docker-compose up -d postgresql1316

# MS SQL Server only
docker-compose up -d mssql

# Both PostgreSQL instances
docker-compose up -d postgresql161 postgresql1316
```

#### Development Tools

```bash
# Nexus repository only
docker-compose up -d nexus

# SonarQube only
docker-compose up -d sonarqube

# RabbitMQ only
docker-compose up -d rabbitmq
```

#### Custom Combinations

```bash
# Databases + Message Broker
docker-compose up -d postgresql161 mssql rabbitmq

# Full CI/CD stack
docker-compose up -d nexus sonarqube
```

## Accessing Services

### Databases

**PostgresML 2.10:**

```
Host:     localhost
Port:     5434
Database: postgresml
Username: postgresml (from .env POSTGRESML_USER)
```

Dashboard: http://localhost:8001

**PostgreSQL 16.1:**

```
Host: localhost
Port: 5432
Username: postgres (from .env)
Password: (from .env)
```

**PostgreSQL 13.16:**

```
Host: localhost
Port: 5439
Username: postgres (from .env)
Password: (from .env)
```

**MS SQL Server:**

```
Host: localhost
Port: 1433
Username: sa
Password: (MSSQL_SA_PASSWORD from .env)
```

## PostgresML

[PostgresML](https://postgresml.org/) brings machine learning capabilities directly into PostgreSQL. It ships as an
all-in-one Docker image that bundles the `pgml` extension, a pre-loaded `postgresml` database, and a web dashboard for
running and monitoring ML workloads — without any additional infrastructure.

### What you can do with PostgresML

- **Train & run models in SQL** — supervised learning, regression, classification directly via SQL functions
- **Embeddings & vector search** — generate and query embeddings using built-in sentence transformers
- **LLM inference** — run large language models (e.g. Llama, Mistral) through `pgml.transform()`
- **pgvector integration** — native `vector` type for similarity search

### Starting the service

```bash
docker-compose up -d postgresml210
```

The configuration in docker-compose.yml is equivalent to ``docker run -it -v postgresml_data:/var/lib/postgresql -p 5434:5432 -p 8001:8000 ghcr.io/postgresml/postgresml:2.10.0 sudo -u postgresml psql -d postgresml``.

The container starts an interactive `psql` session as the user defined by `POSTGRESML_USER`. Because it uses a
pseudo-TTY (`tty: true`), attach to it with:

```bash
docker attach postgresml210
```

Or open an independent psql session:

```bash
docker exec -it postgresml210 psql -U postgresml -d postgresml
```

### Ports

| Port (host) | Port (container) | Purpose              |
|-------------|------------------|----------------------|
| `5434`      | `5432`           | PostgreSQL           |
| `8001`      | `8000`           | Dashboard / REST API |

### Dashboard

Open **http://localhost:8001** in a browser to access the PostgresML web dashboard. It provides:

- A query editor for running `pgml.*` functions interactively
- Model training and deployment views
- Real-time inference monitoring

### Quick start (SQL)

```sql
-- Generate an embedding
SELECT pgml.embed('sentence-transformers/all-MiniLM-L6-v2', 'Hello, world!');

-- Train a simple regression model
SELECT pgml.train(
    project_name => 'my_model',
    task         => 'regression',
    relation_name => 'my_table',
    y_column_name => 'target'
);

-- Run inference
SELECT pgml.predict('my_model', ARRAY[1.0, 2.0, 3.0]);
```

### Environment variable

| Variable           | Default      | Description                                      |
|--------------------|--------------|--------------------------------------------------|
| `POSTGRESML_USER`  | `postgresml` | OS user used to run the initial `psql` command   |

### Persistent data

Data is stored in the `postgresml_data` named volume mounted at `/var/lib/postgresql`.

## PostgreSQL Extensions

To add PostgreSQL extensions (e.g., pgvector for vector similarity search), you have two options:

### Option 1: Install Directly in Running Container

Connect to the PostgreSQL container and create the extension:

```bash
# Connect to PostgreSQL
docker exec -it postgres-16.1 psql -U postgres

# Create the extension (inside psql)
CREATE EXTENSION vector;
```

### Option 2: Include in Application Schema

Add the extension to your application's `schema.sql` file (recommended for Spring Boot projects):

```sql
CREATE EXTENSION IF NOT EXISTS vector;
```

This ensures the extension is created automatically when your application initializes the database.

### Tools

**Nexus Repository:**

- URL: http://localhost:8111
- Default credentials: admin / (retrieve from container logs on first run)
- Get initial password: `docker exec nexus3 cat /nexus-data/admin.password`

**SonarQube:**

- URL: http://localhost:9000
- Default credentials: admin / admin (change on first login)
- ⏱️ Takes 1-2 minutes to start

**RabbitMQ Management:**

- URL: http://localhost:15672
- Default credentials: rabbitmq / rabbitmq
- AMQP Port: 5672

## Health Checks

All services include health checks. Monitor service health:

```bash
docker-compose ps
```

Check specific service health:

```bash
docker inspect --format='{{.State.Health.Status}}' <container_name>
```

## Resource Limits

Services are configured with resource limits to prevent system overload:

| Service          | CPU Limit | Memory Limit | CPU Reserved | Memory Reserved |
|------------------|-----------|--------------|--------------|-----------------|
| PostgreSQL 16.1  | 2         | 2GB          | 0.5          | 512MB           |
| PostgreSQL 13.16 | 2         | 2GB          | 0.5          | 512MB           |
| MS SQL Server    | 4         | 4GB          | 1            | 2GB             |
| Nexus            | 2         | 4GB          | 0.5          | 2GB             |
| SonarQube        | 2         | 4GB          | 0.5          | 2GB             |

## Persistent Data

All service data is stored in named Docker volumes:

- `postgresml_data` - PostgresML 2.10 data
- `pgdata161` - PostgreSQL 16.1 data
- `pgdata1316` - PostgreSQL 13.16 data
- `mssql2019` - MS SQL Server data
- `nexus-data` - Nexus repository data
- `sonarqube_extensions` - SonarQube plugins and extensions
- `rabbitmq-data` - RabbitMQ data

List volumes:

```bash
docker volume ls
```

## Troubleshooting

### Service won't start

Check logs for the specific service:

```bash
docker-compose logs <service_name>
```

### Port conflicts

If ports are already in use, modify the port mappings in `docker-compose.yml`:

```yaml
ports:
  - "NEW_PORT:CONTAINER_PORT"
```

### Out of memory

Reduce the number of running services or increase Docker's memory allocation in Docker Desktop settings.

### Reset a service

Stop and remove a specific service with its volume:

```bash
docker-compose stop <service_name>
docker-compose rm -f <service_name>
docker volume rm infrastructure_<volume_name>
docker-compose up -d <service_name>
```

## Development Workflow

**New developer onboarding:**

1. Clone the repository
2. Create `.env` file with credentials
3. Run `docker-compose up -d`
4. Access services via localhost

## References

- [PostgresML Documentation](https://postgresml.org/docs)
- [PostgresML GitHub](https://github.com/postgresml/postgresml)
- [PostgresML Docker Hub (GHCR)](https://github.com/postgresml/postgresml/pkgs/container/postgresml)
- [PostgreSQL Docker Setup](https://www.baeldung.com/ops/postgresql-docker-setup)
- [PostgreSQL Docker Official Image](https://www.docker.com/blog/how-to-use-the-postgres-docker-official-image/)
- [PostgreSQL Docker Hub](https://hub.docker.com/_/postgres)
- [MS SQL Server Docker](https://learn.microsoft.com/en-us/sql/linux/quickstart-install-connect-docker)
- [Nexus Repository Docker](https://hub.docker.com/r/sonatype/nexus3)
- [SonarQube Docker](https://hub.docker.com/_/sonarqube)
- [RabbitMQ Docker](https://hub.docker.com/_/rabbitmq)
