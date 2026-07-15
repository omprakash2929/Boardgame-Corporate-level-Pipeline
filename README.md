# 🎲 Boardgame — Corporate-Level CI/CD Pipeline

A production-style end-to-end DevOps pipeline for a Java (Spring Boot / Maven) Boardgame web application — covering source control, code quality, security scanning, containerization, Kubernetes deployment, and full observability.

![Java](https://img.shields.io/badge/Java-Maven-red)
![Jenkins](https://img.shields.io/badge/CI%2FCD-Jenkins-D24939)
![Docker](https://img.shields.io/badge/Container-Docker-2496ED)
![Kubernetes](https://img.shields.io/badge/Orchestration-Kubernetes-326CE5)
![SonarQube](https://img.shields.io/badge/Code%20Quality-SonarQube-4E9BCD)
![Trivy](https://img.shields.io/badge/Security-Trivy-1904DA)
![Prometheus](https://img.shields.io/badge/Monitoring-Prometheus-E6522C)
![Grafana](https://img.shields.io/badge/Dashboards-Grafana-F46800)

---

## 📖 Overview

This project demonstrates a complete DevSecOps workflow: every code push is automatically built, tested, scanned for bugs and vulnerabilities, containerized, pushed to a registry, deployed to Kubernetes, and monitored in real time — with zero manual steps in between.

**Pipeline flow:**

```
Git Push → Jenkins CI → Compile & Test → Trivy FS Scan → SonarQube Analysis
        → Quality Gate → Build (Maven) → Publish to Nexus
        → Docker Build & Tag → Trivy Image Scan → Push to Docker Registry
        → Deploy to Kubernetes → Verify Deployment
        → Prometheus + Grafana (live monitoring)
```

---

## 🧰 Tech Stack

| Layer | Tool |
|---|---|
| Source Control | Git & GitHub |
| CI/CD Orchestration | Jenkins |
| Build Tool | Maven |
| Code Quality | SonarQube |
| Security Scanning (filesystem & image) | Trivy |
| Artifact Repository | Nexus |
| Containerization | Docker |
| Orchestration | Kubernetes |
| Metrics Collection | Prometheus, Node Exporter, Blackbox Exporter |
| Dashboards | Grafana |

---

## 🔁 CI/CD Pipeline (Jenkins)

The Jenkins pipeline runs the following stages on every build:

1. **Tool Install** — provisions JDK, Maven, and other build tools
2. **Git Checkout** — pulls the latest source from GitHub
3. **Compile** — compiles the Java source
4. **Test** — runs the unit test suite
5. **File System Scan** — Trivy scans the filesystem for known vulnerabilities (`trivy-fs-report.html`)
6. **SonarQube Analysis** — static code analysis for bugs, code smells, and security hotspots
7. **Quality Gate** — build fails if SonarQube quality gate isn't passed
8. **Build** — packages the app into a JAR with Maven
9. **Publish to Nexus** — uploads the build artifact to the Nexus repository
10. **Build & Tag Docker Image** — builds and tags the Docker image
11. **Docker Image Scan** — Trivy scans the built image for vulnerabilities (`trivy-image-report.html`)
12. **Push Docker Image** — pushes the image to the container registry
13. **Deploy to Kubernetes** — applies manifests to the cluster (`deployment-service.yaml`)
14. **Verify Deployment** — confirms rollout succeeded
15. **Post Actions** — notifications / cleanup

> Pipeline status, stage timings, and history are visible on the Jenkins **Stage View**.

---

## 🛡️ Code Quality & Security

- **SonarQube** enforces a quality gate on every build — tracking bugs, vulnerabilities, code smells, duplications, and test coverage.
- **Trivy** scans both the filesystem (pre-build) and the final Docker image (post-build), with HTML reports archived as Jenkins build artifacts.

| Metric | Result |
|---|---|
| Quality Gate | ✅ Passed |
| Vulnerabilities | 0 |
| Security Review | A |
| Maintainability | A |

---

## 📦 Project Structure

```
Boardgame/
├── src/                        # Application source code
├── k8s/                        # Kubernetes manifests
│   └── deployment-service.yaml
├── monitoring/                 # Observability stack
│   ├── docker-compose.yml
│   ├── prometheus/
│   │   └── prometheus.yml
│   ├── blackbox_exporter/
│   │   └── blackbox.yml
│   └── grafana/
│       ├── provisioning/
│       │   ├── datasources/
│       │   └── dashboards/
│       └── dashboards/
├── Dockerfile
├── Jenkinsfile
├── pom.xml
├── mvnw / mvnw.cmd
├── sonar-project.properties
└── README.md
```

---

## 📊 Monitoring & Observability

The `monitoring/` stack runs alongside the app via Docker Compose:

- **Node Exporter** — host-level metrics (CPU, memory, disk)
- **Blackbox Exporter** — HTTP probes for uptime and response health
- **Prometheus** — scrapes and stores all metrics
- **Grafana** — visual dashboards for probe status, HTTP duration, and target health

### Run the monitoring stack

```bash
cd monitoring
docker compose up -d
```

| Service | URL |
|---|---|
| Prometheus | `http://localhost:9090` |
| Grafana | `http://localhost:3000` |
| Blackbox Exporter | `http://localhost:9115` |
| Node Exporter metrics | `http://localhost:9100/metrics` |

All Prometheus scrape targets (Prometheus itself, Blackbox probes) report **UP** status, and Grafana's Blackbox Exporter dashboard tracks probe duration, HTTP status, and SSL expiry for each monitored endpoint.

---

## 🚀 Getting Started

### Prerequisites
- Java 17+ and Maven
- Docker & Docker Compose
- A running Kubernetes cluster (Minikube, EKS, etc.)
- Jenkins with SonarQube, Trivy, Docker, and Kubernetes plugins configured

### Build locally

```bash
git clone https://github.com/omprakash2929/Boardgame-Corporate-level-Pipeline.git
cd Boardgame-Corporate-level-Pipeline
./mvnw clean install
```

### Run with Docker

```bash
docker build -t boardgame:latest .
docker run -p 8080:8080 boardgame:latest
```

### Deploy to Kubernetes

```bash
kubectl apply -f k8s/deployment-service.yaml
kubectl get pods
kubectl get svc
```

---

## 📈 Pipeline Screenshots

> Add your screenshots here, e.g.:
> - Jenkins Stage View
> - SonarQube Quality Gate
> - Grafana Blackbox Exporter dashboard
> - Prometheus Target Health

---

## 🙋 Author

**Om Prakash**
[GitHub](https://github.com/omprakash2929)

---

## 📄 License

This project is open source and available for learning/demo purposes.
