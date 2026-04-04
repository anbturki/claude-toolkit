---
name: manage-secrets
description: Manage environment secrets — auto-detects the project's secret management approach and adapts. Supports GitHub Variables, AWS Secrets Manager, SSM, Vault, Doppler, Infisical, Kubernetes, and more.
user-invocable: true
argument-hint: "[pull|push|add|sync|list] [env-name]"
allowed-tools: Read, Glob, Grep, Bash, Agent
---

# Secrets Management

Manage environment variables and secrets. **Adapt to the project's actual setup.**

## Step 1: Detect Project Setup

Before doing anything, discover how this project manages secrets:

```
1. Read CLAUDE.md for secret management conventions
2. Check for Makefile targets: grep -E "secrets|env" Makefile
3. Check for config files: .env, .env.example, docker-compose.yml
4. Check for provider config: .doppler.yaml, .infisical.json, vault-config.hcl
5. Check for CI/CD: .github/workflows/*.yml (look for secret references)
6. Check for infrastructure: kubernetes/, terraform/, infrastructure/
7. Check for package.json scripts related to secrets/env
```

Based on what you find, use the matching approach below.

## Step 2: Execute Using Detected Provider

### If: Makefile with `secrets` targets

The project has custom Makefile commands. Use them as-is:
```bash
make secrets                    # Pull
make secrets-save               # Push
make secrets ENV=staging        # Pull from environment
make secrets-save ENV=staging   # Push to environment
```

### If: GitHub Variables / Secrets (gh CLI)

```bash
# Variables (readable, for non-sensitive config)
gh variable list
gh variable set ENV_FILE < .env
gh variable get ENV_FILE > .env

# Secrets (write-only, for sensitive values)
gh secret list
gh secret set SECRET_NAME < value
gh secret set SECRET_NAME --env staging < value
```

### If: AWS Secrets Manager

```bash
aws secretsmanager get-secret-value --secret-id <name> --query SecretString --output text > .env
aws secretsmanager update-secret --secret-id <name> --secret-string file://.env
aws secretsmanager list-secrets --query 'SecretList[].Name'
```

### If: AWS SSM Parameter Store

```bash
aws ssm get-parameters-by-path --path /app/env/ --with-decryption
aws ssm put-parameter --name /app/env/KEY --value "val" --type SecureString --overwrite
```

### If: HashiCorp Vault

```bash
vault kv get -format=json secret/app/env
vault kv put secret/app/env @.env
```

### If: Doppler

```bash
doppler secrets download --no-file --format env > .env
doppler secrets upload .env
```

### If: Infisical

```bash
infisical export --env=prod > .env
infisical secrets set --env=prod KEY=value
```

### If: Kubernetes Secrets

```bash
kubectl -n <namespace> create secret generic <secret-name> \
  --from-env-file=.env --dry-run=client -o yaml | kubectl apply -f -
kubectl -n <namespace> get secret <secret-name> -o jsonpath='{.data.KEY}' | base64 -d
```

### If: Docker Compose only

Secrets live in `.env` file referenced by `docker-compose.yml`. No remote sync — just edit `.env` directly.

### If: Vercel / Railway / Fly.io

```bash
# Vercel
vercel env pull .env.local
vercel env add KEY

# Railway
railway variables set KEY=value

# Fly.io
fly secrets set KEY=value
fly secrets list
```

## Step 3: Adding a New Variable

When the user wants to add a new env var:

1. **Add to `.env`** (and `.env.example` if it exists)
2. **Add to validation schema** — search for env validation (Zod schema, `z.object`, `env.ts`, `config.ts`)
3. **Add to config object** — if the project wraps env vars in a config module
4. **Push to provider** — using the detected provider's push command
5. **Update deployment config** — if the var needs to reach production:
   - Kubernetes: update deployment YAML or secret
   - Docker Compose: add to `environment:` section
   - CI/CD: add to workflow env or secrets
   - PaaS: use platform CLI to set

## Rules

1. **Detect first, act second** — never assume which provider is in use
2. **Never commit `.env` files** to git
3. **Maintain `.env.example`** with placeholder values
4. **Use the project's established approach** — don't introduce a new provider
5. **Separate secrets from config** — API keys go in secret stores, non-sensitive config can use env files
6. **If unsure, ask** — don't guess which provider to use
