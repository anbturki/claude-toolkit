---
name: manage-secrets
description: Manage .env secrets via GitHub Variables. Use for pulling, pushing, and syncing environment variables.
user-invocable: false
---

# Secrets Management

Environment variables are stored as GitHub Variables (not Secrets) to allow bidirectional sync. The `.env` file is stored as a single variable called `ENV_FILE`.

## Prerequisites

- `gh` CLI authenticated (`gh auth login`)
- Repository access with variable permissions

## Commands

All commands are in the project `Makefile`:

| Command | Description |
|---------|-------------|
| `make secrets` | Pull `.env` from GitHub (default/repo level) |
| `make secrets ENV=staging` | Pull `.env` from a GitHub environment |
| `make secrets-save` | Push `.env` to GitHub (default/repo level) |
| `make secrets-save ENV=staging` | Push `.env` to a GitHub environment |
| `make secrets-create-env ENV=staging` | Create a GitHub environment |
| `make secrets-delete-env ENV=staging` | Delete a GitHub environment |
| `make secrets-list-envs` | List all GitHub environments |

## How It Works

- **Storage**: The entire `.env` file is stored as a single GitHub variable named `ENV_FILE`
- **Scope**: Without `ENV=`, operates at the repository level. With `ENV=<name>`, operates on a specific GitHub environment
- **Auth**: Commands unset `GITHUB_TOKEN` to force `gh` CLI to use its own auth (avoids conflicts with GitHub Actions tokens)

## Adding New Env Vars

When adding a new environment variable:

1. Add to `packages/shared/config/env.ts` (Zod schema)
2. Add to `packages/shared/config/index.ts` (config object) if needed
3. Add to `.env` with the value
4. Run `make secrets-save` to push to GitHub
5. If the var is used in Kubernetes, add it to the relevant deployment YAML in `infrastructure/kubernetes/`
6. If the var is a secret (API keys, passwords), add it to the Kubernetes `backend-secrets` on the cluster

## File Locations

| File | Purpose |
|------|---------|
| `.env` | Local environment variables |
| `packages/shared/config/env.ts` | Zod schema validating all env vars |
| `packages/shared/config/index.ts` | Config object consuming env vars |
| `infrastructure/kubernetes/*/deployment.yaml` | K8s deployment env configuration |

## Kubernetes Secrets

Kubernetes deployments reference secrets from `backend-secrets`:

```yaml
env:
  - name: DATABASE_URL
    valueFrom:
      secretKeyRef:
        name: backend-secrets
        key: DATABASE_URL
```

To update Kubernetes secrets on the cluster:
```bash
kubectl -n confera-cx create secret generic backend-secrets --from-env-file=.env.prod --dry-run=client -o yaml | kubectl apply -f -
```
