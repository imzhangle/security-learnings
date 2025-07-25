Here's how to improve your DevOps process using HashiCorp Vault for enhanced security and automation:

## 1. Vault Integration Architecture

```yaml
# vault-integration-architecture.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: vault-agent-config
  labels:
    app: gravitee-am

  vault-agent.hcl: |
    exit_after_auth = true
    pid_file = "/home/vault/.pid"
    
    auto_auth {
      method {
        type = "kubernetes"
        mount_path = "auth/kubernetes"
        config = {
          role = "gravitee-role"
        }
      }
      
      sink {
        type = "file"
        config = {
          path = "/home/vault/.vault-token"
        }
      }
    }
    
    template_config {
      exit_on_retry_failure = true
      static_secret_render_interval = "5m"
    }
    
    template {
      source = "/etc/vault/templates/jwt-cert.tpl"
      destination = "/opt/gravitee/certificates/jwt/jwt-keystore.p12"
      perms = 0400
    }
    
    template {
      source = "/etc/vault/templates/tls-cert.tpl"
      destination = "/opt/gravitee/certificates/tls/tls.crt"
      perms = 0400
    }
    
    template {
      source = "/etc/vault/templates/tls-key.tpl"
      destination = "/opt/gravitee/certificates/tls/tls.key"
      perms = 0400
    }
    
    template {
      source = "/etc/vault/templates/passwords.tpl"
      destination = "/opt/gravitee/secrets/passwords.env"
      perms = 0400
    }

  jwt-cert.tpl: |
    {{ with secret "pki/cert/jwt-signing" }}
    {{ .Data.certificate }}
    {{ end }}

  tls-cert.tpl: |
    {{ with secret "pki/cert/https-server" }}
    {{ .Data.certificate }}
    {{ end }}

  tls-key.tpl: |
    {{ with secret "pki/cert/https-server" }}
    {{ .Data.private_key }}
    {{ end }}

  passwords.tpl: |
    TLS_KEYSTORE_PASSWORD={{ with secret "kv/gravitee/tls" }}{{ .Data.data.keystore_password }}{{ end }}
    TLS_KEY_PASSWORD={{ with secret "kv/gravitee/tls" }}{{ .Data.data.key_password }}{{ end }}
    JWT_KEYSTORE_PASSWORD={{ with secret "kv/gravitee/jwt" }}{{ .Data.data.keystore_password }}{{ end }}
    JWT_KEY_PASSWORD={{ with secret "kv/gravitee/jwt" }}{{ .Data.data.key_password }}{{ end }}
    MONGODB_USERNAME={{ with secret "kv/gravitee/database" }}{{ .Data.data.username }}{{ end }}
    MONGODB_PASSWORD={{ with secret "kv/gravitee/database" }}{{ .Data.data.password }}{{ end }}
    ADMIN_PASSWORD={{ with secret "kv/gravitee/admin" }}{{ .Data.data.password }}{{ end }}
```

## 2. Enhanced Deployment with Vault Agent Sidecar

