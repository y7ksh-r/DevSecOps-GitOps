# DevSecOps GitOps Repository – Kubernetes Manifests

This repository contains the **GitOps layer** of my end-to-end **DevSecOps project**.  
It defines the **desired state of the Kubernetes cluster**, including application deployments, security policies, rollouts, autoscaling, remediation, and chaos experiments, using **ArgoCD** and **Kustomize**.

This repository **does not build images** and **does not provision infrastructure**.  
It consumes outputs from CI/CD pipelines and infrastructure repositories and represents how the cluster **should look at all times**.

All changes are applied **only via Git commits** and continuously enforced by ArgoCD.

---

## What This Repository Manages

Using GitOps principles, this repository controls:

- Application deployments (Go, Node.js, Python)
- Environment-specific overlays (dev, prod)
- Ingress configuration
- Progressive delivery (Argo Rollouts)
- Horizontal Pod Autoscaling (HPA)
- Admission control policies (Kyverno)
- Auto-remediation using Kubernetes CronJobs
- Chaos engineering experiments
- Namespace management

This enforces **declarative, auditable, and reproducible** Kubernetes deployments.

---

## Repository Structure

```text
├── applications/      # ArgoCD Application definitions
├── base/              # Base Kubernetes manifests (environment-agnostic)
├── overlays/          # Environment-specific overlays (dev, prod, rollout)
├── kyverno/           # Admission control policies
├── namespaces/        # Namespace definitions
├── tests/             # Policy and load testing manifests
└── README.md
```

---

## ArgoCD Applications

```
applications/
├── apps-dev.yaml            # App-of-apps for dev environment
├── go-app.yaml
├── node-app.yaml
├── python-app.yaml
├── go-app-rollout.yaml      # Rollout-based application
└── kyverno-policies.yaml
```

These files define ArgoCD Applications, not Kubernetes workloads.
They instruct ArgoCD what to deploy, from where, and into which namespace.

---

## Base Manifests

```
base/
├── go-app/
├── node-app/
└── python-app/
```

Each base contains:

- Deployment
- Service
- Kustomization

---

## Environment Overlays

### Dev Environment

```
overlays/dev/
├── ingress.yaml
├── go-app/
├── node-app/
├── python-app/
├── remediation/
└── chaos/
```

Dev overlays add:

- Image tags from CI
- Resource requests and limits
- Ingress configuration
- Auto-remediation CronJob
- Chaos engineering experiments

### Progressive Delivery (Dev Rollout)

```
overlays/dev-rollout/
└── go-app/
```

This overlay replaces a standard Deployment with:

- Argo Rollout
- Canary deployment strategy
- HPA integration
- Used to demonstrate progressive delivery and autoscaling.

### Production Overlay

```
overlays/prod/
```

Production overlays reuse the same base manifests but apply:

- Environment-specific patches
- Production-safe configuration
- No chaos experiments

### Admission Control – Kyverno

```
kyverno/
├── base/
│   ├── allow-only-ecr.yaml
│   ├── disallow-privileged.yaml
│   └── require-resources.yaml
└── overlays/dev/
```

Enforced policies:

- Only images from Amazon ECR
- No privileged containers
- Mandatory CPU and memory limits
- Policies are enforced at admission time, not after deployment.

###Auto-Remediation

```
overlays/dev/remediation/
```

Includes:

- Kubernetes CronJob
- RBAC (ServiceAccount, Role, RoleBinding)

Purpose:

- Detect and clean failed pods
- Demonstrate GitOps-managed remediation
- No manual intervention required

### Chaos Engineering

```
overlays/dev/chaos/
```

Includes:

- Chaos RBAC
- Pod-delete experiments
- LitmusChaos-compatible manifests

Purpose:

- Validate application resilience
- Observe system behavior under failure
- Demonstrate controlled chaos in dev only

### Namespaces

```
namespaces/
├── dev-namespace.yaml
└── prod-namespace.yaml
```

Namespaces are GitOps-managed to ensure:

- Consistent isolation
- Policy enforcement

### Testing Manifests

```
tests/
├── kyverno/
└── load/
```

Used to:

- Validate Kyverno policies (negative testing)
- Generate load for autoscaling demonstrations

---

## Why This Repository Matters

This repository demonstrates:

- Real GitOps practices
- Clear separation of concerns
- Policy-driven Kubernetes security
- Progressive delivery and resilience testing
- Zero-drift Kubernetes environments

---

## Project Context

This GitOps repository is part of a larger end-to-end DevSecOps project, which also includes:

- Infrastructure repository (Terraform + AWS EKS)
- CI/CD repository (security scans, image build & push)
- Runtime security and observability stack
