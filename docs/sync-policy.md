# Sync Policy

This note records how PlatformOne decides whether Argo CD should sync
applications automatically in each environment.

## Terms

| Setting | Meaning |
| --- | --- |
| Auto-sync | Argo CD automatically applies Git changes to the cluster. |
| Self-heal | Argo CD automatically corrects live cluster drift back to Git. |
| Prune | Argo CD automatically deletes live resources that were removed from Git. |

These settings are related, but they are not the same decision.

## Current Local Decision

The local smoke app currently enables automated sync and self-heal, but keeps
prune disabled:

```yaml
syncPolicy:
  syncOptions:
    - CreateNamespace=true
  automated:
    prune: false
    selfHeal: true
```

This lives in:

```text
applications/smoke-echo-local.yaml
```

## Why This Is Acceptable Locally

Local is a learning and validation environment. At this point, normal sync,
drift detection, invalid manifest behavior, and rollback options have been
tested enough for the smoke app to use auto-sync safely.

`selfHeal: true` is useful locally because it proves the GitOps contract:

```text
Git remains the desired state, and manual cluster drift is repaired.
```

`prune: false` is intentionally conservative. Resource deletion behavior should
be tested separately before allowing Argo CD to remove live objects
automatically.

## Environment Policy Direction

| Environment | Auto-sync | Self-heal | Prune | Starting Position |
| --- | --- | --- | --- | --- |
| local | Yes | Yes | No | Appropriate for smoke testing and fast feedback. |
| dev | Likely yes | Likely yes | Cautious | Enable after CI validation is reliable. |
| staging | Maybe | Maybe | No initially | Prefer controlled promotion and release validation. |
| production | No initially | No initially | No initially | Start manual until approvals, rollback, and observability are proven. |

## When To Enable Auto-Sync

Auto-sync becomes appropriate when:

- manifests render successfully in CI;
- admission and resource guardrails are understood;
- rollback options are documented;
- ownership is clear;
- Argo CD itself is healthy and adequately resourced;
- the environment can tolerate automatic deployment after merge.

## When To Avoid Auto-Sync

Keep auto-sync disabled when:

- the team is still learning sync, drift, and rollback behavior;
- CI does not validate manifests yet;
- environment ownership is unclear;
- production approval gates are not defined;
- bad desired state could create broad impact.

## Prune Rule

Do not enable prune by default.

Before enabling prune, test what happens when a resource is removed from Git and
confirm that Argo CD deletes only the expected object.

For production-like environments, prune should require stronger confidence in
application boundaries, review controls, and recovery paths.

## Operating Rule

Use this rule of thumb:

```text
Local can automate reconciliation.
Production starts manual.
Prune waits until deletion behavior is explicitly tested.
```