```yaml
# gravitee-am-vault-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: gravitee-am-management-api
  labels:
    app: gravitee-am
    component: management-api
spec:
  replicas: 1
  selector:
    matchLabels:
      app: gravitee-am
      component: management-api
  template:
    metadata:
      labels:
        app: gravitee-am
        component: management-api
      annotations:
        vault.hashicorp.com/agent-inject: "true"
        vault.hashicorp.com/agent-inject-status: "update"
        vault.hashicorp.com/role: "gravitee-role"
        vault.hashicorp.com/agent-pre-populate-only: "true"
        
        # JWT Certificate
        vault.hashicorp.com/agent-inject-secret-jwt-keystore: "pki/issue/jwt-signing"
        vault.hashicorp.com/agent-inject-template-jwt-keystore: |
          {{- with secret "pki/issue/jwt-signing" -}}
          {{ .Data.certificate }}
          {{- end -}}
        
        # TLS Certificates
        vault.hashicorp.com/agent-inject-secret-tls-cert: "pki/issue/https-server"
        vault.hashicorp.com/agent-inject-template-tls-cert: |
          {{- with secret "pki/issue/https-server" -}}
          {{ .Data.certificate }}
          {{- end -}}
        
        vault.hashicorp.com/agent-inject-secret-tls-key: "pki/issue/https-server"
        vault.hashicorp.com/agent-inject-template-tls-key: |
          {{- with secret "pki/issue/https-server" -}}
          {{ .Data.private_key }}
          {{- end -}}
        
        # Passwords
        vault.hashicorp.com/agent-inject-secret-passwords: "kv/data/gravitee/all"
        vault.hashicorp.com/agent-inject-template-passwords: |
          {{- with secret "kv/data/gravitee/all" -}}
          TLS_KEYSTORE_PASSWORD={{ .Data.data.tls_keystore_password }}
          TLS_KEY_PASSWORD={{ .Data.data.tls_key_password }}
          JWT_KEYSTORE_PASSWORD={{ .Data.data.jwt_keystore_password }}
          JWT_KEY_PASSWORD={{ .Data.data.jwt_key_password }}
          MONGODB_USERNAME={{ .Data.data.mongodb_username }}
          MONGODB_PASSWORD={{ .Data.data.mongodb_password }}
          ADMIN_PASSWORD={{ .Data.data.admin_password }}
          {{- end -}}
    spec:
      serviceAccountName: gravitee-vault-sa
      securityContext:
        runAsNonRoot: true
        runAsUser: 1001
        fsGroup: 1001
        runAsGroup: 1001
      
      initContainers:
      - name: cert-setup
        image: busybox:1.35
        command:
        - sh
        - -c
        - |
          echo "Setting up certificates from Vault..."
          mkdir -p /opt/gravitee/certificates/{jwt,tls}
          cp /vault/secrets/jwt-keystore /opt/gravitee/certificates/jwt/jwt-keystore.p12
          cp /vault/secrets/tls-cert /opt/gravitee/certificates/tls/tls.crt
          cp /vault/secrets/tls-key /opt/gravitee/certificates/tls/tls.key
          chmod 400 /opt/gravitee/certificates/*/*
          ls -la /opt/gravitee/certificates/*/
        volumeMounts:
        - name: certificates
          mountPath: /opt/gravitee/certificates
        - name: vault-secrets
          mountPath: /vault/secrets
      
      containers:
      - name: gravitee-am-management-api
        image: graviteeio/am-management-api:3.20
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 8092
          name: http
        - containerPort: 8443
          name: https
        
        env:
        # Source all passwords from Vault-injected file
        - name: TLS_KEYSTORE_PASSWORD
          valueFrom:
            secretKeyRef:
              name: gravitee-static-secrets
              key: tls-keystore-password
        - name: TLS_KEY_PASSWORD
          valueFrom:
            secretKeyRef:
              name: gravitee-static-secrets
              key: tls-key-password
        - name: JWT_KEYSTORE_PASSWORD
          valueFrom:
            secretKeyRef:
              name: gravitee-static-secrets
              key: jwt-keystore-password
        - name: JWT_KEY_PASSWORD
          valueFrom:
            secretKeyRef:
              name: gravitee-static-secrets
              key: jwt-key-password
        - name: MONGODB_USERNAME
          valueFrom:
            secretKeyRef:
              name: gravitee-static-secrets
              key: mongodb-username
        - name: MONGODB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: gravitee-static-secrets
              key: mongodb-password
        - name: ADMIN_PASSWORD
          valueFrom:
            secretKeyRef:
              name: gravitee-static-secrets
              key: admin-password
        
        volumeMounts:
        - name: certificates
          mountPath: /opt/gravitee/certificates
          readOnly: true
        - name: config-volume
          mountPath: /opt/gravitee/config
        - name: logs-volume
          mountPath: /opt/gravitee/logs
        
        resources:
          requests:
            memory: "512Mi"
            cpu: "250m"
          limits:
            memory: "1Gi"
            cpu: "500m"
        
        livenessProbe:
          httpGet:
            path: /management/health
            port: 8092
          initialDelaySeconds: 120
          periodSeconds: 30
        
        readinessProbe:
          httpGet:
            path: /management/health
            port: 8092
          initialDelaySeconds: 30
          periodSeconds: 10
      
      volumes:
      - name: certificates
        emptyDir:
          medium: Memory
      - name: vault-secrets
        emptyDir:
          medium: Memory
      - name: config-volume
        configMap:
          name: gravitee-config
      - name: logs-volume
        emptyDir: {}
```

