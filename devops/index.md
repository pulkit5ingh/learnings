# DevOps Engineer Roadmap

A structured, project-driven curriculum to become a production-ready DevOps / Platform Engineer.  
9 phases. 36 assignments. 22 projects. ~8–10 months.

---

## Phase 1 — Linux & Networking Fundamentals
**Duration:** 2–3 weeks

### Topics
- Linux file system, permissions, users, groups
- Shell scripting (bash): variables, loops, conditionals, functions
- Process management: ps, top, kill, systemd, cron
- Networking: TCP/IP, DNS, HTTP/HTTPS, SSH, curl, netstat
- Package management: apt, yum, snap
- File I/O, pipes, redirection, grep, awk, sed

### Assignments
- A1.1 — Linux permissions and user management
- A1.2 — Shell scripting: automation scripts
- A1.3 — Networking diagnostics and debugging
- A1.4 — Process management and systemd services

### Projects
- P1.1 — System monitoring dashboard (bash)
- P1.2 — Automated server provisioning script

---

## Phase 2 — Version Control & CI/CD Basics
**Duration:** 2–3 weeks

### Topics
- Git internals: commits, branches, merge, rebase, cherry-pick
- Git workflows: GitFlow, trunk-based development, PR reviews
- GitHub Actions: workflows, jobs, steps, secrets, artifacts
- Semantic versioning, changelog automation
- Artifact management: GitHub Packages, Nexus, JFrog

### Assignments
- A2.1 — Git branching strategies and conflict resolution
- A2.2 — GitHub Actions: first CI pipeline
- A2.3 — Multi-stage pipeline with tests and artifacts
- A2.4 — Semantic versioning and automated changelog

### Projects
- P2.1 — Full CI/CD pipeline for a Node.js or Python app
- P2.2 — Multi-environment deployment pipeline (dev/staging/prod)

---

## Phase 3 — Containers (Docker)
**Duration:** 2–3 weeks

### Topics
- Docker architecture: daemon, images, containers, registries
- Dockerfile best practices: multi-stage builds, layer caching, non-root users
- Docker Compose: multi-service apps, networks, volumes
- Container networking: bridge, host, overlay
- Container security: scanning, least privilege, read-only filesystems
- Private registries: Docker Hub, GitHub Container Registry, ECR

### Assignments
- A3.1 — Dockerfile optimization and multi-stage builds
- A3.2 — Docker Compose: multi-service application
- A3.3 — Container networking and volume management
- A3.4 — Container image security scanning with Trivy

### Projects
- P3.1 — Containerize a full-stack web application
- P3.2 — Private container registry with automated CI/CD push

---

## Phase 4 — Container Orchestration (Kubernetes)
**Duration:** 3–4 weeks

### Topics
- Kubernetes architecture: control plane, worker nodes, etcd
- Core objects: Pod, Deployment, ReplicaSet, Service, Namespace
- Configuration: ConfigMap, Secret, PersistentVolume, PersistentVolumeClaim
- Networking: Ingress, NetworkPolicy, CoreDNS
- Scaling: HPA, VPA, Cluster Autoscaler
- Helm: charts, values, templating, repositories
- RBAC: roles, bindings, service accounts

### Assignments
- A4.1 — Core Kubernetes objects: deploy, scale, rollout
- A4.2 — ConfigMaps, Secrets, and PersistentVolumes
- A4.3 — Ingress, HPA, and resource limits
- A4.4 — Helm chart authoring

### Projects
- P4.1 — Deploy a microservices app to Kubernetes
- P4.2 — Kubernetes cluster with autoscaling and RBAC
- P4.3 — Helm chart for a production-grade application

---

## Phase 5 — Infrastructure as Code (Terraform & Ansible)
**Duration:** 3–4 weeks

### Topics
- Terraform: providers, resources, variables, outputs, state
- Terraform modules: reusable components, module registry
- Remote state: S3 backend, state locking with DynamoDB
- Terraform Cloud / Atlantis for team workflows
- Ansible: inventory, playbooks, tasks, handlers, variables
- Ansible roles, Galaxy, Vault for secrets
- Packer for machine image building

### Assignments
- A5.1 — Terraform: provision VPC and EC2 on AWS
- A5.2 — Terraform modules and remote state management
- A5.3 — Ansible playbooks: configure a web server
- A5.4 — Ansible roles and dynamic inventory

### Projects
- P5.1 — Full AWS infrastructure with Terraform (VPC, ECS, RDS, ALB)
- P5.2 — Configuration management pipeline with Ansible + Packer

---

## Phase 6 — Cloud Platforms (AWS)
**Duration:** 3–4 weeks

### Topics
- AWS compute: EC2, ECS, EKS, Lambda, Fargate
- AWS storage: S3, EBS, EFS, Glacier
- AWS networking: VPC, subnets, security groups, ALB, NLB, Route 53
- AWS database: RDS, Aurora, DynamoDB, ElastiCache
- AWS IAM: users, roles, policies, STS, cross-account
- AWS Serverless: Lambda, API Gateway, SQS, SNS, EventBridge
- AWS cost optimization: reserved instances, spot, Savings Plans
- Multi-region and disaster recovery patterns

### Assignments
- A6.1 — AWS core services: EC2, S3, VPC from CLI
- A6.2 — IAM: least privilege, roles, cross-account access
- A6.3 — Serverless: Lambda + API Gateway + SQS pipeline
- A6.4 — VPC design: subnets, NAT, ALB, security groups

