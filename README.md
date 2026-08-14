```markdown
# EmployeeHub

EmployeeHub is a Spring Boot-based employee management application built with Java 21 and Maven.

The project also demonstrates a complete DevOps workflow using GitHub, GitHub Actions, Docker, GitHub Container Registry (GHCR), SonarQube, JaCoCo, and Kubernetes.

---

# Architecture

The overall workflow is:

Developer
   |
   | git push
   v
GitHub Repository
   |
   v
GitHub Actions
   |
   +--------------------+
   |                    |
   v                    v
Maven Tests          SonarQube
   |                    |
   v                    v
JaCoCo              Code Analysis
   |
   v
Docker Build
   |
   v
Docker Image
   |
   v
GitHub Container Registry (GHCR)
   |
   v
Kubernetes
   |
   +-----------------------------+
   |              |              |
   v              v              v
 Pod 1          Pod 2          Pod 3
   |              |              |
   +--------------+--------------+
                  |
                  v
        Kubernetes Service
                  |
                  v
           EmployeeHub
```

---

# Technology Stack

| Technology     | Purpose                            |
| -------------- | ---------------------------------- |
| Java 21        | Application development            |
| Spring Boot    | Backend framework                  |
| Maven          | Build and dependency management    |
| JUnit          | Application testing                |
| JaCoCo         | Java test coverage                 |
| SonarQube      | Static code quality analysis       |
| Docker         | Application containerization       |
| GitHub         | Source code management             |
| GitHub Actions | CI/CD automation                   |
| GHCR           | Docker image registry              |
| Kubernetes     | Container orchestration            |
| Docker Desktop | Local Kubernetes cluster           |
| Render         | Cloud deployment option            |
| Terraform      | Infrastructure-as-code exploration |

---

# 1. Application Architecture

EmployeeHub is a Spring Boot application.

The application follows a layered architecture:

```text
Controller
    |
    v
Service
    |
    v
Repository
    |
    v
Database
```

### Controller

Handles HTTP requests and responses.

### Service

Contains business logic.

### Repository

Handles database access.

> Note: Controller/Service/Repository separation is a layered architecture. It does not by itself mean the application is a microservices architecture.

---

# 2. GitHub Repository

The source code is maintained in GitHub.

The basic development workflow is:

```text
Developer changes code
        |
        v
git add .
        |
        v
git commit
        |
        v
git push origin main
        |
        v
GitHub
```

---

# 3. Maven

The project uses the Maven Wrapper:

```text
mvnw
mvnw.cmd
```

The Maven Wrapper allows the project to execute Maven commands without requiring a separately configured Maven installation.

For example:

```powershell
.\mvnw.cmd test
```

Run tests:

```powershell
.\mvnw.cmd test
```

Build the application:

```powershell
.\mvnw.cmd clean package
```

---

# 4. GitHub Actions

The CI/CD workflow is located at:

```text
.github/workflows/ci.yml
```

The workflow is triggered when code is pushed to the `main` branch or when a pull request targets `main`.

The pipeline performs tasks such as:

```text
Checkout source code
        |
        v
Set up Java 21
        |
        v
Run Maven tests
        |
        v
Build Spring Boot application
        |
        v
Build Docker image
        |
        v
Push Docker image to GHCR
        |
        v
Deploy to Kubernetes
```

The project also uses a self-hosted GitHub Actions runner.

---

# 5. Self-Hosted GitHub Actions Runner

The project uses a self-hosted GitHub Actions runner named:

```text
employeehub-runner
```

The runner is installed on the local Windows machine.

This is useful because the Kubernetes cluster is running locally through Docker Desktop.

The flow is:

```text
GitHub
   |
   v
Self-hosted GitHub Actions Runner
   |
   v
Windows Machine
   |
   v
Docker Desktop Kubernetes
```

A GitHub-hosted runner normally cannot directly access the Kubernetes cluster running on your local machine.

The self-hosted runner allows the workflow to execute commands such as:

```powershell
kubectl apply -f k8s/deployment.yaml
```

against the local Kubernetes cluster.

---

# 6. SonarQube

SonarQube is used for static code analysis.

It helps identify:

* Bugs
* Vulnerabilities
* Code smells
* Maintainability issues
* Duplicated code
* Other code quality issues

The project was successfully analyzed using SonarQube.

Example:

```text
Maven
   |
   v
SonarQube Scanner
   |
   v
SonarQube Server
   |
   v
EmployeeHub Code Quality Report
```

