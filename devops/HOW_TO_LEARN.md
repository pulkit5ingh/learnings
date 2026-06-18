# How to Learn This Program

This file is your guide to getting the most out of this DevOps Engineer roadmap.  
Read this before you start anything else.

---

## What This Program Is

This is a **self-paced, project-driven curriculum** to become a production-ready DevOps / Platform Engineer. It is structured around the actual tools and workflows used at companies like Netflix, Airbnb, and most modern tech startups.

There are 9 phases. Each phase has:
- **Topics to study** (concepts, tools, documentation)
- **Assignments (A)** — focused, hands-on practice on one skill
- **Projects (P)** — larger builds that combine multiple skills into something real

You learn by doing. Assignments build understanding. Projects build confidence.

---

## Who This Is For

This program assumes you already know:
- Basic programming in any language (Python preferred for many tools)
- Terminal usage (cd, ls, basic shell commands)
- Git basics (clone, commit, push, pull)

If you are completely new to Linux, spend 1 week on Linux basics first, then come back.

---

## How Long Will This Take

| Phase | Topic | Estimated Time |
|-------|-------|---------------|
| Phase 1 | Linux & Networking | 2–3 weeks |
| Phase 2 | Version Control & CI/CD | 2–3 weeks |
| Phase 3 | Docker & Containers | 2–3 weeks |
| Phase 4 | Kubernetes | 4–5 weeks |
| Phase 5 | Terraform & Ansible | 3–4 weeks |
| Phase 6 | AWS Cloud | 3–4 weeks |
| Phase 7 | Monitoring & Observability | 2–3 weeks |
| Phase 8 | DevSecOps | 2–3 weeks |
| Phase 9 | Advanced (GitOps, SRE) | 3–4 weeks |
| **Total** | | **~8–10 months** |

At 2–3 hours per day: finish in 8–9 months.  
At weekends only (8–10 hours/week): 14–16 months.

Kubernetes (Phase 4) is the steepest learning curve. Budget extra time there.

---

## The Right Order

For each phase, follow this sequence every time:

```
Step 1 — Study the concepts (2–4 days)
Step 2 — Do the assignments in order (1–2 days each)
Step 3 — Build the projects (3–7 days each)
Step 4 — Review what you learned and document it
Step 5 — Move to the next phase
```

Do not skip assignments. Do not do projects before assignments. The assignments are the bridge between theory and real projects.

---

## How to Approach Each Assignment

1. **Read the entire file first.** Know what you are building before writing any command.
2. **Type every command manually.** No copy-pasting. Muscle memory matters in DevOps.
3. **Break things on purpose.** After getting something working, try to break it and understand why.
4. **If stuck for > 30 minutes**, look up the specific concept — not the full solution.
5. **Check every deliverable** before marking the assignment done. Every checkbox matters.

---

## How to Approach Each Project

### Day 1 — Plan
- Read the entire project file before touching anything
- Draw the architecture on paper: boxes, arrows, what connects to what
- Break the project into 5–10 small tasks in order
- Set up a new GitHub repo, virtual environment, and folder structure

### Day 2 to N — Build
- Build one piece at a time
- Get each piece working before connecting it to the next
- Commit to git at the end of every working session
- Do not try to build everything at once

### Final Day — Polish and Publish
- Verify every deliverable checkbox
- Write a README: what it does, how to run it, what you learned
- Push to GitHub — public projects are your portfolio
- Record a short demo if there is a UI (Loom or asciinema)

---

## Tools You Need

Set up everything below before starting Phase 1:

### Accounts (all have free tiers)
- **GitHub** — version control and CI/CD (GitHub Actions)
- **AWS** — cloud platform (use Free Tier, set $10 billing alert immediately)
- **Docker Hub or GHCR** — container registry
- **Cloudflare** — for custom domain DNS (free)
- **PagerDuty** — incident management (free developer account)

### Local Software
```bash
# macOS with Homebrew
brew install git curl wget jq tree

# Kubernetes tools
brew install kubectl helm minikube k9s

# Terraform and cloud tools
brew install terraform aws-cli

# Container tools
brew install --cask docker  # Docker Desktop
brew install trivy cosign conftest

# Ansible
pip install ansible

# Observability tools
brew install argocd

# Terminal improvements
brew install tmux zsh-syntax-highlighting
```

