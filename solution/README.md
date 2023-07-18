### Annotate the service as follow:
```bash
oc annotate service my-http-listener-service service.beta.openshift.io/serving-cert-secret-name=https-server-certs
```

### Check for the newly created secret:
```bash
oc get svc |grep https-server-certs
```

### The secret contain two files, tls.crt & tls.key. Mount them both to the https server:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-http-listener
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-http-listener
  template:
    metadata:
      labels:
        app: my-http-listener
    spec:
      containers:
      - name: my-http-listener
        image: mendhak/http-https-echo:26
        env:
        - name: HTTP_PORT
          value: "8080"
        - name: HTTPS_PORT
          value: "8443"
        volumeMounts:
        - name: cert-volume
          mountPath: /app/fullchain.pem
          subPath: tls.crt
        - name: cert-volume
          mountPath: /app/privkey.pem
          subPath: tls.key
        ports:
        - containerPort: 8080
          name: http
        - containerPort: 8443
          name: https
        securityContext:
          allowPrivilegeEscalation: false
          capabilities:
            drop:
              - ALL
      volumes:
      - name: cert-volume
        secret:
          secretName: https-server-certs
      securityContext:
        runAsNonRoot: true
        seccompProfile:
          type: RuntimeDefault
```

### Mount the service ca-cert configmap to the curl client pod:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: curl-pod
spec:
  replicas: 1
  selector:
    matchLabels:
      app: curl-pod
  template:
    metadata:
      labels:
        app: curl-pod
    spec:
      containers:
      - name: curl-container
        image: curlimages/curl
        command: ["sleep", "infinity"]
        volumeMounts:
        - name: ca-volume
          mountPath: /certs
        securityContext:
          allowPrivilegeEscalation: false
          capabilities:
            drop: ["ALL"]
          runAsNonRoot: true
          seccompProfile:
            type: "RuntimeDefault"
      volumes:
      - name: ca-volume
        configMap:
          name: openshift-service-ca.crt
          items:
          - key: service-ca.crt
            path: service-ca.crt
```

### Issue curl and point the cacert file:
```bash
curl --cacert /certs/service-ca.crt  https://my-http-listener-service.https-server-project.svc:8443 
```
