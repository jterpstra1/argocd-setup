# Copilot Instructions

## Repository Purpose
- This repository bootstraps Argo CD and its baseline cluster configuration.
- `argocd-setup/overlays/prod` is the production overlay.
- `argocd-setup/overlays/vdgt-k3s` is the Hetzner k3s bootstrap overlay.

## Bootstrap Lifecycle (vdgt-k3s)
- Bootstrap starts by applying `argocd-setup/overlays/vdgt-k3s` with `kubectl apply -k`.
- Argo CD then self-manages the same overlay (`argocd-self-management` points to `overlays/vdgt-k3s`).
- Argo CD bootstraps cluster-wide ingress and certificates through three Applications:
  - `bootstrap-ingress-nginx`
  - `bootstrap-cert-manager`
  - `bootstrap-cluster-issuers`
- `bootstrap-ingress-nginx` installs the F5 NGINX Ingress Controller (`nginx-ingress` chart) and exposes it via a Hetzner `LoadBalancer` service.
- Argo CD ingress terminates TLS at NGINX and receives HTTP internally.

## Conventions
- Keep app/project segregation in `overlays/*/projects` in sync with `namespaces.yaml` and `argocd-cmd-params-cm.patch.yaml`.
- Prefer explicit, pinned chart versions for bootstrap components.