### Projects
- P6.1 — Three-tier web application on AWS (EC2 + RDS + ALB)
- P6.2 — Serverless event-driven backend (Lambda + SQS + DynamoDB)
- P6.3 — Multi-region active-passive disaster recovery setup

---

## Phase 7 — Monitoring, Logging & Observability
**Duration:** 2–3 weeks

### Topics
- Metrics: Prometheus, Grafana, PromQL
- Logging: ELK stack (Elasticsearch, Logstash, Kibana) or Loki + Grafana
- Distributed tracing: OpenTelemetry, Jaeger, Tempo
- Alerting: Alertmanager, PagerDuty, OpsGenie
- SLI, SLO, SLA definitions
- Dashboards and runbooks
- Synthetic monitoring: Blackbox Exporter, Pingdom

### Assignments
- A7.1 — Prometheus + Grafana: instrument and visualize an app
- A7.2 — Centralized logging with Loki or ELK
- A7.3 — Distributed tracing with OpenTelemetry + Jaeger
- A7.4 — Alerting rules and PagerDuty integration

### Projects
- P7.1 — Full observability stack for a microservices application
- P7.2 — Incident management: SLO dashboard + runbook + alerting

---

## Phase 8 — DevSecOps & Compliance
**Duration:** 2–3 weeks

### Topics
- SAST: Semgrep, SonarQube, Bandit
- DAST: OWASP ZAP, Nuclei
- Dependency scanning: Snyk, Dependabot, OWASP Dependency-Check
- Container scanning: Trivy, Grype, Clair
- Secrets management: HashiCorp Vault, AWS Secrets Manager
- Policy as code: OPA, Conftest, Kyverno
- Supply chain security: SBOM, Sigstore/Cosign, SLSA
- Compliance: SOC 2, CIS Benchmarks, Cloud Security Posture Management

### Assignments
- A8.1 — SAST/DAST integration in GitHub Actions
- A8.2 — Container supply chain: scan, sign, verify with Cosign
- A8.3 — HashiCorp Vault: dynamic secrets for an application
- A8.4 — Policy as code with OPA and Conftest

### Projects
- P8.1 — Secure CI/CD pipeline with automated security gates
- P8.2 — Secrets management system with Vault + Kubernetes
- P8.3 — Cloud security posture: CIS benchmark audit and remediation

---

## Phase 9 — Advanced: GitOps, SRE & Platform Engineering
**Duration:** 3–4 weeks

### Topics
- GitOps: principles, ArgoCD, Flux
- SRE: toil reduction, error budgets, runbooks, postmortems
- Chaos engineering: Chaos Monkey, LitmusChaos, Gremlin
- Platform engineering: Internal Developer Platforms, Backstage
- FinOps: cloud cost visibility, tagging strategies, rightsizing
- Multi-cluster management: ArgoCD App of Apps, Cluster API
- Progressive delivery: canary, blue-green, feature flags (Flagger, Argo Rollouts)

### Assignments
- A9.1 — GitOps: deploy and sync with ArgoCD
- A9.2 — SLO definition and error budget tracking
- A9.3 — Chaos engineering: game day on a staging cluster
- A9.4 — Progressive delivery: canary deployment with Argo Rollouts

### Projects
- P9.1 — GitOps platform: multi-environment with ArgoCD App of Apps
- P9.2 — SRE operations kit: SLO dashboard + runbook + postmortem template
- P9.3 — Internal developer platform with self-service app deployment

---

## Master Capstone Projects

**M1 — Production-Ready Kubernetes Platform**  
Build a full production Kubernetes platform: multi-node cluster, Helm-deployed apps, Prometheus/Grafana observability, ArgoCD GitOps, Vault secrets, Trivy scanning, HPA, and Ingress — all provisioned with Terraform.

**M2 — Full DevOps Pipeline from Scratch**  
Take a real open-source project and build everything from scratch: CI with GitHub Actions, Dockerized build, ECS/EKS deployment, Terraform infra, monitoring, alerting, and automated security scanning.

**M3 — Multi-Cloud Disaster Recovery**  
Design and implement a multi-region, cross-cloud (AWS + GCP) disaster recovery system with RTO < 1 hour and RPO < 15 minutes. Includes automated failover, health checks, and chaos testing.

**M4 — Self-Service Developer Platform**  
Build an internal developer platform: service catalog (Backstage), one-click environment provisioning (Terraform Cloud), self-service database access, and a cost dashboard — so developers can deploy without touching infrastructure.

---

## Key Resources

| Resource | Best For |
|----------|---------|
| Linux Command Line (William Shotts, free online) | Phase 1 |
| The DevOps Handbook | Mindset and culture |
| Kubernetes in Action (Manning) | Phase 4 |
| Terraform: Up & Running (Brikman) | Phase 5 |
| AWS Documentation | Phase 6 |
| Site Reliability Engineering (Google, free online) | Phase 9 |
| TechWorld with Nana (YouTube) | All phases — best DevOps YouTube channel |
| KodeKloud | Hands-on labs for K8s, Ansible, Terraform |
| AWS Skill Builder | Free AWS labs |
| Play with Kubernetes (labs.play-with-k8s.com) | Free K8s sandbox |