## 3. Vault Policy and Role Configuration

```hcl
# vault-policies.hcl

# Policy for Gravitee application
path "pki/issue/jwt-signing" {
  capabilities = ["create", "update"]
}

path "pki/issue/https-server" {
  capabilities = ["create", "update"]
}

path "kv/data/gravitee/*" {
  capabilities = ["read"]
}

path "kv/data/gravitee/all" {
  capabilities = ["read"]
}

path "auth/token/lookup-self" {
  capabilities = ["read"]
}

path "auth/token/renew-self" {
  capabilities = ["update"]
}

path "sys/mounts" {
  capabilities = ["read"]
}
```

```bash
#!/bin/bash
# vault-setup.sh

# Create policies
vault policy write gravitee-policy vault-policies.hcl

# Enable Kubernetes auth method
vault auth enable kubernetes

# Configure Kubernetes auth
vault write auth/kubernetes/config \
  token_reviewer_jwt="$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)" \
  kubernetes_host="https://$KUBERNETES_PORT_443_TCP_ADDR:443" \
  kubernetes_ca_cert=@/var/run/secrets/kubernetes.io/serviceaccount/ca.crt

# Create role for Gravitee
vault write auth/kubernetes/role/gravitee-role \
  bound_service_account_names=gravitee-vault-sa \
  bound_service_account_namespaces=gravitee-project \
  policies=gravitee-policy \
  ttl=1h

# Enable PKI secrets engine for certificates
vault secrets enable -path=pki pki

# Configure PKI for JWT certificates
vault write pki/roles/jwt-signing \
  allowed_domains="gravitee.local" \
  allow_subdomains=true \
  max_ttl="8760h"

# Configure PKI for HTTPS certificates
vault write pki/roles/https-server \
  allowed_domains="gravitee-am.local" \
  allow_subdomains=true \
  max_ttl="8760h"

# Enable KV secrets engine
vault secrets enable -path=kv kv-v2

# Store passwords in Vault
vault kv put kv/gravitee/all \
  tls_keystore_password="secure-tls-keystore-password" \
  tls_key_password="secure-tls-key-password" \
  jwt_keystore_password="secure-jwt-keystore-password" \
  jwt_key_password="secure-jwt-key-password" \
  mongodb_username="gravitee_user" \
  mongodb_password="secure-mongodb-password" \
  admin_password="secure-admin-password"
```

## 4. CI/CD Pipeline Integration

```yaml
# .gitlab-ci.yml
stages:
  - validate
  - build
  - test
  - deploy
  - verify

variables:
  VAULT_ADDR: $VAULT_ADDRESS
  KUBE_NAMESPACE: $OPENSHIFT_PROJECT

before_script:
  - export VAULT_TOKEN=$(vault login -token-only -method=github token=$GITHUB_TOKEN)
  - oc login --token=$OPENSHIFT_TOKEN --server=$OPENSHIFT_SERVER

validate_vault_secrets:
  stage: validate
  script:
    - |
      echo "Validating Vault connectivity..."
      vault status
    - |
      echo "Validating required secrets exist..."
      vault kv get kv/gravitee/all
      vault read pki/roles/jwt-signing
      vault read pki/roles/https-server
  only:
    - merge_requests
    - master

build_and_deploy:
  stage: deploy
  script:
    - |
      echo "Deploying Gravitee AM with Vault integration..."
      oc apply -f k8s/service-account.yaml
      oc apply -f k8s/configmaps.yaml
      oc apply -f k8s/deployment.yaml
    - |
      echo "Waiting for deployment..."
      oc rollout status deployment/gravitee-am-management-api
  only:
    - master

certificate_rotation:
  stage: deploy
  script:
    - |
      echo "Rotating certificates via Vault..."
      # Request new certificates from Vault
      vault write pki/issue/jwt-signing common_name="jwt.gravitee.local"
      vault write pki/issue/https-server common_name="gravitee-am.local"
    - |
      echo "Restarting deployment to pick up new certificates..."
      oc rollout restart deployment/gravitee-am-management-api
      oc rollout restart deployment/gravitee-am-gateway
  only:
    - schedules  # Run on schedule for certificate rotation

verify_deployment:
  stage: verify
  script:
    - |
      echo "Verifying Vault integration..."
      POD_NAME=$(oc get pods -l app=gravitee-am -o jsonpath='{.items[0].metadata.name}')
      oc exec $POD_NAME -- ls -la /opt/gravitee/certificates/
    - |
      echo "Testing API connectivity..."
      oc port-forward service/gravitee-am-management-api 8092:8092 &
      sleep 10
      curl -f http://localhost:8092/management/health || exit 1
  only:
    - master
```