SonarQube does not replace unit testing.

Testing and static analysis have different purposes.

---

# 7. JaCoCo

JaCoCo stands for:

**Java Code Coverage**

JaCoCo measures how much of the Java code is executed by automated tests.

The flow is:

```text
JUnit Tests
    |
    v
JaCoCo
    |
    v
Coverage Report
    |
    v
SonarQube
```

The Maven project contains the JaCoCo Maven plugin.

A successful build produced:

```text
Tests run: 1
Failures: 0
Errors: 0
```

and generated:

```text
target/jacoco.exec
```

and a JaCoCo report.

---

# 8. Docker

The application is packaged into a Docker image.

Dockerfile:

```dockerfile
FROM eclipse-temurin:21-jre

WORKDIR /app

COPY target/*.jar app.jar

EXPOSE 8080

ENTRYPOINT ["java", "-jar", "app.jar"]
```

The Dockerfile performs the following:

```text
Spring Boot JAR
      |
      v
Java 21 Runtime
      |
      v
Docker Image
```

Build the image:

```powershell
docker build -t employeehub:latest .
```

The application listens on:

```text
8080
```

---

# 9. GitHub Container Registry

The Docker image is stored in GitHub Container Registry (GHCR).

Image:

```text
ghcr.io/pugalesan-pugal/employeehub:latest
```

The registry provides a central location from which authorized systems can pull the Docker image.

The flow is:

```text
Docker Build
     |
     v
EmployeeHub Image
     |
     v
GHCR
     |
     +------> Kubernetes
     |
     +------> Other authorized environments
```

A versioned image such as:

```text
ghcr.io/pugalesan-pugal/employeehub:2.0
```

was also built locally, but the GHCR authentication issue for pushing that specific version was intentionally skipped.

The current Kubernetes deployment uses:

```text
ghcr.io/pugalesan-pugal/employeehub:latest
```

---

# 10. Kubernetes

Kubernetes is used to orchestrate the EmployeeHub containers.

The Kubernetes configuration is stored in:

```text
k8s/
```

Current files:

```text
k8s/
├── deployment.yaml
├── service.yaml
├── configmap.yaml
└── secret.yaml
```

---

# 11. Kubernetes Deployment

The Deployment manages the EmployeeHub Pods.

The current configuration uses:

```yaml
replicas: 3
```

Therefore Kubernetes maintains three EmployeeHub Pods.

Example:

```text
Deployment
     |
     v
ReplicaSet
     |
     +------ Pod 1
     |
     +------ Pod 2
     |
     +------ Pod 3
```

Current state:

```text
3/3 Pods Running
```

---

# 12. Kubernetes Service

The application is exposed using a Kubernetes Service.

Service:

```text
employeehub-service
```

Type:

```text
NodePort
```

The current port mapping is:

```text
8080:32478
```

Therefore the application can be accessed locally through:

```text
http://localhost:32478
```

The Service routes traffic to the EmployeeHub Pods.

```text
User
 |
 v
Kubernetes Service
 |
 +------ Pod 1
 |
 +------ Pod 2
 |
 +------ Pod 3
```

The Service provides a stable endpoint even though individual Pod names and IP addresses can change.

---

# 13. Kubernetes ConfigMap

The project uses a ConfigMap for normal application configuration.

File:

```text
k8s/configmap.yaml
```

Example:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: employeehub-config
data:
  SPRING_PROFILES_ACTIVE: "default"
```

The Deployment loads these values using:

```yaml
envFrom:
  - configMapRef:
      name: employeehub-config
```

ConfigMaps should contain non-sensitive configuration.

---

# 14. Kubernetes Secret

The project also demonstrates Kubernetes Secrets.

File:

```text
k8s/secret.yaml
```

Example:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: employeehub-secret
type: Opaque
stringData:
  DEMO_SECRET: "my-secret-value"
```

The Deployment loads the Secret using:

```yaml
envFrom:
  - secretRef:
      name: employeehub-secret
```

Secrets are intended for sensitive configuration such as:

* Passwords
* Tokens
* Credentials
* API keys

The current secret is only a dummy learning value.

Real credentials should never be committed directly into Git.

---

# 15. Kubernetes Health Checks

The EmployeeHub Deployment uses Spring Boot Actuator health checks.

Endpoint:

```text
/actuator/health
```

Two Kubernetes probes are configured.

## Readiness Probe

The readiness probe determines whether a Pod is ready to receive traffic.

