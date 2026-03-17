# SonarQube Deployment

SonarQube provides static code analysis inside the CI/CD platform.

## Objective

Deploy a self-hosted SonarQube instance backed by PostgreSQL, with persistent storage and a layout suitable for integration with GitLab pipelines.

## Deployment model

The original implementation used Docker Compose with:

- one `sonarqube` service
- one `postgres` service for the database
- persistent volumes for SonarQube state and PostgreSQL data

## Example compose file

```yaml
version: "3"

services:
  sonarqube:
    image: sonarqube:lts-community
    depends_on:
      - sonar_db
    restart: always
    environment:
      SONAR_JDBC_URL: jdbc:postgresql://sonar_db:5432/sonar
      SONAR_JDBC_USERNAME: sonar
      SONAR_JDBC_PASSWORD: sonar
    ports:
      - "9000:9000"
    volumes:
      - sonarqube_conf:/opt/sonarqube/conf
      - sonarqube_data:/opt/sonarqube/data
      - sonarqube_extensions:/opt/sonarqube/extensions
      - sonarqube_logs:/opt/sonarqube/logs
      - sonarqube_temp:/opt/sonarqube/temp

  sonar_db:
    image: postgres:16
    restart: always
    environment:
      POSTGRES_USER: sonar
      POSTGRES_PASSWORD: sonar
      POSTGRES_DB: sonar
    volumes:
      - sonar_db:/var/lib/postgresql
      - sonar_db_data:/var/lib/postgresql/data

volumes:
  sonarqube_conf:
  sonarqube_data:
  sonarqube_extensions:
  sonarqube_logs:
  sonarqube_temp:
  sonar_db:
  sonar_db_data:
```

## Why this layout works

This layout separates:

- application runtime
- SonarQube persistent directories
- database storage

That makes it easier to restart services without losing all analysis state.

## Practical notes

### Default credentials in examples

The compose example uses simple placeholder credentials for clarity. In a real environment, replace them with proper secrets management.

### Port exposure

By default, SonarQube is exposed on port `9000`. If it sits behind a reverse proxy, document the proxy entrypoint and TLS behavior clearly.

### Storage

Keep an eye on:

- database growth
- plugin and extension storage
- logs
- temporary files

### CI integration

Once SonarQube is up, the next step is integrating it into GitLab pipelines so code analysis becomes part of the build and quality gate process.

## Validation checklist

After deployment, confirm:

- SonarQube UI is reachable
- PostgreSQL is healthy
- SonarQube finishes startup successfully
- a sample project can be scanned from CI
- persisted data survives restarts

## Common failure modes

Typical issues include:

- wrong JDBC configuration
- PostgreSQL not ready when SonarQube starts
- disk pressure
- reverse proxy or TLS issues
- broken CI authentication toward SonarQube