## 5. Automated Certificate Rotation

```yaml
# certificate-rotation-cronjob.yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: gravitee-cert-rotation
spec:
  schedule: "0 2 * * 0"  # Every Sunday at 2 AM
  jobTemplate:
    spec:
      template:
        spec:
          serviceAccountName: gravitee-vault-sa
          containers:
          - name: cert-rotation
            image: hashicorp/vault:latest
            env:
            - name: VAULT_ADDR
              value: "https://vault:8200"
            command:
            - sh
            - -c
            - |
              # Renew certificates in Vault
              vault write pki/issue/jwt-signing common_name="jwt.gravitee.local" ttl="8760h"
              vault write pki/issue/https-server common_name="gravitee-am.local" ttl="8760h"
              
              # Trigger deployment restart to pick up new certificates
              kubectl rollout restart deployment/gravitee-am-management-api
              kubectl rollout restart deployment/gravitee-am-gateway
              
              echo "Certificate rotation completed"
            volumeMounts:
            - name: vault-token
              mountPath: /var/run/secrets/kubernetes.io/serviceaccount
              readOnly: true
          volumes:
          - name: vault-token
            projected:
              sources:
              - serviceAccountToken:
                  path: token
                  expirationSeconds: 7200
                  audience: vault
          restartPolicy: OnFailure
```

## 6. Monitoring and Alerting

```yaml
# monitoring-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: gravitee-monitoring
  labels:
    app: gravitee-am

  prometheus-rules.yaml: |
    groups:
    - name: gravitee.vault
      rules:
      - alert: VaultTokenExpiry
        expr: vault_token_ttl_seconds < 3600
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "Vault token for Gravitee will expire soon"
          description: "Vault token TTL is below 1 hour"
      
      - alert: CertificateExpiry
        expr: vault_pki_cert_expiry_seconds < 604800  # 7 days
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "SSL certificate will expire soon"
          description: "Certificate for {{ $labels.common_name }} expires in less than 7 days"

  grafana-dashboard.json: |
    {
      "dashboard": {
        "title": "Gravitee AM with Vault",
        "panels": [
          {
            "title": "Vault Token Status",
            "type": "graph",
            "targets": [
              {
                "expr": "vault_token_ttl_seconds",
                "legendFormat": "Token TTL"
              }
            ]
          },
          {
            "title": "Certificate Expiry",
            "type": "graph", 
            "targets": [
              {
                "expr": "vault_pki_cert_expiry_seconds",
                "legendFormat": "{{common_name}} expiry"
              }
            ]
          }
        ]
      }
    }
```

## 7. Enhanced Security Service Account

```yaml
# service-account.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: gravitee-vault-sa
  labels:
    app: gravitee-am

---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: gravitee-vault-role
  labels:
    app: gravitee-am
rules:
- apiGroups: [""]
  resources: ["pods", "pods/exec"]
  verbs: ["get", "list"]
- apiGroups: [""]
  resources: ["secrets"]
  verbs: ["get", "list"]
- apiGroups: ["apps"]
  resources: ["deployments"]
  verbs: ["get", "patch"]

---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: gravitee-vault-rolebinding
  labels:
    app: gravitee-am
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: gravitee-vault-role
subjects:
- kind: ServiceAccount
  name: gravitee-vault-sa
```

## Key Improvements with Vault Integration:

