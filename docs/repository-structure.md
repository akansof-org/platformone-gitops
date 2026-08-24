# GitOps Repository Structure

This note captures the intended direction for structuring the PlatformOne
GitOps repository. It is a working guide, not a final design. We will refine it
as more applications, environments, and promotion rules are added.

## Purpose

The repository should make four things clear:

- What applications exist.
- What environments exist.
- What Argo CD owns.
- What is local-only test material.

The main rule is to keep concerns separate:

```text
AppProject = permission boundary
Application = Argo CD pointer
apps/ = Kubernetes workload desired state
environments/ = environment policy and context
docs/ = operating notes and decisions
```

## Current Direction

Use this shape as the baseline:

```text
platformone-gitops/
  appprojects/
  applications/
  applicationsets/
  apps/
  clusters/
  environments/
  product-registration/
  docs/
```

## Directory Roles

### `appprojects/`

Stores Argo CD `AppProject` resources.

Use this for permission and ownership boundaries:

```text
local-platform.yaml
dev-platform.yaml
staging-platform.yaml
prod-platform.yaml
```

An `AppProject` answers:

```text
What sources, destinations, and resource kinds are allowed?
```

### `applications/`

Stores Argo CD `Application` resources.

Use this for app-to-environment deployment pointers:

```text
smoke-echo-local.yaml
mastermeds-api-local.yaml
```

An `Application` answers:

```text
Which repo path should Argo CD watch, and where should it deploy it?
```

Keep this simple at first. A flat layout is fine until the number of
applications or environments makes grouping necessary.

### `applicationsets/`

Stores Argo CD `ApplicationSet` resources when one definition needs to generate
multiple Applications.

Do not use this too early. Prefer explicit `Application` manifests until the
pattern is proven.

### `apps/`

Stores deployable application manifests.

Use one folder per app:

```text
apps/
  smoke-echo/
  mastermeds-api/
```

For each app, prefer a base/overlay shape:

```text
apps/
  smoke-echo/
    base/
    overlays/
      local/
      dev/
      staging/
      prod/
```

`base/` holds the common workload shape. `overlays/<environment>/` holds the
environment-specific differences.

### `clusters/`

Stores cluster-level registration or cluster-specific GitOps configuration when
needed.

Do not use this as a dumping ground for application manifests. If a manifest is
for an application workload, it belongs under `apps/`.

### `environments/`

Stores environment context and policy.

Use this to describe things like:

- target cluster
- namespaces
- auto-sync policy
- approval expectations
- environment owners
- promotion rules
- special local-only constraints

At this stage, this can be documentation-first:

```text
environments/
  local/
    README.md
  dev/
    README.md
```

### `product-registration/`

Stores metadata that connects products or teams to PlatformOne GitOps.

Use this when a product needs to be registered with ownership, environment, or
deployment metadata before application manifests are added.

### `docs/`

Stores operating notes, decisions, and explanations that support the GitOps
repository.

Examples:

```text
docs/repository-structure.md
docs/sync-and-drift.md
docs/rollback.md
docs/ownership.md
```

## Minimal Local Stage

For the current local GitOps stage, the minimum useful structure is:

```text
platformone-gitops/
  appprojects/
    local-platform.yaml
  applications/
    smoke-echo-local.yaml
  apps/
    smoke-echo/
      base/
      overlays/
        local/
  environments/
    local/
      README.md
  docs/
    repository-structure.md
```

That is enough to support the current delivery-path actions without pretending
that dev, staging, and production are already fully designed.

## Open Questions

- When should `applications/` move from flat files to environment folders?
- Which environment should first use auto-sync?
- Should AppProjects be manually bootstrapped or managed by a root Application?
- What belongs in `clusters/` versus `environments/`?
- When should ApplicationSets replace explicit Applications?

