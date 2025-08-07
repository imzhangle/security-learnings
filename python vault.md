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