### **Security Enhancements:**
- ✅ **Zero Secret Storage**: No secrets stored in Kubernetes
- ✅ **Dynamic Secrets**: Certificates and passwords generated on-demand
- ✅ **Automatic Rotation**: Built-in certificate and secret rotation
- ✅ **Audit Trail**: Complete logging of secret access

### **Operational Improvements:**
- ✅ **Automated Provisioning**: Secrets injected at runtime
- ✅ **Policy-Based Access**: Fine-grained access control
- ✅ **Centralized Management**: Single source of truth for secrets
- ✅ **Compliance**: Meets enterprise security requirements

### **DevOps Benefits:**
- ✅ **GitOps Friendly**: Configuration without sensitive data
- ✅ **CI/CD Integration**: Automated secret management in pipelines
- ✅ **Reduced Risk**: No manual secret handling
- ✅ **Scalability**: Works across multiple environments

This Vault-integrated approach provides enterprise-grade security while maintaining operational simplicity and automation capabilities.




Yes, using the **OpenShift Vault Operator** significantly changes the architecture and simplifies the integration. Here's how it impacts your setup:

## Key Differences with OpenShift Vault Operator

### 1. **Simplified Authentication**
```yaml
# No need for manual Vault policy/role setup
# Vault Operator handles Kubernetes auth automatically

apiVersion: redhatcop.redhat.io/v1alpha1
kind: VaultSecret
metadata:
  name: gravitee-jwt-certificates
spec:
  vaultSecretDefinition:
    path: "pki/issue/jwt-signing"
    payload:
      common_name: "jwt.gravitee.local"
      ttl: "8760h"
  destination:
    name: gravitee-jwt-cert
    type: Opaque
---
apiVersion: redhatcop.redhat.io/v1alpha1
kind: VaultSecret
metadata:
  name: gravitee-tls-certificates
spec:
  vaultSecretDefinition:
    path: "pki/issue/https-server"
    payload:
      common_name: "gravitee-am.local"
      ttl: "8760h"
  destination:
    name: gravitee-tls-cert
    type: kubernetes.io/tls
---
apiVersion: redhatcop.redhat.io/v1alpha1
kind: VaultSecret
metadata:
  name: gravitee-passwords
spec:
  vaultSecretDefinition:
    path: "kv/data/gravitee/all"
  destination:
    name: gravitee-passwords
    type: Opaque
```

### 2. **Enhanced Deployment with Operator Integration**
```yaml
# gravitee-am-vault-operator-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: gravitee-am-management-api
  labels:
    app: gravitee-am
    component: management-api
spec:
  replicas: 1
  selector:
    matchLabels:
      app: gravitee-am
      component: management-api
  template:
    metadata:
      labels:
        app: gravitee-am
        component: management-api
      # Vault Operator annotations (simpler than HashiCorp's)
      annotations:
        vault.security.banzaicloud.io/vault-addr: "https://vault:8200"
        vault.security.banzaicloud.io/vault-role: "gravitee-role"
        vault.security.banzaicloud.io/vault-skip-verify: "false"
        vault.security.banzaicloud.io/vault-tls-secret: "vault-tls"
    spec:
      serviceAccountName: gravitee-vault-sa  # Vault Operator manages this
      securityContext:
        runAsNonRoot: true
        runAsUser: 1001
        fsGroup: 1001
      
      containers:
      - name: gravitee-am-management-api
        image: graviteeio/am-management-api:3.20
        ports:
        - containerPort: 8092
          name: http
        - containerPort: 8443
          name: https
        
        # Vault Operator injects secrets as environment variables
        env:
        - name: TLS_CERT
          valueFrom:
            secretKeyRef:
              name: gravitee-tls-cert
              key: tls.crt
        - name: TLS_KEY
          valueFrom:
            secretKeyRef:
              name: gravitee-tls-cert
              key: tls.key
        - name: JWT_KEYSTORE
          valueFrom:
            secretKeyRef:
              name: gravitee-jwt-cert
              key: keystore
        
        # Passwords from Vault Operator managed secrets
        envFrom:
        - secretRef:
            name: gravitee-passwords
        
        volumeMounts:
        - name: certificates
          mountPath: /opt/gravitee/certificates
          readOnly: true
        - name: config-volume
          mountPath: /opt/gravitee/config
        - name: logs-volume
          mountPath: /opt/gravitee/logs
        
        resources:
          requests:
            memory: "512Mi"
            cpu: "250m"
          limits:
            memory: "1Gi"
            cpu: "500m"
        
        livenessProbe:
          httpGet:
            path: /management/health
            port: 8092
          initialDelaySeconds: 120
          periodSeconds: 30
        
        readinessProbe:
          httpGet:
            path: /management/health
            port: 8092
          initialDelaySeconds: 30
          periodSeconds: 10
      
      volumes:
      - name: certificates
        projected:
          sources:
          - secret:
              name: gravitee-tls-cert
          - secret:
              name: gravitee-jwt-cert
      - name: config-volume
        configMap:
          name: gravitee-config
      - name: logs-volume
        emptyDir: {}
```

