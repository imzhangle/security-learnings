For working with HashiCorp Vault in Python, here are the most popular and recommended libraries:

## 1. HVAC (Most Popular & Recommended)

**HVAC** is the official Python client library for HashiCorp Vault.

```bash
pip install hvac
```## Library Comparison

### 1. HVAC (Recommended)
```bash
pip install hvac
```

**Pros:**
- Official HashiCorp-endorsed library
- Most comprehensive feature set
- Active development and maintenance
- Excellent documentation
- Supports all Vault auth methods
- Type hints available

**Cons:**
- Larger dependency footprint

### 2. PyVault
```bash
pip install pyvault
```

**Pros:**
- Lightweight
- Simple API
- Good for basic operations

**Cons:**
- Less feature-complete
- Not as actively maintained
- Limited auth method support

### 3. Vault-CLI Python wrapper
```bash
pip install vault-cli
```

**Pros:**
- Wraps the official Vault CLI
- Always up-to-date with Vault features

**Cons:**
- Requires Vault CLI binary
- Less pythonic interface

### 4. Direct HTTP Requests
```bash
pip install requests
```

**Pros:**
- No additional dependencies beyond requests
- Full control over API calls
- Lightweight

**Cons:**
- More boilerplate code
- Manual error handling
- No built-in auth helpers

## Quick Start with HVAC

```python
import hvac

# Initialize client
client = hvac.Client(
    url='https://vault.example.com:8200',
    token='your-vault-token'
)

# Verify authentication
if client.is_authenticated():
    print("Successfully authenticated!")

# List KV secrets
secrets = client.secrets.kv.v2.list_secrets(
    path='',
    mount_point='secret'
)
print(secrets['data']['keys'])

# Read a secret
secret = client.secrets.kv.v2.read_secret_version(
    path='myapp/database',
    mount_point='secret'
)
print(secret['data']['data'])
```

## For GitHub Actions

Add to your workflow:
```yaml
- name: Install Vault client
  run: pip install hvac

- name: Access Vault secrets
  env:
    VAULT_ADDR: ${{ secrets.VAULT_ADDR }}
    VAULT_TOKEN: ${{ secrets.VAULT_TOKEN }}
  run: python vault_script.py
```

**HVAC is the recommended choice** for most use cases due to its comprehensive feature set, official support, and excellent documentation. Would you like me to show you specific examples for your use case?



You're right! The code is in the artifact above. Here it is directly:

```python
import hvac
import os

# Configuration
VAULT_ADDR = os.environ.get('VAULT_ADDR', 'https://vault.example.com:8200')
GITHUB_TOKEN = os.environ.get('GITHUB_TOKEN', 'your_github_token')

# Initialize Vault client
client = hvac.Client(url=VAULT_ADDR)

# Authenticate with GitHub token
try:
    print("Authenticating with GitHub token...")
    auth_response = client.auth.github.login(token=GITHUB_TOKEN)
    
    # Set the Vault token
    client.token = auth_response['auth']['client_token']
    
    print("Authentication successful!")
    print(f"Policies: {auth_response['auth']['policies']}")
    
except Exception as e:
    print(f"Authentication failed: {e}")
    exit(1)

# Now you can use the client to access Vault
try:
    # List secrets at pbs/t24dops in kv-v2 mount
    response = client.secrets.kv.v2.list_secrets(
        path='pbs/t24dops',
        mount_point='kv-v2'
    )
    
    if response and 'data' in response:
        secrets = response['data']['keys']
        print(f"\nSecrets found in pbs/t24dops:")
        for secret in secrets:
            print(f"  - {secret}")
    else:
        print("No secrets found")

except Exception as e:
    print(f"Error accessing secrets: {e}")
```

**Install the dependency:**
```bash
pip install hvac
```

**Set your environment variables and run:**
```bash
export VAULT_ADDR="https://your-vault-server.com:8200"
export GITHUB_TOKEN="your_github_token_here"
python script.py
```
