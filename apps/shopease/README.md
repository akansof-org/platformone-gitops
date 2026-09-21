# ShopEase deployment configuration

This directory is the authoritative home for ShopEase deployment desired state on PlatformOne.

```text
base/             Common workload manifests
overlays/local/   Local k3d deployment configuration
```

Status: directory scaffolding only. No deployable Kustomization or Argo CD registration has been added yet.

Application source, Dockerfiles, CI, tests, and product documentation live in `shopease-app` under `04-products/` in the local workspace. Upstream deployment examples remain in that checkout as reference material.

Argo CD Application and AppProject definitions belong in this repository's `applications/` and `appprojects/` directories. Release changes will update the image digests here after application images have been built, tested, scanned, and published.