### 3. **Vault Operator Configuration**
```yaml
# vault-connection.yaml
apiVersion: redhatcop.redhat.io/v1alpha1
kind: VaultConnection
metadata:
  name: vault-connection
spec:
  caBundle: ""
  skipTLSVerify: false
  address: "https://vault.vault.svc:8200"
  tlsServerName: "vault.vault.svc"
---
apiVersion: redhatcop.redhat.io/v1alpha1
kind: VaultAuth
metadata:
  name: gravitee-auth
spec:
  vaultConnection: vault-connection
  kubernetes:
    role: gravitee-role
    serviceAccount: gravitee-vault-sa
    audiences:
    - vault
---
apiVersion: redhatcop.redhat.io/v1alpha1
kind: VaultPolicy
metadata:
  name: gravitee-policy
spec:
  vaultAuth: gravitee-auth
  policy: |
    path "pki/issue/jwt-signing" {
      capabilities = ["create", "update"]
    }
    path "pki/issue/https-server" {
      capabilities = ["create", "update"]
    }
    path "kv/data/gravitee/*" {
      capabilities = ["read"]
    }
```

### 4. **Certificate Auto-Rotation with Vault Operator**
```yaml
# certificate-rotation-with-operator.yaml
apiVersion: redhatcop.redhat.io/v1alpha1
kind: VaultSecret
metadata:
  name: gravitee-jwt-cert-auto
  annotations:
    # Auto-refresh every 24 hours
    vault.redhatcop.io/refresh-period: "24h"
spec:
  vaultAuth: gravitee-auth
  vaultSecretDefinition:
    path: "pki/issue/jwt-signing"
    payload:
      common_name: "jwt.gravitee.local"
      ttl: "8760h"  # 1 year
  destination:
    name: gravitee-jwt-cert
    type: Opaque
    labels:
      app: gravitee-am
      refresh: "auto"
---
apiVersion: redhatcop.redhat.io/v1alpha1
kind: VaultSecret
metadata:
  name: gravitee-tls-cert-auto
  annotations:
    # Auto-refresh every 24 hours
    vault.redhatcop.io/refresh-period: "24h"
spec:
  vaultAuth: gravitee-auth
  vaultSecretDefinition:
    path: "pki/issue/https-server"
    payload:
      common_name: "gravitee-am.local"
      ttl: "8760h"
  destination:
    name: gravitee-tls-cert
    type: kubernetes.io/tls
```

### 5. **Enhanced Monitoring with Operator**
```yaml
# monitoring-with-operator.yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: gravitee-vault-monitor
  labels:
    app: gravitee-am
spec:
  selector:
    matchLabels:
      app: gravitee-am
  endpoints:
  - port: http
    path: /management/health
    interval: 30s
---
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: gravitee-vault-rules
spec:
  groups:
  - name: gravitee.vault.operator
    rules:
    - alert: VaultSecretRefreshFailed
      expr: vault_secret_refresh_failures_total > 0
      for: 5m
      labels:
        severity: critical
      annotations:
        summary: "Vault secret refresh failed for Gravitee"
        description: "Vault Operator failed to refresh secrets for Gravitee deployment"
    
    - alert: VaultConnectionDown
      expr: vault_connection_status == 0
      for: 2m
      labels:
        severity: critical
      annotations:
        summary: "Vault connection is down"
        description: "Gravitee deployment cannot connect to Vault"
```

