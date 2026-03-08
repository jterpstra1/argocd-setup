# Argo CD installation and configuration

This repository is used for bootstrapping Argo CD on a Kubernetes cluster.  
It also enables self-management of Argo CD using Argo CD itself.

To ensure separation of concerns three repositories are used to manage the Argo CD setup:

* Argo CD installation and
  configuration: [https://github.com/jterpstra1/argocd-setup](https://github.com/jterpstra1/argocd-setup)
* Argo CD application definitions: [https://github.com/jterpstra1/argocd-apps](https://github.com/jterpstra1/argocd-apps)
* Kubernetes manifests or Helm charts used by Argo CD
  applications: [https://github.com/jterpstra1/argocd-k8s-resources](https://github.com/jterpstra1/argocd-k8s-resources)

## Usage

Apply the production overlay (everything will be deployed in the `argocd` namespace):

```bash
kubectl apply -k argocd-setup/overlays/prod
```

This setup ensures that Argo CD is installed and configured to manage its own resources within the cluster.

Access the Argo CD UI:

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:80
```

## Hetzner k3s Bootstrap (`vdgt-k3s`)

For initial cluster bootstrap on Hetzner k3s, apply the `vdgt-k3s` overlay:

```bash
kubectl apply -k argocd-setup/overlays/vdgt-k3s
```

This overlay installs Argo CD and then uses Argo CD to bootstrap:

* F5 NGINX Ingress Controller (`nginx-ingress`) with `Service.type=LoadBalancer`
* `cert-manager` with CRDs (cluster-wide)
* Let's Encrypt `ClusterIssuer` resources (cluster-wide)

Argo CD ingress is configured for TLS with cert-manager and uses `letsencrypt-prod` by default.

### Certificates

Both `letsencrypt-prod` and `letsencrypt-staging` ClusterIssuers use **DNS-01 via Cloudflare**. HTTP-01 is not used because the cluster sits behind the Cloudflare proxy, which would break the ACME self-check.

The Cloudflare API token must be created manually in the cluster (it is **not** stored in Git):

```bash
kubectl create secret generic cloudflare-api-token-secret \
  -n cert-manager \
  --from-literal=api-token='<CLOUDFLARE_API_TOKEN>'
```

Required Cloudflare API token permissions (scoped to `vdgtconsultancy.be`):
- Zone / Zone / Read
- Zone / DNS / Edit

### Hetzner LoadBalancer

The NGINX ingress `LoadBalancer` service is pinned to Hetzner location `nbg1` via the annotation:

```yaml
load-balancer.hetzner.cloud/location: nbg1
```