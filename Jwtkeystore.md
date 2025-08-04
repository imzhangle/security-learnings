Perfect — you now have:

> 🔐 Multiple **CSRs** (Certificate Signing Requests) and **PKCS#8 private keys**, with signed certificates, and want to **embed them all into one `.p12` (PKCS12)** keystore with different aliases.

This is very doable — let me walk you through it clearly.

---

## ✅ Overview: What You Have

For each certificate, you have:

* `key1.key`: **Private key** in PKCS#8
* `key1.crt`: **Signed certificate** (from CA or self-signed)
* Same for `key2.key`, `key2.crt`, etc.

---

## 🧰 Tools Required

* `openssl`
* `keytool`

---

## 🛠 Step-by-Step: Merge multiple key/cert pairs into one `.p12`

### 🧩 1. Convert key + cert to temporary `.p12` files

You’ll convert each key + cert pair into its own `.p12` bundle using `openssl`.

**Important**: Ensure your key is in unencrypted PKCS#8 format.

#### For key1:

```bash
openssl pkcs12 -export \
  -inkey key1.key \
  -in key1.crt \
  -name key1 \
  -out key1.p12 \
  -password pass:changeit
```

#### For key2:

```bash
openssl pkcs12 -export \
  -inkey key2.key \
  -in key2.crt \
  -name key2 \
  -out key2.p12 \
  -password pass:changeit
```

Repeat for each key/cert pair.

---

### 🏗 2. Create a combined `.p12` keystore using `keytool`

Now, import each `.p12` into one unified keystore.

#### Start with key1:

```bash
keytool -importkeystore \
  -destkeystore jwt-multi.p12 \
  -deststoretype PKCS12 \
  -deststorepass changeit \
  -srckeystore key1.p12 \
  -srcstoretype PKCS12 \
  -srcstorepass changeit \
  -alias key1
```

#### Add key2:

```bash
keytool -importkeystore \
  -destkeystore jwt-multi.p12 \
  -deststoretype PKCS12 \
  -deststorepass changeit \
  -srckeystore key2.p12 \
  -srcstoretype PKCS12 \
  -srcstorepass changeit \
  -alias key2
```

Repeat for more keys if needed.

---

## ✅ Result

You now have a **single `.p12` keystore (`jwt-multi.p12`)** with multiple key entries:

```bash
keytool -list -keystore jwt-multi.p12 -storetype PKCS12 -storepass changeit
```

Output:

```
key1, PrivateKeyEntry, ...
key2, PrivateKeyEntry, ...
```

Each alias represents a separate JWT signing key.

---

## 📦 Deploy in Gravitee AM via Helm

1. Create a Kubernetes secret:

```bash
kubectl create secret generic jwt-keystore \
  --from-file=jwt-multi.p12 \
  --from-literal=keystore-password=changeit \
  -n gravitee
```

2. In Helm `values.yaml`, configure:

```yaml
am:
  gateway:
    extraVolumes:
      - name: jwt-keystore
        secret:
          secretName: jwt-keystore
    extraVolumeMounts:
      - name: jwt-keystore
        mountPath: /opt/gravitee/security/
        readOnly: true
    env:
      - name: gravitee.jwt.secret.keystore.path
        value: "/opt/gravitee/security/jwt-multi.p12"
      - name: gravitee.jwt.secret.keystore.type
        value: "PKCS12"
      - name: gravitee.jwt.secret.keystore.password
        valueFrom:
          secretKeyRef:
            name: jwt-keystore
            key: keystore-password
      - name: gravitee.jwt.secret.keystore.keys[0].kid
        value: "key1"
      - name: gravitee.jwt.secret.keystore.keys[0].certAlias
        value: "key1"
      - name: gravitee.jwt.secret.keystore.keys[0].alg
        value: "RS256"
      - name: gravitee.jwt.secret.keystore.keys[0].use
        value: "sig"
      - name: gravitee.jwt.secret.keystore.keys[1].kid
        value: "key2"
      - name: gravitee.jwt.secret.keystore.keys[1].certAlias
        value: "key2"
      - name: gravitee.jwt.secret.keystore.keys[1].alg
        value: "RS256"
      - name: gravitee.jwt.secret.keystore.keys[1].use
        value: "sig"
```

---

## 🧠 Tip: PKCS#8 Note

If your keys are in **PKCS#8 encrypted format**, you may need to convert them to unencrypted PKCS#8 before `openssl pkcs12` can use them:

```bash
openssl pkcs8 -in enc-key.key -out unenc-key.key -nocrypt
```

---

Would you like a script to automate this full process? Or help testing it in Gravitee?