```yaml
readinessProbe:
  httpGet:
    path: /actuator/health
    port: 8080
  initialDelaySeconds: 20
  periodSeconds: 10
```

If a Pod is not ready, Kubernetes can temporarily remove it from Service traffic.

## Liveness Probe

The liveness probe determines whether the application is still alive.

```yaml
livenessProbe:
  httpGet:
    path: /actuator/health
    port: 8080
  initialDelaySeconds: 30
  periodSeconds: 10
```

If the application repeatedly fails the liveness check, Kubernetes can restart the container.

---

# 16. Kubernetes Resource Requests and Limits

The Deployment also defines resource requests and limits.

```yaml
resources:
  requests:
    cpu: "250m"
    memory: "256Mi"
  limits:
    cpu: "500m"
    memory: "512Mi"
```

Requests represent the resources the container expects.

Limits define the maximum resources the container should consume.

This helps Kubernetes schedule and control workloads.

---

# 17. Kubernetes Scaling

The current Deployment has:

```yaml
replicas: 3
```

Therefore:

```text
Low traffic
    |
    v
3 Pods
```

If one Pod crashes:

```text
Pod 1  Running
Pod 2  Failed
Pod 3  Running
```

Kubernetes/ReplicaSet attempts to create another Pod so that the desired replica count returns to three.

However, the current project does **not** yet have automatic traffic-based scaling.

For example:

```text
Traffic increases
       |
       v
3 Pods → 5 Pods → 10 Pods
```

would require a Horizontal Pod Autoscaler (HPA) and a working metrics source.

The local Docker Desktop cluster currently does not have the Metrics API available.

Therefore:

```text
Current:
3 fixed replicas
+
automatic replacement of failed Pods

Not yet:
automatic traffic-based scaling
```

---

# 18. Kubernetes vs Render

Kubernetes and Render are separate deployment environments.

They should not be thought of as:

```text
Kubernetes → Render
```

Instead, the project has two possible deployment paths.

## Kubernetes

```text
GitHub
   |
   v
GitHub Actions
   |
   v
Self-hosted Runner
   |
   v
Docker Desktop Kubernetes
   |
   v
EmployeeHub Pods
```

## Render

GitHub Actions can also trigger a Render deployment using a Render Deploy Hook:

```text
GitHub
   |
   v
GitHub Actions
   |
   v
Render Deploy Hook
   |
   v
Render
   |
   v
Live Application
```

The Render deployment is independent of the local Kubernetes deployment.

---

# 19. Terraform

Terraform was explored as an Infrastructure-as-Code tool.

The purpose of Terraform is to define infrastructure using configuration files rather than manually creating infrastructure.

For example:

```text
Terraform
    |
    v
Infrastructure Configuration
    |
    v
Cloud / Platform Resources
```

Terraform was installed and explored for the project, but Terraform deployment was intentionally skipped.

Therefore Terraform is currently **not part of the completed CI/CD deployment path**.

---

# 20. Complete CI/CD Flow

The intended complete workflow is:

```text
Developer
    |
    | Code changes
    v
GitHub Repository
    |
    | git push
    v
GitHub Actions
    |
    +----------------------+
    |                      |
    v                      v
Maven Tests             SonarQube
    |                      |
    v                      v
JaCoCo Coverage       Code Quality
    |
    v
Maven Build
    |
    v
Docker Build
    |
    v
Docker Image
    |
    v
GHCR
    |
    v
Self-Hosted Runner
    |
    v
kubectl
    |
    v
Kubernetes Deployment
    |
    v
3 EmployeeHub Pods
    |
    v
Kubernetes Service
    |
    v
EmployeeHub
```

---

# 21. What Happens When a User Accesses the Application?

When the application is served through Kubernetes:

```text
User
  |
  v
Kubernetes Service
  |
  +----------+----------+
  |          |          |
  v          v          v
 Pod 1      Pod 2      Pod 3
  |          |          |
  +----------+----------+
             |
             v
       EmployeeHub
```

The Service distributes traffic among the available Pods.

Kubernetes also maintains the desired number of replicas.

For example, if one Pod fails:

```text
Pod 1    Running
Pod 2    Failed
Pod 3    Running
```

Kubernetes attempts to restore the desired state:

```text
Pod 1    Running
Pod 2    New Pod
Pod 3    Running
```

---

# 22. Local Kubernetes vs Production Kubernetes

The current Kubernetes environment is:

```text
Docker Desktop Kubernetes
```

running on the local development machine.

Therefore it is primarily useful for:

