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


Got it — here’s a **complete GitHub Actions workflow** that will:

1. Request a GitHub OIDC token
2. Check if it’s empty
3. Verify it’s a JWT
4. Decode and print the claims (payload) so you can compare with your Vault `bound_audiences` and `bound_subject`

```yaml
name: Inspect GitHub OIDC Token

on:
  workflow_dispatch: # Run manually from Actions tab

jobs:
  inspect-oidc:
    runs-on: ubuntu-latest
    permissions:
      id-token: write
      contents: read

    steps:
      - name: Request OIDC token from GitHub
        id: get_token
        run: |
          echo "Requesting OIDC token..."
          TOKEN=$(curl -s -H "Authorization: Bearer $ACTIONS_ID_TOKEN_REQUEST_TOKEN" \
                        "$ACTIONS_ID_TOKEN_REQUEST_URL?audience=https://vault.example.com" \
                  | jq -r '.value')

          echo "OIDC_TOKEN=$TOKEN" >> $GITHUB_ENV

      - name: Check if token is empty
        run: |
          if [ -z "$OIDC_TOKEN" ]; then
            echo "❌ OIDC token is EMPTY!"
            exit 1
          else
            echo "✅ Token length: ${#OIDC_TOKEN}"
          fi

      - name: Verify token format (JWT)
        run: |
          IFS='.' read -ra PARTS <<< "$OIDC_TOKEN"
          if [ "${#PARTS[@]}" -ne 3 ]; then
            echo "❌ Not a valid JWT format"
            exit 1
          else
            echo "✅ Looks like a JWT with ${#PARTS[@]} parts"
          fi

      - name: Decode JWT header
        run: |
          HEADER=$(echo "$OIDC_TOKEN" | cut -d '.' -f1 | base64 -d 2>/dev/null || echo "Base64 decode failed")
          echo "🔹 JWT Header:"
          echo "$HEADER" | jq .

      - name: Decode JWT payload (claims)
        run: |
          PAYLOAD=$(echo "$OIDC_TOKEN" | cut -d '.' -f2 | base64 -d 2>/dev/null || echo "Base64 decode failed")
          echo "🔹 JWT Payload (claims):"
          echo "$PAYLOAD" | jq .

      - name: Decode JWT signature (raw, not verified)
        run: |
          SIGNATURE=$(echo "$OIDC_TOKEN" | cut -d '.' -f3)
          echo "🔹 JWT Signature (raw base64url):"
          echo "$SIGNATURE"
```

---

### How to use:

1. Save this as `.github/workflows/inspect-oidc.yml`
2. Push to your repo
3. Run it manually from **Actions → Inspect GitHub OIDC Token → Run workflow**
4. In the logs, look at **"JWT Payload (claims)"** to see the exact `aud` and `sub` values
   → These are what you match in Vault with `bound_audiences` and `bound_subject`

---

If you want, I can now give you the **Vault role config** that matches exactly the claims you’ll see from this workflow, so your token will stop coming back as `null`. That way we can close the loop.





Great! You already have a solid workflow to inspect the GitHub OIDC token. Now you want to **use that OIDC token** in a Python script (`test_vault.py`) to authenticate to **HashiCorp Vault**.

Here’s how to extend your workflow to:

1. Pass the OIDC token (`OIDC_TOKEN`) as an environment variable.
2. Run `python test_vault.py` that uses it to log in to Vault.

---

### ✅ Updated Workflow with Python Step

