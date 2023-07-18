# Service Serving Certificate Demo


The following demo shows how to annotate a service in an OpenShift cluster with a Service Serving Certificate to enable safe communication between Openshift pods over HTTPS.

## Prerequisites
   - OpenShift CLI (oc) installed and configured
   - Access to an OpenShift cluster

## During this demo you will do the following:

   Step 1: Create new project.
   
   Step 2: Deploy http/https simple server that support both http and https connections.
   
   Step 3: Deploy a Curl client pod to test the secure communication with the HTTP server.
   
   Step 4: Annotate the https server service to sign it using the Service Serving Certificate.
   
   Step 5: Adjust the server and client deployments to consume the certificates to allow https communication.


### Step 1: Create a New Project
1. Open a terminal or command prompt.
2. Create a new project:
```bash
   oc new-project https-server-project
```


### Step 2: Deploy the HTTP/HTTPS Simple Server
1. Deploy the HTTP/HTTPS simple server:

```bash
oc apply -f http-server.yaml
```


### Step 3: Deploy the Curl Client
1. Deploy the curl client:

```bash
oc apply -f curl-client.yaml
```
2. Issue curl command from client to server
   - Did the curl execute correctly while using the https endpoint?
  

### Step 4: Annotate the HTTPS Server Service

1. Use the following article to annotate the service so that the service-signer-CA generates a new secret with a signed pair of crt and key. 
   
**https://docs.openshift.com/container-platform/4.12/security/certificates/service-serving-certificate.html#add-service-certificate_service-serving-certificate**

2. Confirm the creation of the secret.
   - what else is needed ?

### Step 5: Adjust the server and client deployments to consume the certificates to allow https communication

1. Modify the https server to consume the secret, the https server expect to get the crt and key as follow:
   - tls.crt: should be mounted at /app/fullchain.pem
   - tls.key: should be mounted at /app/privkey.pem

2. Modify the curl pod to use the ca certficate to turst the https server
Use curl command and manully point the certificate and test https communication 