* Learning
* Development
* Testing
* Demonstrating Kubernetes deployment

It is not currently a publicly accessible production Kubernetes cluster.

A production architecture could use a managed Kubernetes service such as:

```text
Azure Kubernetes Service (AKS)
Amazon EKS
Google Kubernetes Engine (GKE)
```

and expose the application through an appropriate cloud Load Balancer/Ingress.

---

# 23. Project Status

## Completed

* [x] Spring Boot application
* [x] Layered Controller/Service/Repository architecture
* [x] GitHub repository
* [x] Maven Wrapper
* [x] Automated tests
* [x] JaCoCo coverage
* [x] SonarQube analysis
* [x] Dockerfile
* [x] Docker image
* [x] GitHub Container Registry integration
* [x] GitHub Actions workflow
* [x] Self-hosted GitHub Actions runner
* [x] Docker Desktop Kubernetes
* [x] Kubernetes Deployment
* [x] 3 Kubernetes replicas
* [x] Kubernetes Service
* [x] ConfigMap
* [x] Kubernetes Secret demonstration
* [x] Liveness probe
* [x] Readiness probe
* [x] CPU/memory resource configuration

## Skipped / Not Yet Implemented

* [ ] HPA automatic traffic-based scaling
* [ ] Metrics Server
* [ ] Kubernetes Ingress
* [ ] Production cloud Kubernetes cluster
* [ ] Versioned GHCR image deployment
* [ ] Terraform deployment
* [ ] Production-grade secret management

---

# 24. Key DevOps Concepts Demonstrated

This project demonstrates the following concepts:

### Continuous Integration

```text
Git Push
   ↓
GitHub Actions
   ↓
Build
   ↓
Test
   ↓
SonarQube
```

### Containerization

```text
Spring Boot JAR
      ↓
Dockerfile
      ↓
Docker Image
```

### Container Registry

```text
Docker Image
      ↓
GHCR
```

### Container Orchestration

```text
Docker Image
      ↓
Kubernetes Deployment
      ↓
Pods
```

### Service Discovery / Networking

```text
Kubernetes Service
      ↓
Pods
```

### Configuration Management

```text
ConfigMap
      ↓
Application
```

### Secret Management

```text
Secret
      ↓
Application
```

### Application Health Management

```text
Liveness Probe
Readiness Probe
      ↓
Kubernetes
```

### Resource Management

```text
CPU Requests/Limits
Memory Requests/Limits
      ↓
Kubernetes
```

---

# 25. Important Distinction

This project should be described as a **Spring Boot application with a DevOps/Kubernetes deployment pipeline**, rather than claiming it is a microservices application.

The Controller → Service → Repository structure is a layered application architecture.

Kubernetes provides container orchestration, while GitHub Actions provides CI/CD automation.

---

# 26. Final Architecture

```text
                         DEVELOPER
                             |
                             | git push
                             v
                    +------------------+
                    |     GitHub       |
                    +--------+---------+
                             |
                             v
                    +------------------+
                    | GitHub Actions   |
                    +--------+---------+
                             |
              +--------------+--------------+
              |              |              |
              v              v              v
           Maven         SonarQube       Docker
           Tests         + JaCoCo         Build
              |              |              |
              +--------------+--------------+
                             |
                             v
                    +------------------+
                    |      GHCR        |
                    | Docker Registry  |
                    +--------+---------+
                             |
                             v
                    +------------------+
                    | Self-hosted      |
                    | GitHub Runner    |
                    +--------+---------+
                             |
                             | kubectl
                             v
                    +------------------+
                    |    Kubernetes    |
                    | Docker Desktop   |
                    +--------+---------+
                             |
                     +-------+-------+
                     |       |       |
                     v       v       v
                   Pod 1   Pod 2   Pod 3
                     |       |       |
                     +-------+-------+
                             |
                             v
                    +------------------+
                    | Kubernetes       |
                    | Service          |
                    +--------+---------+
                             |
                             v
                       EmployeeHub
```

---

# Conclusion

EmployeeHub demonstrates a complete path from source code to containerized application deployment.

The main flow is:

**GitHub → GitHub Actions → Maven → JaCoCo → SonarQube → Docker → GHCR → Kubernetes → EmployeeHub**

Kubernetes currently maintains three application replicas and provides service-based traffic routing, health checks, configuration management, and resource management.

Render is a separate cloud deployment option triggered through GitHub Actions, while Terraform was explored but intentionally skipped.

```
```