### VS Code Extensions
- Remote - SSH (work on remote servers)
- Docker
- Kubernetes
- HashiCorp Terraform
- YAML
- GitLens
- shellcheck (shell script linting)

---

## AWS Cost Management

AWS charges real money. Follow these rules to avoid surprises:

1. **Set a billing alert immediately**: Billing → Budgets → Create budget → $10/month alert
2. **Destroy resources after each assignment**: `terraform destroy` when done experimenting
3. **Use Free Tier services**: t3.micro EC2, RDS db.t3.micro, Lambda first 1M requests free
4. **Never leave NAT Gateways running**: $0.045/hour — destroy them when not learning
5. **Use `terraform plan` before `apply`**: see what will cost money before creating it
6. **Use minikube/kind for Kubernetes locally**: avoid EKS ($0.10/hour for the control plane)

Expected monthly cost during learning: $5–15 if you are careful. $30–50 if you forget to destroy resources.

---

## Local Kubernetes Options

You do not need a cloud Kubernetes cluster for most of Phase 4:

```bash
# Option A: minikube (simplest)
brew install minikube
minikube start --cpus 4 --memory 8192
minikube addons enable ingress metrics-server

# Option B: kind (faster, good for multi-node)
brew install kind
kind create cluster --config kind-config.yaml

# Option C: k3d (lightest)
brew install k3d
k3d cluster create local --servers 1 --agents 2
```

Use a real cloud cluster (EKS/GKE) only for Phase 6 and Phase 9 projects.

---

## Git Workflow for Every Project

```bash
# Start of every project
git init
git add .
git commit -m "initial project setup"
gh repo create project-name --public
git push -u origin main

# During work
git add specific-file.yaml
git commit -m "add: nginx ingress configuration"

# At end of each working session
git push origin main
```

Use conventional commit messages:
- `feat:` new feature
- `fix:` bug fix
- `docs:` documentation
- `chore:` maintenance
- `refactor:` code restructure

---

## When You Are Stuck

### If a concept is not clicking
- Read the official documentation (always primary source)
- Watch TechWorld with Nana on YouTube — best DevOps explanations
- Build a smaller version first: if Kubernetes is overwhelming, just run one pod
- Draw the architecture — most DevOps concepts click when drawn

### If a project feels overwhelming
- Break it into smaller pieces on paper
- Build the simplest thing that "works" first, even if incomplete
- Check: have you done all the assignments for this phase? If not, do them first

### If something is not working
- Read the error message fully — it usually tells you the exact problem
- Check logs: `kubectl logs`, `journalctl -u service`, `docker logs container`
- Isolate the component — test each piece separately
- Use `--dry-run`, `terraform plan`, `helm template` to check before applying

### If you lose motivation
- Deploy something — even a small thing running in a cluster feels good
- Re-read the project list and pick one that excites you
- Remember: DevOps engineers are among the most in-demand and well-paid engineers

---

## How to Study Concepts

| Resource | Best For |
|----------|---------|
| TechWorld with Nana (YouTube) | All phases — best DevOps explanations on the internet |
| official Kubernetes docs (kubernetes.io) | Phase 4 — the authoritative source |
| Terraform docs (developer.hashicorp.com) | Phase 5 |
| AWS documentation | Phase 6 — verbose but accurate |
| KodeKloud (paid, ~$15/mo) | Hands-on labs with a real cluster in the browser |
| Google SRE Book (free online) | Phase 7, 9 — foundational SRE concepts |
| Linux Command Line book (free online) | Phase 1 |
| Play with Kubernetes (play-with-k8s.com) | Free K8s sandbox for quick experiments |

---

## Certifications (Optional but Useful)

After completing this program, you will be ready for:

