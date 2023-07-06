#  Workshop: Annotating Service with Service Serving Certificate


This workshop demonstrates how to annotate a service in an OpenShift cluster with a Service Serving Certificate to enable secure communication over HTTPS between pods in Openshift

## Prerequisites
   - OpenShift CLI (oc) installed and configured
   - Access to an OpenShift cluster

## During this workshop, you will perform the following steps:

   Step 1: Create new project.
   
   Step 2: Deploy http/https simple server that support both http and https connections.
   
   Step 3: Deploy a Curl client pod to test the secure communication with the HTTP server.
   
   Step 4: Annotate the https server service to sign it using the Service Serving Certificate.
   
   Step 5: Adjust the server and client deployments to consume the certificates to allow https communication.


### Step 1: Create a New Project
1. Open a terminal or command prompt.
2. Run the following command to create a new project:
```bash
   oc new-project https-server-project
```

### Step 2: Deploy HTTP/HTTPS Simple Server
1. Create a file named http-server.yaml and copy the following content into it:

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
      ports:
      - containerPort: 8080
        name: http
      - containerPort: 8443
        name: https
---
apiVersion: v1
kind: Service
metadata:
  name: my-http-listener-service
spec:
  selector:
    app: my-http-listener
  ports:
  - name: http
    protocol: TCP
    port: 8080
    targetPort: 8080
  - name: https
    protocol: TCP
    port: 8443
    targetPort: 8443   
```

2. Run the following command to deploy the HTTP/HTTPS simple server:

```bash
oc apply -f http-server.yaml
```
### Step 3: Deploy the Curl Client
1. Create a file named curl-client.yaml and copy the following content into it:
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
```
2. Run the following command to deploy the curl client:

```bash
oc apply -f curl-client.yaml
```
3. Run curl command from client to server
   - Did the curl worked using the https endpoint ?
  

### Step 4: Annotate the HTTPS Server Service

1. Use the following article to annotate the service so the service-signer-CA will create a new signed pair of crt and key 
   
**https://docs.openshift.com/container-platform/4.12/security/certificates/service-serving-certificate.html#add-service-certificate_service-serving-certificate**

2. Confirm the creation of one secret and one configmap.
   - what the configmap is used for ?

### Step 5: Adjust the server and client deployments to consume the certificates to allow https communication

1. Modify the http server to consume the secret.

The http server expect to get the crt and key as follow:
   - tls.crt: should be mounted at /app/fullchain.pem
   - tls.key: should be mounted at /app/privkey.pem

2. Modify the curl client to use the ca certficate to turst the https server
Use curl command and manully point the certificate and test https communication 
