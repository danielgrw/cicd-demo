
# CICD-DEMO

This project aims to be the basic skeleton to apply continuous integration and continuous delivery.

## Topology

CICD Demo uses some kubernetes primitives to deploy:

* Deployment
* Services
* Ingress ( with TLS )

```bash
     internet
        |
   [ Ingress ]
   --|-----|--
   [ Services ]
   --|-----|--
   [   Pods   ]

```

This project includes:

* Spring Boot java app
* Jenkinsfile integration to run pipelines
* Dockerfile containing the base image to run java apps
* Makefile and docker-compose to make the pipeline steps much simpler
* Kubernetes deployment file demonstrating how to deploy this app in a simple Kubernetes cluster

## Pipeline Setup

Pipelines exist at Travis.

Some pipelines are configured by **GitHub/Projects**. If you have created a repository in one of these, your project will be **automatically** built if it has a Jenkinsfile/Travis/Gitlab/CircleCI.

Other pipelines are configured manually under folders. You can create a project manually with the following steps:

## Taller Jenkins (mínimo requerido)

Este repositorio ya incluye un `Jenkinsfile` declarativo para usar **Pipeline script from SCM** con estas etapas:

1. `Checkout` (código desde SCM)
2. `Build` (`mvn clean package -DskipTests`)
3. `Test` (`mvn test`)
4. `Docker Build` (`docker build -t mi-app:latest .`)
5. `Static Analysis (SonarQube)`
6. `Quality Gate` (falla el pipeline si SonarQube no aprueba)
7. `Container Security Scan (Trivy)` (falla si encuentra vulnerabilidades `CRITICAL`)
8. `Deploy` (solo en `main`/`master`, levanta contenedor local)

### Requisitos en Jenkins

- Plugins: **Git**, **Pipeline**, **Docker Pipeline**, **SonarQube Scanner for Jenkins**, **Workspace Cleanup**
- Herramientas en el agente Jenkins: `docker`, `mvn`, `trivy`
- Configurar SonarQube en Jenkins con el nombre: `SonarQube`

### SonarQube local (ejemplo)

```bash
docker run -d --name sonarqube -p 9000:9000 sonarqube:community
```

### Trivy en agente Jenkins (Linux, ejemplo)

```bash
sudo apt-get update
sudo apt-get install -y wget gnupg
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo gpg --dearmor -o /usr/share/keyrings/trivy.gpg
echo "deb [signed-by=/usr/share/keyrings/trivy.gpg] https://aquasecurity.github.io/trivy-repo/deb generic main" | sudo tee /etc/apt/sources.list.d/trivy.list
sudo apt-get update
sudo apt-get install -y trivy
```

### Flujo de validación final

1. Crear job tipo **Pipeline** en Jenkins.
2. Seleccionar **Pipeline script from SCM** y apuntar a este repositorio.
3. Hacer un cambio pequeño en la app y `push`.
4. Verificar que Jenkins ejecute todo el pipeline.
5. Confirmar despliegue local en `http://localhost` cuando las validaciones pasan.

How to run the app:

```make
make
```

## Testing

Unit tests and integrations tests are separated using [JUnit Categories][].

[JUnit Categories]: https://maven.apache.org/surefire/maven-surefire-plugin/examples/junit.html

### Unit Tests

```java
mvn test -Dgroups=UnitTest
```

Or using Docker:

```bash
make build
```

### Integration Tests

```java
mvn integration-test -Dgroups=IntegrationTests
```

Or using Docker:

```bash
make integrationTest
```

### System Tests

System tests run with Selenium using docker-compose to run a [Selenium standalone container][] with Chrome.

[Selenium standalone container]: https://github.com/SeleniumHQ/docker-selenium

Using Docker:

* If you are running locally, make sure the `$APP_URL` is populated and points to a valid instance of your application. This variable is populated automatically in Jenkins.

```bash
APP_URL=http://dev-cicd-demo-master.anzcd.internal/ make systemTest
```
