Great question! You're asking:

> **How can GitHub Actions use different Vault roles per environment (e.g., staging, production), and how does Vault know which role to use?**

Let’s break this down clearly.

---

### ✅ Goal: Use Different Vault Roles Based on Environment

You want:
- `staging` → use `github-actions-staging` role in Vault
- `production` → use `github-actions-prod` role in Vault

And you want **Vault to trust the environment context** from GitHub Actions securely.

---

### 🔐 How Vault Knows the Role: It Doesn’t — You Choose the Role in the Workflow

Vault **does not automatically know** which role to use. Instead, **your GitHub Actions workflow explicitly tells Vault which role to assume** during authentication.

So, you control the role at the workflow level — and can vary it by branch, environment, or job.

---

### ✅ Strategy: Use Different Roles per Environment in Separate Jobs

Here’s how to structure your workflow to use different roles per environment:

```yaml
# .github/workflows/deploy.yml

name: Deploy with Vault

on:
  push:
    branches:
      - main
      - staging

permissions:
  id-token: write
  contents: read

jobs:
  deploy-staging:
    if: github.ref == 'refs/heads/staging'
    runs-on: ubuntu-latest
    environment: staging
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Get secrets from Vault (staging role)
        uses: hashicorp/vault-action@v3
        with:
          url: https://vault.example.com
          method: jwt
          jwtGithubWorkflow: true
          role: github-actions-staging  # 👈 Staging role
          secrets: |
            secret/data/staging/db-password db_password

      - name: Deploy to Staging
        run: echo "Using: $VAULT_SECRETS_secret_data_staging_db_password_db_password"

  deploy-prod:
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    environment: production
    permissions:
      id-token: write
      contents: read
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Get secrets from Vault (prod role)
        uses: hashicorp/vault-action@v3
        with:
          url: https://vault.example.com
          method: jwt
          jwtGithubWorkflow: true
          role: github-actions-prod  # 👈 Production role
          secrets: |
            secret/data/prod/db-password db_password

      - name: Deploy to Production
        run: echo "Deploying with prod secrets..."
```

---

### 🔐 How Vault Validates the Role

When the `vault-action` runs:

1. GitHub issues an **OIDC token** containing claims like:
   - `repository: owner/repo`
   - `ref`: `refs/heads/main` or `refs/heads/staging`
   - `environment`: if using GitHub Environments
   - `actor`: who triggered it

2. The action sends:
   - This JWT
   - The `role` name (`github-actions-staging` or `github-actions-prod`)

3. Vault checks:
   - Is the JWT signed by GitHub?
   - Does the `role` exist?
   - Does the role allow this repo/branch/environment?

---

### ✅ Secure Role Configuration in Vault

You can make roles **context-aware** using **bound claims**.

#### Example: Restrict role by branch

```bash
# Create staging role - only allows staging branch
vault write auth/jwt/role/github-actions-staging \
    bound_issuer="https://token.actions.githubusercontent.com" \
    user_claim="sub" \
    bound_subject="repo:myorg/myrepo:ref:refs/heads/staging" \
    token_policies="staging-secrets" \
    ttl="10m"
```

```bash
# Create prod role - only allows main branch
vault write auth/jwt/role/github-actions-prod \
    bound_issuer="https://token.actions.githubusercontent.com" \
    user_claim="sub" \
    bound_subject="repo:myorg/myrepo:ref:refs/heads/main" \
    token_policies="prod-secrets" \
    ttl="5m"
```

> ✅ This ensures that **only workflows running on `main`** can assume the `github-actions-prod` role.

> 🔐 The `bound_subject` enforces branch-level security.

---

### 🛠️ Alternative: Use GitHub Environment in Subject

If you use GitHub Environments (like `production`), you can bind to that:

```bash
vault write auth/jwt/role/github-actions-prod \
    bound_issuer="https://token.actions.githubusercontent.com" \
    bound_claims_type=string \
    bound_claims='{"environment":"production"}' \
    token_policies="prod-secrets"
```

Then in GitHub Actions:

```yaml
env:
  VAULT_ROLE: github-actions-${{ github.environment }}
```

And use dynamically (requires scripting):

```yaml
- name: Login to Vault
  run: |
    curl -s -X POST $VAULT_ADDR/v1/auth/jwt/login \
      -d @- <<EOF
    {
      "jwt": "${{ id_token }}",
      "role": "${{ env.VAULT_ROLE }}"
    }
    EOF
```

> ⚠️ The `vault-action` doesn’t support dynamic roles directly, so for full dynamic control, you may need `curl`.

---

### ✅ Summary: How Vault Knows the Role

| Question | Answer |
|--------|--------|
| ❓ How does Vault know which role to use? | ❗ It doesn't — the **workflow specifies the role**. |
| ❓ How is it secure? | ✅ Vault **validates the OIDC token** and **checks role bindings** (e.g., branch, repo, env). |
| ❓ Can I have per-env roles? | ✅ Yes — use **different roles per job**, with **bound claims** for security. |
| ❓ Best practice? | ✅ Smallest privilege: one role per env with minimal policies. |

---

### ✅ Recommended Vault Role Pattern

| GitHub Branch | GitHub Env | Vault Role | Vault Policy | Bound Subject/Claims |
|--------------|------------|------------|--------------|------------------------|
| `staging`    | `staging`  | `gh-staging` | `staging-ro` | `repo:org/repo:ref:refs/heads/staging` |
| `main`       | `production` | `gh-prod` | `prod-rw` | `{"environment":"production"}` |

---

Let me know if you want a full example using `curl` + dynamic role selection based on `github.environment`!


# .github/workflows/test-vault-list.yml

name: Test Vault - List Secrets

on:
  workflow_dispatch:  # Allow manual run
  push:
    branches:
      - main

permissions:
  id-token: write
  contents: read

jobs:
  test-vault:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Login to Vault and List Secrets
        uses: hashicorp/vault-action@v3
        with:
          url: https://your-vault-address.com
          caCertificate: ${{ secrets.VAULT_CA_CERT }} # Optional: for self-signed certs
          method: jwt
          jwtGithubWorkflow: true
          role: github-actions-test
          # We won't read secrets, just list
          # But action requires at least one 'secrets' line to trigger auth
          secrets: |
            secret/data/test/placeholder placeholder # dummy to trigger login

      - name: List secrets in 'test/' folder
        run: |
          # Use VAULT_TOKEN from vault-action
          export VAULT_TOKEN=${{ env.VAULT_TOKEN }}

          # List secrets under secret/test/
          vault list secret/metadata/test/
