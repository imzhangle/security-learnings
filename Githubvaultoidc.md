To use GitHub Actions’ **OIDC token** to authenticate with HashiCorp Vault, you’ll follow a process where Vault trusts GitHub’s OIDC issuer and maps your GitHub workflow identity to a Vault role.

Here’s the step-by-step:

---

## **1. Enable JWT/OIDC Auth in Vault**

```bash
vault auth enable jwt
```

---

## **2. Configure Vault to Trust GitHub’s OIDC Issuer**

GitHub’s OIDC token issuer URL is:

```
https://token.actions.githubusercontent.com
```

Example Vault config:

```bash
vault write auth/jwt/config \
    oidc_discovery_url="https://token.actions.githubusercontent.com" \
    bound_issuer="https://token.actions.githubusercontent.com"
```

---

## **3. Create a Vault Role for GitHub Workflows**

You can restrict by repository, branch, or other claims in the GitHub token.

Example:

```bash
vault write auth/jwt/role/my-github-role \
    bound_audiences="https://vault.example.com" \
    bound_subject="repo:my-org/my-repo:ref:refs/heads/main" \
    user_claim="sub" \
    policies="my-policy" \
    ttl="15m"
```

* `bound_subject` matches the GitHub OIDC `sub` claim format:

  ```
  repo:<org>/<repo>:ref:<ref>
  ```

  For example: `repo:octocat/hello-world:ref:refs/heads/main`
* `bound_audiences` should match the `aud` claim you request in the GitHub token.

---

## **4. Request the GitHub OIDC Token in Your Workflow**

GitHub exposes an OIDC endpoint via `$ACTIONS_ID_TOKEN_REQUEST_URL`.

Example GitHub Actions YAML:

```yaml
name: Vault OIDC Auth

on:
  push:
    branches: [ main ]

jobs:
  vault-auth:
    runs-on: ubuntu-latest
    permissions:
      id-token: write
      contents: read
    steps:
      - name: Get GitHub OIDC token
        id: oidc
        run: |
          TOKEN=$(curl -s -H "Authorization: Bearer $ACTIONS_ID_TOKEN_REQUEST_TOKEN" \
                     "$ACTIONS_ID_TOKEN_REQUEST_URL?audience=https://vault.example.com" | jq -r '.value')
          echo "token=$TOKEN" >> $GITHUB_OUTPUT

      - name: Exchange OIDC token for Vault token
        run: |
          VAULT_TOKEN=$(curl -s --request POST \
            --data "{\"jwt\": \"${{ steps.oidc.outputs.token }}\", \"role\": \"my-github-role\"}" \
            https://vault.example.com/v1/auth/jwt/login | jq -r '.auth.client_token')
          echo "Vault token: $VAULT_TOKEN"
```

---

## **5. Use the Vault Token**

Now you can use `$VAULT_TOKEN` to access secrets or perform Vault operations within the workflow.

Example:

```bash
vault kv get secret/my-secret
```

---

✅ **Key Points to Remember**

* **`permissions.id-token: write`** is required for the GitHub Action to request an OIDC token.
* The `audience` parameter in the request **must match** Vault’s `bound_audiences` in your role.
* The `bound_subject` claim lets you lock the Vault role to a specific repo or branch.

---

If you want, I can give you a **minimal working GitHub Actions + Vault OIDC example** that you could just copy-paste and run. It would handle both the Vault side and workflow side. Would you like me to prepare that?