### 6. **CI/CD Integration with Vault Operator**
```yaml
# .gitlab-ci.yml - Vault Operator version
stages:
  - validate
  - deploy
  - verify
  - monitor

variables:
  OPENSHIFT_PROJECT: $CI_PROJECT_NAMESPACE

validate_vault:
  stage: validate
  script:
    - oc project $OPENSHIFT_PROJECT
    - |
      echo "Validating Vault Operator resources..."
      oc get vaultconnection
      oc get vaultauth
      oc get vaultpolicy
      oc get vaultsecret
  only:
    - merge_requests
    - master

deploy_with_operator:
  stage: deploy
  script:
    - oc project $OPENSHIFT_PROJECT
    - |
      echo "Deploying Vault Operator resources..."
      oc apply -f vault-connection.yaml
      oc apply -f vault-auth.yaml
      oc apply -f vault-policy.yaml
      oc apply -f vault-secrets.yaml  # This creates the actual secrets
    - |
      echo "Deploying Gravitee AM..."
      oc apply -f gravitee-deployment.yaml
    - oc rollout status deployment/gravitee-am-management-api
  only:
    - master

verify_secrets:
  stage: verify
  script:
    - oc project $OPENSHIFT_PROJECT
    - |
      echo "Verifying secrets are created by Vault Operator..."
      oc get secret gravitee-jwt-cert
      oc get secret gravitee-tls-cert
      oc get secret gravitee-passwords
    - |
      echo "Checking Vault Secret status..."
      oc get vaultsecret gravitee-jwt-cert-auto -o jsonpath='{.status}'
  only:
    - master

monitor_rotation:
  stage: monitor
  script:
    - oc project $OPENSHIFT_PROJECT
    - |
      echo "Monitoring certificate rotation..."
      # Check if VaultSecret is refreshing properly
      LAST_REFRESH=$(oc get vaultsecret gravitee-jwt-cert-auto -o jsonpath='{.status.lastVaultSecretUpdate}')
      echo "Last refresh: $LAST_REFRESH"
  only:
    - schedules
```

### 7. **Vault Operator Service Account**
```yaml
# vault-operator-service-account.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: gravitee-vault-sa
  labels:
    app: gravitee-am
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: gravitee-vault-role
  labels:
    app: gravitee-am
rules:
- apiGroups: [""]
  resources: ["pods", "services", "endpoints"]
  verbs: ["get", "list", "watch"]
- apiGroups: ["redhatcop.redhat.io"]
  resources: ["vaultsecrets", "vaultsecrets/status"]
  verbs: ["get", "list", "watch", "update"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: gravitee-vault-rolebinding
  labels:
    app: gravitee-am
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: gravitee-vault-role
subjects:
- kind: ServiceAccount
  name: gravitee-vault-sa
```

## Key Benefits of OpenShift Vault Operator:

### **Simplified Management:**
- ✅ **CRD-Based**: Manage Vault resources as Kubernetes custom resources
- ✅ **Automatic Auth**: No manual Vault policy/role setup needed
- ✅ **Self-Service**: Developers can manage their own Vault integrations

### **Enhanced Automation:**
- ✅ **Auto-Rotation**: Built-in secret refresh capabilities
- ✅ **Reconciliation**: Automatic recovery from failures
- ✅ **Status Monitoring**: Built-in status reporting

### **Better Integration:**
- ✅ **Native OpenShift**: Tight integration with OpenShift features
- ✅ **Operator Pattern**: Follows Kubernetes operator best practices
- ✅ **Monitoring**: Built-in Prometheus metrics and alerts

### **Improved Security:**
- ✅ **Least Privilege**: Automatic minimal permission assignment
- ✅ **Audit Trail**: Comprehensive logging and monitoring
- ✅ **Compliance**: Meets enterprise security requirements

The OpenShift Vault Operator provides a much more streamlined and automated approach compared to manual Vault integration, making it easier to manage secrets and certificates in your Gravitee deployment.