| Certification | After Phase | Difficulty |
|--------------|-------------|----------|
| AWS Cloud Practitioner | Phase 6 | Easy |
| AWS Solutions Architect Associate | Phase 6 | Medium |
| CKA (Certified Kubernetes Administrator) | Phase 4-5 | Hard |
| CKAD (Certified Kubernetes App Developer) | Phase 4 | Medium |
| Terraform Associate | Phase 5 | Easy |
| AWS DevOps Engineer Professional | Phase 8-9 | Very Hard |

Do not chase certifications before building projects. Projects come first.

---

## Building Your Portfolio

Every project should go on GitHub. By the end of this program you will have:
- 22 public repositories (one per project)
- Real infrastructure code (Terraform, Helm charts)
- Working CI/CD pipelines
- Documented runbooks

What companies actually want to see in a DevOps portfolio:
1. A working CI/CD pipeline with stages (not just `echo hello`)
2. Kubernetes manifests with security contexts, resource limits, health checks
3. Terraform code that provisions real infrastructure
4. Evidence of monitoring: dashboards, alert rules
5. A README that explains the architecture and how to deploy it

### Good README structure for a DevOps project
1. Architecture diagram (even a text ASCII diagram works)
2. Tech stack and why you chose each tool
3. Prerequisites (what you need installed)
4. How to deploy (exact commands)
5. How to run tests
6. What you learned / what was hard

---

## Milestones — How to Know You Are on Track

After completing each phase, you should be able to do this without help:

| After Phase | You Can Do |
|-------------|-----------|
| Phase 1 | Write a bash script with error handling; debug a network issue with curl/dig/ss |
| Phase 2 | Build a complete GitHub Actions CI pipeline; resolve a git merge conflict |
| Phase 3 | Write a multi-stage Dockerfile; run a multi-service app with Docker Compose |
| Phase 4 | Deploy an app to Kubernetes; write HPA, Ingress, and RBAC; debug with kubectl |
| Phase 5 | Provision AWS VPC + EC2 with Terraform modules; configure a server with Ansible |
| Phase 6 | Design and deploy a 3-tier AWS architecture; explain IAM roles and trust policies |
| Phase 7 | Instrument an app with Prometheus; query PromQL; write alerting rules |
| Phase 8 | Explain and run SAST, DAST, container scanning; configure Vault dynamic secrets |
| Phase 9 | Set up ArgoCD GitOps; define SLOs and error budgets; run a chaos experiment |

If you cannot do the thing in the right column, do not move forward. Revisit the assignments.

---

## Quick Reference — File Naming

| File | What it is |
|------|-----------|
| `index.md` | Full roadmap with all phases |
| `HOW_TO_LEARN.md` | This file |
| `A1.1.md` | Assignment 1 from Phase 1 |
| `A4.3.md` | Assignment 3 from Phase 4 |
| `P2.1.md` | Project 1 from Phase 2 |
| `P9.2.md` | Project 2 from Phase 9 |

Always do assignments before projects within each phase.

---

## The Most Important Rules

**Rule 1: Do not skip assignments to get to projects.**  
Projects feel exciting, assignments feel tedious. The assignments exist because you cannot build a Kubernetes project without understanding Pod → Deployment → Service first.

**Rule 2: Destroy cloud resources after each assignment.**  
A NAT Gateway left running for a month costs $32. That is not a lesson you want to learn.

**Rule 3: Version control everything.**  
Every script, every manifest, every Terraform file goes into Git from day one. This is not optional — it is the job.

**Rule 4: Read error messages.**  
90% of DevOps debugging is reading the error message and acting on it. Most people do not read the full error. Read the full error.

**Rule 5: The docs are always right.**  
When a tutorial conflicts with the official documentation, the docs win. Tutorials go stale. Docs (mostly) do not.

---

## Final Thought

DevOps is a discipline where the gap between knowing and doing is enormous. You can read about Kubernetes for a month and still not know how to debug a CrashLoopBackOff. You can build a working Kubernetes cluster in a day and understand it completely.

The only way through this curriculum is to build things. Break things. Fix things. Build again.

Every time you get an application running in a cluster, or a pipeline deploying automatically, or a dashboard showing real metrics — that is proof of skill. Collect that proof for 9 months and you will be a strong DevOps engineer.

Start with `A1.1.md`. Good luck.
