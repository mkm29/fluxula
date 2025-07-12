# fluxula - Bootstrap Kubernetes clusters with FluxCD

```bash
flux bootstrap github \
  --token-auth \
  --owner=mkm29 \
  --repository=fluxula \
  --branch=main \
  --path=clusters/talos-cluster \
  --personal
```

## Core Components

### 1. Prometheus Operator CRDs Helm Chart

```bash
flux create helmrelease prometheus-operator-crds \
  --interval=30m \
  --chart=prometheus-operator-crds \
  --chart-version=21.0.0 \
  --source=HelmRepository/prometheus-community \
  --export > clusters/talos-cluster/core/prometheus-operator-crds.yaml
```

```bash
flux create helmrelease tetragon \
  --interval=10m \
  --chart=tetragon \
  --chart-version=1.4.0 \
  --source=HelmRepository/cilium.flux-system \
  --target-namespace=tetragon \
  --create-target-namespace \
  --export > clusters/talos-cluster/core/tetragon2.yaml
```
