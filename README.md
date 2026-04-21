# argocd-examples

A collection of Kubernetes and Argo CD manifests used for teaching and demonstration purposes.

## Repository Structure

```
examples/
├── nginx/
│   ├── nginx-deployment.yaml   # Nginx Deployment (1 replica, namespace: nginx-ns)
│   └── nginx-svc.yaml          # NodePort Service exposing port 30080
└── podinfo/
    └── podinfo.yaml            # Argo CD Application deploying Podinfo via Helm
```

## Examples

### nginx

A basic Kubernetes application consisting of:

- **Deployment** — runs a single `nginx:latest` container in the `nginx-ns` namespace.
- **Service** — exposes the deployment as a `NodePort` on port `30080`.

Apply manually:

```bash
kubectl create namespace nginx-ns
kubectl apply -f examples/nginx/
```

### podinfo

An Argo CD `Application` resource that deploys [Podinfo](https://github.com/stefanprodan/podinfo) from its official Helm chart.

Key configuration:
- Chart version: `6.11.2`
- 2 replicas
- Custom UI color and message
- Automated sync with `prune` and `selfHeal` enabled
- Destination namespace `podinfo-demo` is created automatically

Apply with:

```bash
kubectl apply -f examples/podinfo/podinfo.yaml
```

> Argo CD must be installed and running in the `argocd` namespace for this to work.

## Prerequisites

- A running Kubernetes cluster (e.g. [k3s](https://k3s.io/), [kind](https://kind.sigs.k8s.io/), or [minikube](https://minikube.sigs.k8s.io/))
- `kubectl` configured to point at the cluster
- [Argo CD](https://argo-cd.readthedocs.io/en/stable/getting_started/) installed for the `podinfo` example
