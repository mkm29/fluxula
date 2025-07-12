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

