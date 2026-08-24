# Local Environment

Local is the laptop/k3d environment used to prove the PlatformOne GitOps
delivery path before introducing additional environments.

## Current Values

| Setting | Value |
| --- | --- |
| Namespace | `apps` |
| Hostname pattern | `*.platformone.local` |
| Smoke app host | `smoke-echo.platformone.local` |
| Sync mode | Manual |
| Auto-sync | Disabled for now |
| Owner | `platform-engineering` |
| Purpose | Local GitOps smoke testing |

## Notes

- Local app-specific values live in each app's `overlays/local/` directory.
- This environment should stay small and explicit while the delivery path is
  being proven.
- Secrets do not belong in this repository.

