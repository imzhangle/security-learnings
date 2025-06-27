Here are **detailed steps** for the **application team** to install **Gravitee API Management (APIM)** into an **OpenShift cluster**, with **separate components**: API Gateway, Access Management, and Admin UI — including **YAML structure, routing, and service separation**.

---

## ✅ Overview

Gravitee’s architecture includes:

1. **API Gateway** – Handles API traffic
2. **Access Management (AM)** – Identity, OAuth2, tokens
3. **APIM UI (Admin / Developer Portal)** – Web UI

Each needs:

* Separate Deployment or StatefulSet
* Own Service
* Individual OpenShift `Route`

---

## 🔧 STEP 1: Pre-reqs & Namespaces

1. Create a dedicated namespace/project:

```bash
oc new-project gravitee
```

2. Ensure the namespace can pull required container images (private registry if needed).

---

## 🧱 STEP 2: Deploy Gravitee Components Separately

### 2.1. API Gateway

```yaml
# gravitee-gateway-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: gravitee-gateway
  labels:
    app: gravitee-gateway
spec:
  replicas: 2
  selector:
    matchLabels:
      app: gravitee-gateway
  template:
    metadata:
      labels:
        app: gravitee-gateway
    spec:
      containers:
        - name: gateway
          image: graviteeio/apim-gateway:4.3.0
          ports:
            - containerPort: 8082
          env:
            - name: gravitee_management_api_url
              value: "http://gravitee-management-api:8083/management"

---
# Service
apiVersion: v1
kind: Service
metadata:
  name: gravitee-gateway
spec:
  selector:
    app: gravitee-gateway
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8082

---
# Route
apiVersion: route.openshift.io/v1
kind: Route
metadata:
  name: gravitee-gateway
spec:
  host: gateway.apps.example.com
  to:
    kind: Service
    name: gravitee-gateway
  tls:
    termination: edge
```

---

### 2.2. Management API

```yaml
# gravitee-management-api-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: gravitee-management-api
  labels:
    app: gravitee-management-api
spec:
  replicas: 1
  selector:
    matchLabels:
      app: gravitee-management-api
  template:
    metadata:
      labels:
        app: gravitee-management-api
    spec:
      containers:
        - name: management-api
          image: graviteeio/apim-management-api:4.3.0
          ports:
            - containerPort: 8083

---
# Service
apiVersion: v1
kind: Service
metadata:
  name: gravitee-management-api
spec:
  selector:
    app: gravitee-management-api
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8083

---
# Route
apiVersion: route.openshift.io/v1
kind: Route
metadata:
  name: gravitee-management-api
spec:
  host: management.apps.example.com
  to:
    kind: Service
    name: gravitee-management-api
  tls:
    termination: edge
```

---

### 2.3. APIM UI (Admin Portal)

```yaml
# gravitee-apim-ui-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: gravitee-apim-ui
  labels:
    app: gravitee-apim-ui
spec:
  replicas: 1
  selector:
    matchLabels:
      app: gravitee-apim-ui
  template:
    metadata:
      labels:
        app: gravitee-apim-ui
    spec:
      containers:
        - name: apim-ui
          image: graviteeio/apim-console-webui:4.3.0
          ports:
            - containerPort: 8080

---
# Service
apiVersion: v1
kind: Service
metadata:
  name: gravitee-apim-ui
spec:
  selector:
    app: gravitee-apim-ui
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080

---
# Route
apiVersion: route.openshift.io/v1
kind: Route
metadata:
  name: gravitee-apim-ui
spec:
  host: admin.apps.example.com
  to:
    kind: Service
    name: gravitee-apim-ui
  tls:
    termination: edge
```

---

## 🔐 STEP 3: Access Management (Optional Module)

If you're using **Gravitee Access Management (AM)**:

```yaml
# gravitee-am-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: gravitee-am
  labels:
    app: gravitee-am
spec:
  replicas: 1
  selector:
    matchLabels:
      app: gravitee-am
  template:
    metadata:
      labels:
        app: gravitee-am
    spec:
      containers:
        - name: access-management
          image: graviteeio/am-management-api:3.20.0
          ports:
            - containerPort: 8093

---
# Service
apiVersion: v1
kind: Service
metadata:
  name: gravitee-am
spec:
  selector:
    app: gravitee-am
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8093

---
# Route
apiVersion: route.openshift.io/v1
kind: Route
metadata:
  name: gravitee-am
spec:
  host: am.apps.example.com
  to:
    kind: Service
    name: gravitee-am
  tls:
    termination: edge
```

---

## 📂 STEP 4: Organize YAMLs by Component

You can organize the structure like:

```
gravitee/
├── gateway/
│   ├── deployment.yaml
│   ├── service.yaml
│   └── route.yaml
├── management-api/
│   ├── deployment.yaml
│   ├── service.yaml
│   └── route.yaml
├── apim-ui/
│   ├── deployment.yaml
│   ├── service.yaml
│   └── route.yaml
└── access-management/
    ├── deployment.yaml
    ├── service.yaml
    └── route.yaml
```

This allows the **infra team** to review routes and services separately and apply them selectively.

---

## 🧪 STEP 5: Apply Manifests

The application team can apply these using:

```bash
oc apply -f gravitee/gateway/
oc apply -f gravitee/management-api/
oc apply -f gravitee/apim-ui/
oc apply -f gravitee/access-management/   # if using AM
```

---

## ✅ Summary

* Application team **splits Gravitee components** into separate manifests.
* Each component has its own Deployment, Service, and Route.
* Platform team **reviews ingress (Route)** and provides external DNS & TLS.
* Application team handles configuration, persistence, and connectivity.

Would you like me to create a Helm-based version or a GitOps-compatible folder structure (e.g., Argo CD friendly)?
