Here's a complete OpenShift deployment YAML example with both TLS certificates for HTTPS and JWT certificates for signing JWT tokens:

```yaml
# gravitee-am-full-deployment.yaml
apiVersion: v1
kind: Secret
metadata:
  name: gravitee-tls-cert
  labels:
    app: gravitee-am
type: kubernetes.io/tls
data:
  # These should be base64 encoded
  # tls.crt: <base64-encoded-cert>
  # tls.key: <base64-encoded-key>
---
apiVersion: v1
kind: Secret
metadata:
  name: gravitee-jwt-cert
  labels:
    app: gravitee-am
type: Opaque
data:
  # jwt-keystore.p12: <base64-encoded-keystore>
  # jwt-public.pem: <base64-encoded-public-cert>
---
apiVersion: v1
kind: Secret
metadata:
  name: gravitee-passwords
  labels:
    app: gravitee-am
type: Opaque
data:
  # Store base64 encoded passwords
  # tls-key-password: <base64-encoded-password>
  # jwt-keystore-password: <base64-encoded-password>
  # jwt-key-password: <base64-encoded-password>
  # mongodb-password: <base64-encoded-password>
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: gravitee-config
  labels:
    app: gravitee-am
data:
  gravitee.yml: |
    ###########################################################################
    # Gravitee Configuration
    ###########################################################################
    
    # JWT Configuration for token signing
    jwt:
      signing:
        keystore:
          type: "PKCS12"
          path: "/opt/gravitee/certificates/jwt/jwt-keystore.p12"
          password: ${JWT_KEYSTORE_PASSWORD}
          alias: "jwt-signing-key"
          keyPassword: ${JWT_KEY_PASSWORD}
      
      encryption:
        keystore:
          type: "PKCS12"
          path: "/opt/gravitee/certificates/jwt/jwt-keystore.p12"
          password: ${JWT_KEYSTORE_PASSWORD}
          alias: "jwt-encryption-key"
          keyPassword: ${JWT_KEY_PASSWORD}
      
      default:
        expDelay: 28800  # 8 hours
        iss: "gravitee-am"
        aud: "gravitee-applications"
    
    # HTTP Server Configuration
    http:
      port: 8092
      host: 0.0.0.0
      idleTimeout: 0
      tcpKeepAlive: true
      compressionSupported: false
      maxHeaderSize: 8192
      maxChunkSize: 8192
      decompressionSupported: false
      perMessageWebSocketCompressionSupported: false
      perFrameWebSocketCompressionSupported: false
      maxWebSocketFrameSize: 65536
      maxWebSocketMessageSize: 65536
      maxFormAttributeSize: 2048
      maxFormFields: 1024
      maxBufferedBytes: 65536
      acceptBacklog: 128
      acceptorThreads: 1
      clientAuth: none
      pool:
        type: worker
        maxThreads: 200
        minThreads: 10
        threadIdleTimeout: 60000
        maxQueuedRequests: -1
    
    # HTTPS Configuration
    https:
      port: 8443
      host: 0.0.0.0
      idleTimeout: 0
      tcpKeepAlive: true
      compressionSupported: false
      maxHeaderSize: 8192
      maxChunkSize: 8192
      decompressionSupported: false
      perMessageWebSocketCompressionSupported: false
      perFrameWebSocketCompressionSupported: false
      maxWebSocketFrameSize: 65536
      maxWebSocketMessageSize: 65536
      ssl:
        clientAuth: none
        keystore:
          type: "PKCS12"
          path: "/opt/gravitee/certificates/tls/tls-keystore.p12"
          password: ${TLS_KEYSTORE_PASSWORD}
          alias: "https-key"
          keyPassword: ${TLS_KEY_PASSWORD}
        truststore:
          type: "JKS"
          path: "/opt/gravitee/certificates/truststore.jks"
          password: ${TRUSTSTORE_PASSWORD}
    
    # Database Configuration
    management:
      type: mongodb
      mongodb:
        uri: mongodb://${MONGODB_USERNAME}:${MONGODB_PASSWORD}@mongodb:27017/gravitee-am
    
    # Security Configuration
    security:
      providers:
        - type: memory
          enabled: true
          configuration:
            users:
              - user:
                  username: admin
                  password: ${ADMIN_PASSWORD}
                  roles:
                    - ADMIN
    
    # Gateway Configuration
    gateway:
      enabled: true
      url: https://gravitee-gateway:8443
    
    # User Management
    user:
      enabled: true
    
    # OAuth2 Configuration
    oauth2:
      enabled: true
    
    # OpenID Connect Configuration
    oidc:
      enabled: true
    
    # Email Configuration
    email:
      enabled: false
    
    # Logging Configuration
    logger:
      level: INFO
      appenders:
        - type: console
          name: console
          layout:
            type: pattern
            pattern: "%d{HH:mm:ss.SSS} [%t] %-5level %logger{36} - %msg%n"
  
  logback.xml: |
    <?xml version="1.0" encoding="UTF-8"?>
    <configuration>
        <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
            <encoder>
                <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
            </encoder>
        </appender>
        
        <root level="INFO">
            <appender-ref ref="STDOUT" />
        </root>
    </configuration>
---
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
    spec:
      securityContext:
        runAsNonRoot: true
        runAsUser: 1001
        fsGroup: 1001
        runAsGroup: 1001
      
      initContainers:
      - name: cert-permissions
        image: busybox:1.35
        command:
        - sh
        - -c
        - |
          echo "Setting certificate permissions..."
          chmod 400 /opt/gravitee/certificates/jwt/*
          chmod 400 /opt/gravitee/certificates/tls/*
          ls -la /opt/gravitee/certificates/jwt/
          ls -la /opt/gravitee/certificates/tls/
        volumeMounts:
        - name: jwt-certificates
          mountPath: /opt/gravitee/certificates/jwt
        - name: tls-certificates
          mountPath: /opt/gravitee/certificates/tls
      
      containers:
      - name: gravitee-am-management-api
        image: graviteeio/am-management-api:3.20
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 8092
          name: http
          protocol: TCP
        - containerPort: 8443
          name: https
          protocol: TCP
        
        env:
        # TLS Configuration
        - name: TLS_KEYSTORE_PASSWORD
          valueFrom:
            secretKeyRef:
              name: gravitee-passwords
              key: tls-keystore-password
        - name: TLS_KEY_PASSWORD
          valueFrom:
            secretKeyRef:
              name: gravitee-passwords
              key: tls-key-password
        - name: TRUSTSTORE_PASSWORD
          valueFrom:
            secretKeyRef:
              name: gravitee-passwords
              key: truststore-password
        
        # JWT Configuration
        - name: JWT_KEYSTORE_PASSWORD
          valueFrom:
            secretKeyRef:
              name: gravitee-passwords
              key: jwt-keystore-password
        - name: JWT_KEY_PASSWORD
          valueFrom:
            secretKeyRef:
              name: gravitee-passwords
              key: jwt-key-password
        
        # Database Configuration
        - name: MONGODB_USERNAME
          valueFrom:
            secretKeyRef:
              name: gravitee-passwords
              key: mongodb-username
        - name: MONGODB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: gravitee-passwords
              key: mongodb-password
        
        # Admin User
        - name: ADMIN_PASSWORD
          valueFrom:
            secretKeyRef:
              name: gravitee-passwords
              key: admin-password
        
        # JVM Configuration
        - name: JAVA_OPTS
          value: "-Xms512m -Xmx1024m -Dfile.encoding=UTF-8"
        
        # Gravitee Home
        - name: GRAVITEE_HOME
          value: "/opt/gravitee"
        
        volumeMounts:
        - name: jwt-certificates
          mountPath: /opt/gravitee/certificates/jwt
          readOnly: true
        - name: tls-certificates
          mountPath: /opt/gravitee/certificates/tls
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
            scheme: HTTP
          initialDelaySeconds: 120
          periodSeconds: 30
          timeoutSeconds: 10
          failureThreshold: 3
        
        readinessProbe:
          httpGet:
            path: /management/health
            port: 8092
            scheme: HTTP
          initialDelaySeconds: 30
          periodSeconds: 10
          timeoutSeconds: 5
          failureThreshold: 3
      
      volumes:
      - name: jwt-certificates
        secret:
          secretName: gravitee-jwt-cert
          defaultMode: 256  # 0400 - read-only for owner
          items:
          - key: jwt-keystore.p12
            path: jwt-keystore.p12
          - key: jwt-public.pem
            path: jwt-public.pem
      
      - name: tls-certificates
        secret:
          secretName: gravitee-tls-cert
          defaultMode: 256  # 0400 - read-only for owner
          items:
          - key: tls.crt
            path: tls.crt
          - key: tls.key
            path: tls.key
      
      - name: config-volume
        configMap:
          name: gravitee-config
          items:
          - key: gravitee.yml
            path: gravitee.yml
          - key: logback.xml
            path: logback.xml
      
      - name: logs-volume
        emptyDir: {}
---
apiVersion: v1
kind: Service
metadata:
  name: gravitee-am-management-api
  labels:
    app: gravitee-am
    component: management-api
spec:
  ports:
  - name: http
    port: 8092
    targetPort: 8092
    protocol: TCP
  - name: https
    port: 8443
    targetPort: 8443
    protocol: TCP
  selector:
    app: gravitee-am
    component: management-api
  type: ClusterIP
---
apiVersion: route.openshift.io/v1
kind: Route
metadata:
  name: gravitee-am-management-api
  labels:
    app: gravitee-am
    component: management-api
spec:
  host: gravitee-am-management-api.apps.cluster.example.com
  to:
    kind: Service
    name: gravitee-am-management-api
    weight: 100
  port:
    targetPort: https
  tls:
    termination: passthrough  # Let Gravitee handle TLS
    insecureEdgeTerminationPolicy: Redirect
  wildcardPolicy: None
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: gravitee-am-gateway
  labels:
    app: gravitee-am
    component: gateway
spec:
  replicas: 1
  selector:
    matchLabels:
      app: gravitee-am
      component: gateway
  template:
    metadata:
      labels:
        app: gravitee-am
        component: gateway
    spec:
      securityContext:
        runAsNonRoot: true
        runAsUser: 1001
        fsGroup: 1001
        runAsGroup: 1001
      
      containers:
      - name: gravitee-am-gateway
        image: graviteeio/am-gateway:3.20
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 8082
          name: http
          protocol: TCP
        - containerPort: 8443
          name: https
          protocol: TCP
        
        env:
        # TLS Configuration
        - name: TLS_KEYSTORE_PASSWORD
          valueFrom:
            secretKeyRef:
              name: gravitee-passwords
              key: tls-keystore-password
        - name: TLS_KEY_PASSWORD
          valueFrom:
            secretKeyRef:
              name: gravitee-passwords
              key: tls-key-password
        
        # JWT Configuration
        - name: JWT_KEYSTORE_PASSWORD
          valueFrom:
            secretKeyRef:
              name: gravitee-passwords
              key: jwt-keystore-password
        - name: JWT_KEY_PASSWORD
          valueFrom:
            secretKeyRef:
              name: gravitee-passwords
              key: jwt-key-password
        
        # Management API Configuration
        - name: MANAGEMENT_API_URL
          value: "https://gravitee-am-management-api:8443/management"
        
        volumeMounts:
        - name: jwt-certificates
          mountPath: /opt/gravitee/certificates/jwt
          readOnly: true
        - name: tls-certificates
          mountPath: /opt/gravitee/certificates/tls
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
            path: /gateway/health
            port: 8082
            scheme: HTTP
          initialDelaySeconds: 120
          periodSeconds: 30
          timeoutSeconds: 10
          failureThreshold: 3
        
        readinessProbe:
          httpGet:
            path: /gateway/health
            port: 8082
            scheme: HTTP
          initialDelaySeconds: 30
          periodSeconds: 10
          timeoutSeconds: 5
          failureThreshold: 3
      
      volumes:
      - name: jwt-certificates
        secret:
          secretName: gravitee-jwt-cert
          defaultMode: 256  # 0400
          items:
          - key: jwt-keystore.p12
            path: jwt-keystore.p12
          - key: jwt-public.pem
            path: jwt-public.pem
      
      - name: tls-certificates
        secret:
          secretName: gravitee-tls-cert
          defaultMode: 256  # 0400
          items:
          - key: tls.crt
            path: tls.crt
          - key: tls.key
            path: tls.key
      
      - name: config-volume
        configMap:
          name: gravitee-gateway-config  # Separate config for gateway
      
      - name: logs-volume
        emptyDir: {}
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: gravitee-gateway-config
  labels:
    app: gravitee-am
data:
  gravitee.yml: |
    ###########################################################################
    # Gravitee Gateway Configuration
    ###########################################################################
    
    # HTTP Server Configuration
    http:
      port: 8082
      host: 0.0.0.0
    
    # HTTPS Configuration
    https:
      port: 8443
      host: 0.0.0.0
      ssl:
        clientAuth: none
        keystore:
          type: "PKCS12"
          path: "/opt/gravitee/certificates/tls/tls-keystore.p12"
          password: ${TLS_KEYSTORE_PASSWORD}
          alias: "https-key"
          keyPassword: ${TLS_KEY_PASSWORD}
    
    # JWT Configuration
    jwt:
      signing:
        keystore:
          type: "PKCS12"
          path: "/opt/gravitee/certificates/jwt/jwt-keystore.p12"
          password: ${JWT_KEYSTORE_PASSWORD}
          alias: "jwt-signing-key"
          keyPassword: ${JWT_KEY_PASSWORD}
    
    # Management API Configuration
    management:
      type: http
      http:
        url: ${MANAGEMENT_API_URL:https://localhost:8443/management}
        ssl:
          trustAll: true  # Set to false in production with proper truststore
    
    # Logging
    logger:
      level: INFO
      appenders:
        - type: console
          name: console
          layout:
            type: pattern
            pattern: "%d{HH:mm:ss.SSS} [%t] %-5level %logger{36} - %msg%n"
---
apiVersion: v1
kind: Service
metadata:
  name: gravitee-am-gateway
  labels:
    app: gravitee-am
    component: gateway
spec:
  ports:
  - name: http
    port: 8082
    targetPort: 8082
    protocol: TCP
  - name: https
    port: 8443
    targetPort: 8443
    protocol: TCP
  selector:
    app: gravitee-am
    component: gateway
  type: ClusterIP
---
apiVersion: route.openshift.io/v1
kind: Route
metadata:
  name: gravitee-am-gateway
  labels:
    app: gravitee-am
    component: gateway
spec:
  host: gravitee-am-gateway.apps.cluster.example.com
  to:
    kind: Service
    name: gravitee-am-gateway
    weight: 100
  port:
    targetPort: https
  tls:
    termination: passthrough
    insecureEdgeTerminationPolicy: Redirect
  wildcardPolicy: None
```