```yaml
name: Inspect GitHub OIDC Token and Test Vault

on:
  workflow_dispatch: # Run manually from Actions tab

jobs:
  inspect-oidc:
    runs-on: ubuntu-latest
    permissions:
      id-token: write  # Required for requesting the OIDC token
      contents: read   # Required to checkout code

    steps:
      - name: Checkout Repository
        uses: actions/checkout@v4

      - name: Request OIDC token from GitHub
        id: get_token
        run: |
          echo "Requesting OIDC token..."
          TOKEN=$(curl -s -H "Authorization: Bearer $ACTIONS_ID_TOKEN_REQUEST_TOKEN" \
                        "$ACTIONS_ID_TOKEN_REQUEST_URL?audience=https://vault.example.com" \
                  | jq -r '.value')

          if [ -z "$TOKEN" ]; then
            echo "❌ Failed to retrieve OIDC token"
            exit 1
          fi

          echo "OIDC_TOKEN=$TOKEN" >> $GITHUB_ENV
          echo "✅ Retrieved OIDC token (length: ${#TOKEN})"

      - name: Check if token is empty
        run: |
          if [ -z "$OIDC_TOKEN" ]; then
            echo "❌ OIDC token is EMPTY!"
            exit 1
          else
            echo "✅ Token length: ${#OIDC_TOKEN}"
          fi

      - name: Verify token format (JWT)
        run: |
          IFS='.' read -ra PARTS <<< "$OIDC_TOKEN"
          if [ "${#PARTS[@]}" -ne 3 ]; then
            echo "❌ Not a valid JWT format"
            exit 1
          else
            echo "✅ Looks like a JWT with ${#PARTS[@]} parts"
          fi

      - name: Decode JWT header
        run: |
          HEADER=$(echo "$OIDC_TOKEN" | cut -d '.' -f1 | base64 -d 2>/dev/null || echo "Base64 decode failed")
          echo "🔹 JWT Header:"
          echo "$HEADER" | jq .

      - name: Decode JWT payload (claims)
        run: |
          PAYLOAD=$(echo "$OIDC_TOKEN" | cut -d '.' -f2 | base64 -d 2>/dev/null || echo "Base64 decode failed")
          echo "🔹 JWT Payload (claims):"
          echo "$PAYLOAD" | jq .

      - name: Run Python script to authenticate to Vault
        env:
          VAULT_ADDR: https://your-vault-address.example.com  # Change to your Vault URL
          VAULT_ROLE: my-github-oidc-role                     # Must match role in Vault
          OIDC_TOKEN: ${{ env.OIDC_TOKEN }}                  # Pass token securely
        run: |
          python3 -m pip install --upgrade pip
          pip install hvac requests

          python test_vault.py
```

---

### ✅ Example `test_vault.py` Script

Make sure this file exists in your repo root (or adjust path accordingly):

```python
import os
import hvac

# Load environment variables
vault_addr = os.getenv("VAULT_ADDR")
vault_role = os.getenv("VAULT_ROLE")
oidc_token = os.getenv("OIDC_TOKEN")

if not vault_addr:
    raise Exception("VAULT_ADDR environment variable is missing")
if not vault_role:
    raise Exception("VAULT_ROLE environment variable is missing")
if not oidc_token:
    raise Exception("OIDC_TOKEN environment variable is missing")

# Connect to Vault
client = hvac.Client(url=vault_addr)

# Authenticate using OIDC role and GitHub's OIDC token
result = client.auth.oidc.login(
    role=vault_role,
    jwt=oidc_token
)

# Set the token so you can make calls
client.token = result['auth']['client_token']

print("🎉 Successfully authenticated to Vault!")

# Example: Read a secret
# secret = client.secrets.kv.v2.read_secret_version(path="myapp/config")
# print("Secret:", secret['data']['data'])
```

> 🔐 Ensure your Vault role (`my-github-oidc-role`) is configured to accept tokens issued to `https://vault.example.com` as the audience.

---

### ✅ Vault OIDC Role Configuration (Reminder)

Ensure Vault has the role set with correct audience:

```bash
vault write auth/oidc/role/my-github-oidc-role \
    bound_subject="repo:your-org/your-repo:ref:refs/heads/main" \
    user_claim="sub" \
    policies="default,myapp" \
    ttl=1h \
    allowed_redirect_uris="http://localhost:8200/oidc/callback" \
    bound_audiences="https://vault.example.com"
```

> 🔁 The `audience` in the OIDC request **must match** `bound_audiences` in the Vault role.

---

### ✅ Final Notes

- Replace:
  - `https://vault.example.com` → your actual Vault audience.
  - `https://your-vault-address.example.com` → your Vault API endpoint.
  - `my-github-oidc-role` → your actual Vault OIDC role name.
- Make sure your Vault server can reach `https://token.actions.githubusercontent.com`.
- Use `hvac` library: `pip install hvac`

---

Let me know if you want to:
- Auto-inject Vault address/role via GitHub secrets.
- Use TLS/client certs with Vault.
- Handle errors or debug login failures.

You're all set for secure, secretless Vault access! 🔐🚀
