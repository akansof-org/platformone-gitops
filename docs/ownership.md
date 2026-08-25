# Repository Ownership Model

This note records the initial ownership model for the PlatformOne GitOps
repository.

## Default Owner

The default repository owner is:

```text
@akansof-org/platformone
```

PlatformOne owns the GitOps repository structure, Argo CD bootstrap objects,
environment policy, and the shared delivery model.

## Ownership By Area

| Area | Primary Owner | Review Support | Responsibility |
| --- | --- | --- | --- |
| `appprojects/` | PlatformOne | SRE, Security as needed | Argo CD permission boundaries. |
| `applications/` | PlatformOne | App owner as needed | Argo CD app-to-environment pointers. |
| `applicationsets/` | PlatformOne | SRE as needed | Generated Application patterns when needed. |
| `apps/` | PlatformOne initially | Product/app owners later | Desired state for deployable workloads. |
| `clusters/` | PlatformOne | SRE | Cluster registration and cluster-specific config. |
| `environments/` | PlatformOne | SRE, Security | Environment policy, sync posture, and operating context. |
| `product-registration/` | PlatformOne | Product/app owners | Product ownership and platform onboarding metadata. |
| `docs/` | PlatformOne | Relevant reviewers | Operating notes, decisions, and learning records. |

## CODEOWNERS Baseline

The current `CODEOWNERS` baseline is:

```text
* @akansof-org/platformone

/clusters/ @akansof-org/platformone @akansof-org/sre
/apps/ @akansof-org/platformone
/policies/ @akansof-org/security
/observability/ @akansof-org/sre
```

This means PlatformOne is the default owner, with SRE and Security review added
for specialized operational and security areas.

## Application Ownership Direction

At this stage, PlatformOne owns the `smoke-echo` app because it is a platform
delivery-path test.

As real product workloads are added, ownership should shift:

```text
PlatformOne owns the delivery framework.
Application teams own their application desired state.
SRE reviews reliability-sensitive changes.
Security reviews privileged or policy-sensitive changes.
```

## Environment Ownership Direction

Environment policy remains PlatformOne-owned.

Environment changes include:

- destination namespace changes;
- auto-sync, self-heal, or prune changes;
- AppProject destination/source changes;
- cluster registration changes;
- production promotion rules.

These should receive review from PlatformOne. SRE or Security should be added
when the change affects reliability, access, policy, or production behavior.

## Change Rules

Normal changes should flow through Git:

```text
branch -> commit -> review -> merge -> Argo CD sync
```

Manual cluster changes are allowed only for emergency recovery or controlled
learning exercises. Any manual change must be followed by a Git correction or a
sync back to Git.

## Production Direction

Production is not active in this local stage, but the intended model is:

- production changes require review before merge;
- production starts with manual sync;
- auto-sync is considered only after CI, rollback, observability, and ownership
  are proven;
- emergency changes must be documented and reconciled back to Git.

## Open Questions

- When should product teams become CODEOWNERS for their own app paths?
- Should each product get a separate AppProject or share environment-level
  AppProjects?
- Which changes require SRE review by default?
- Which changes require Security review by default?
- What approval rules should apply before production sync?