## How to Deploy:

```bash
# 1. Create the secrets first (you'll need to base64 encode your files)
# Create JWT secret
oc create secret generic gravitee-jwt-cert \
  --from-file=jwt-keystore.p12=./your-jwt-keystore.p12 \
  --from-file=jwt-public.pem=./your-jwt-public.pem

# Create TLS secret
oc create secret generic gravitee-tls-cert \
  --from-file=tls.crt=./your-tls-cert.crt \
  --from-file=tls.key=./your-tls-key.key

# Create passwords secret
oc create secret generic gravitee-passwords \
  --from-literal=tls-keystore-password=your-tls-keystore-password \
  --from-literal=tls-key-password=your-tls-key-password \
  --from-literal=jwt-keystore-password=your-jwt-keystore-password \
  --from-literal=jwt-key-password=your-jwt-key-password \
  --from-literal=mongodb-username=your-mongodb-user \
  --from-literal=mongodb-password=your-mongodb-password \
  --from-literal=admin-password=your-admin-password

# 2. Apply the deployment
oc apply -f gravitee-am-full-deployment.yaml

# 3. Check the deployment
oc get pods -l app=gravitee-am
oc get services
oc get routes
```

## Key Features of This Deployment:

1. **Separate Certificates**: TLS certs for HTTPS and JWT certs for token signing
2. **Secure Mounting**: Read-only volumes with proper permissions
3. **Environment Variables**: Passwords from secrets, not hardcoded
4. **Init Container**: Sets proper file permissions
5. **Health Checks**: Liveness and readiness probes
6. **Resource Limits**: CPU and memory constraints
7. **Security Context**: Non-root user execution
8. **Route Configuration**: Proper OpenShift routes with TLS termination
9. **Complete Stack**: Management API + Gateway components

This deployment provides a production-ready setup with proper certificate management for both HTTPS communication and JWT token signing.
