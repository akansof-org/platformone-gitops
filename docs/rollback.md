# Rollback Options

This note captures the local PlatformOne GitOps rollback options. The default
rule is simple:

```text
Rollback through Git first. Use manual cluster changes only for emergencies.
```

## Method Summary

| Method | Changes Git? | Changes Cluster? | Normal Use |
| --- | --- | --- | --- |
| Git revert | Yes | After Argo CD sync | Preferred rollback path |
| Fix forward | Yes | After Argo CD sync | Preferred when the fix is obvious |
| Argo CD history rollback | No | Yes | Fast operational recovery |
| Manual `kubectl` change | No | Yes | Emergency only |

## First Question

Before choosing a rollback path, identify what is wrong:

```text
Git is correct, cluster is wrong -> drift reconciliation.
Git is wrong, cluster followed it -> rollback or fix Git.
Argo CD/control plane is unhealthy -> recover Argo CD first.
```

## Option 1: Revert The Git Commit

Use this when a committed manifest change introduced bad desired state.

Example:

```bash
git revert <bad-commit-sha>
git push
```

Then refresh and sync the Argo CD Application.

Use this when the previous Git revision was known-good and returning to it is
safer than designing a new fix under pressure.

## Option 2: Fix Forward In Git

Use this when the desired recovery is a small correcting change rather than a
full revert.

Example:

```text
Bad value:  requests.cpu: 24m
Good value: requests.cpu: 25m
```

Commit the correction, push it, refresh Argo CD, and sync.

Use this when the mistake is obvious and the correct value is known.

## Option 3: Argo CD History Rollback

Use this when Argo CD has a previous successful sync revision and you need to
restore that revision quickly.

This can be done from the Argo CD UI:

```text
Application -> History and Rollback -> select known-good revision -> Rollback
```

Important: Argo CD rollback restores the live cluster to a previous synced
revision, but it does not rewrite Git.

That means Git `HEAD` can still contain the bad desired state. After the
rollback, the application may become `OutOfSync` because live state and Git
`HEAD` no longer match.

After using Argo CD rollback, reconcile Git by either reverting the bad commit:

```bash
git revert <bad-commit-sha>
git push
```

or by committing a fix-forward change.

Use Argo CD rollback as an operational recovery tool, not as the final source
of truth rollback.

## Option 4: Temporarily Pause Auto-Sync

Use this only when auto-sync is enabled and Argo CD would keep reapplying a bad
desired state.

For the current local smoke app, auto-sync is disabled, so this is not normally
needed yet.

If auto-sync is enabled later, pause it before investigating a bad deployment
that keeps being reapplied.

## Option 5: Emergency Manual Cluster Change

Use this only during an incident when Git or Argo CD cannot restore service
quickly enough.

Example:

```bash
kubectl scale deployment smoke-echo -n apps --replicas=1
```

After any manual change, update Git immediately so the repository returns to
being the source of truth.

Manual changes create drift. They are not the normal rollback path.

## Local Smoke App Example

During the local GitOps stage, the smoke app was intentionally changed to
violate the `apps` namespace `LimitRange`.

Bad desired state:

```yaml
requests:
  cpu: 24m
```

The namespace guardrail requires:

```yaml
min:
  cpu: 25m
```

Expected behavior:

- Argo CD can sync the manifest.
- Kubernetes rejects pod creation for the new ReplicaSet.
- The Application can show `Synced` but remain `Progressing`.
- Existing healthy pods may keep running.

Recovery:

```yaml
requests:
  cpu: 25m
```

Then commit, push, refresh Argo CD, and sync again.

Expected recovery result:

```text
smoke-echo-local -> Synced / Healthy
deployment/smoke-echo -> Available
```

## Verification Commands

Check Argo CD:

```bash
kubectl get application smoke-echo-local -n gitops
```

Check workload health:

```bash
kubectl get deploy,rs,pods,svc,ingress -n apps \
  -l app.kubernetes.io/name=smoke-echo
```

Inspect rollout or admission failures:

```bash
kubectl describe deployment smoke-echo -n apps
kubectl describe rs -n apps -l app.kubernetes.io/name=smoke-echo
kubectl get events -n apps --sort-by=.lastTimestamp
```

## Lessons

- `Synced` means Argo CD applied the desired manifests.
- `Healthy` means Kubernetes successfully made the workload run.
- A bad Git change should normally be fixed in Git.
- A manual cluster fix must be followed by a Git correction.
- Drift repair and rollback are related, but not the same thing.
