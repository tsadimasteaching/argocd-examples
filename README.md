# argocd-examples

A collection of Kubernetes and Argo CD manifests used for teaching and demonstration purposes.

## Repository Structure

```
examples/
├── nginx/                         # Raw K8s manifests — simple Nginx app
│   ├── nginx-deployment.yaml
│   └── nginx-svc.yaml
├── podinfo/                       # ArgoCD Application — Helm, automated sync
│   └── podinfo.yaml
└── fastapi-vue/                   # ArgoCD Application — Kustomize, automated sync
    └── fastapi-vue-app.yaml
```

---

## Examples

### 1. nginx — raw manifests

A minimal Kubernetes application. Start here to understand Deployments and Services before adding Argo CD.

| Resource | Details |
|---|---|
| Deployment | `nginx:latest`, 1 replica, namespace `nginx-ns` |
| Service | NodePort on `30080` |

```bash
kubectl create namespace nginx-ns
kubectl apply -f examples/nginx/
```

---

### 2. podinfo — ArgoCD + Helm, automated sync

An Argo CD `Application` that deploys [Podinfo](https://github.com/stefanprodan/podinfo) directly from its Helm chart repository. **Automated sync** is enabled: any drift from Git is corrected automatically without manual intervention.

```bash
kubectl apply -f examples/podinfo/podinfo.yaml
```

| Setting | Value |
|---|---|
| Chart | `podinfo` `6.11.2` |
| Replicas | 2 |
| Namespace | `podinfo-demo` (auto-created) |
| Sync | Automated, prune + selfHeal |

---

### 4. fastapi-vue — ArgoCD + Kustomize, automated sync

An Argo CD `Application` that deploys a full **FastAPI + Vue.js + PostgreSQL** stack from the [cloud-platforms-fastapi-vue](https://github.com/tsadimasteaching/cloud-platforms-fastapi-vue) repository using **Kustomize**.

Argo CD auto-detects Kustomize when it finds a `kustomization.yaml` file in the target path — no extra configuration needed.

```bash
kubectl apply -f examples/fastapi-vue/fastapi-vue-app.yaml
```

#### What gets deployed

The `k8s/` folder of the source repo contains:

```
k8s/
├── kustomization.yaml          # Root Kustomize config — orchestrates all resources
│                               # and generates ConfigMaps from .env files
├── pull-secret-sa.yaml         # ServiceAccount for pulling images from GHCR
├── certmanager/
│   └── ClusterIssuer.yaml      # Let's Encrypt issuer (cert-manager)
├── db/
│   ├── postgres-deployment.yaml  # PostgreSQL 16, uses PVC for persistence
│   ├── postgres-pvc.yaml         # 100Mi PVC (microk8s-hostpath StorageClass)
│   └── postgres-svc.yaml         # ClusterIP service on port 5432
├── fastapi/
│   ├── fastapi-deployment.yaml   # FastAPI backend (GHCR image), config from ConfigMap
│   ├── fastapi-svc.yaml          # ClusterIP service on port 5000
│   ├── fastapi-traefik.yaml      # Traefik Ingress (HTTP)
│   ├── fastapi-traefik-ssl.yaml  # Traefik Ingress (HTTPS + cert-manager TLS)
│   └── fastapi.env               # DATABASE_URL, SECRET_KEY → becomes fastapi-config ConfigMap
└── vue/
    ├── vue-deployment.yaml       # Vue.js frontend, config from ConfigMap
    ├── vue-svc.yaml              # ClusterIP service on port 8080
    ├── vue-ingress.yaml          # Standard Ingress
    └── vue.env                   # VUE_APP_API_URL → becomes vue-config ConfigMap
```

#### How Kustomize is used here

The `kustomization.yaml` uses two Kustomize features:

- **`resources`** — lists every manifest file to include (no base/overlays in this setup)
- **`configMapGenerator`** — reads `.env` files and generates `ConfigMap` objects automatically

```yaml
configMapGenerator:
  - name: fastapi-config
    envs:
      - fastapi/fastapi.env
  - name: vue-config
    envs:
      - vue/vue.env

generatorOptions:
  disableNameSuffixHash: true   # keeps the ConfigMap name stable across updates
```

When Argo CD syncs, it runs `kustomize build k8s/` and applies the rendered output — including the generated ConfigMaps — to the cluster.

| Setting | Value |
|---|---|
| Source repo | `cloud-platforms-fastapi-vue` @ `HEAD` |
| Source path | `k8s/` |
| Tool | Kustomize (auto-detected) |
| Namespace | `fastapi-vue` (auto-created) |
| Sync | Automated, prune + selfHeal |

> **Prerequisites for this example:** cert-manager and Traefik must be installed in the cluster. A `github-pull-secret` Secret must exist in the destination namespace for the image pull to succeed.

---

## Sync Policy Summary

| Example | Tool | Sync |
|---|---|---|
| nginx | Raw manifests | `kubectl apply` |
| podinfo | Helm | Automated |
| fastapi-vue | Kustomize | Automated |

---

## Prerequisites

- A running Kubernetes cluster (e.g. [k3s](https://k3s.io/), [kind](https://kind.sigs.k8s.io/), or [minikube](https://minikube.sigs.k8s.io/))
- `kubectl` configured to point at the cluster
- [Argo CD](https://argo-cd.readthedocs.io/en/stable/getting_started/) installed in the `argocd` namespace

Install Argo CD:

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```
