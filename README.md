✅ Step 1: Generate a Self-Signed Certificate and Key
You can use openssl to generate the certificate and key:
```
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout tls.key \
  -out tls.crt \
  -subj "/CN=posit-connect-eks-poc.local/O=Self-Signed"
```
✅ Step 2: Create the TLS Secret in Kubernetes

```
kubectl create secret tls rstudio-connect-tls \
  --namespace rstudio-connect \
  --cert=tls.crt \
  --key=tls.key
```
✅ Step 3: Verify the Secret
```
kubectl get secret rstudio-connect-tls -n rstudio-connect
```

```
================================
Create a TLS Secret:
kubectl create secret tls rstudio-connect-tls \
  --namespace rstudio-connect \
  --cert=path/to/tls.crt \
  --key=path/to/tls.key
========================================
Ingress resource with TLS configured:

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: rstudio-connect-ingress
  namespace: rstudio-connect
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  tls:
    - hosts:
        - posit-connect-eks-poc.local  # Replace with your actual domain
      secretName: rstudio-connect-tls
  rules:
    - host: posit-connect-eks-poc.local
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: rstudio-connect-prod
                port:
                  number: 80
=================================================
kubectl apply -f rstudio-connect-ingress.yaml
================================================
```
# Automate this using bash script
``
#!/bin/bash

# Domain and namespace variables
DOMAIN="posit-connect-eks-poc.local"
NAMESPACE="rstudio-connect"
SECRET_NAME="rstudio-connect-tls"

# Generate self-signed certificate and key
openssl req -x509 -nodes -days 365 -newkey rsa:2048   -keyout tls.key   -out tls.crt   -subj "/CN=$DOMAIN/O=Self-Signed"

# Create Kubernetes namespace if it doesn't exist
kubectl get namespace $NAMESPACE >/dev/null 2>&1 || kubectl create namespace $NAMESPACE

# Create TLS secret in Kubernetes
kubectl create secret tls $SECRET_NAME   --namespace $NAMESPACE   --cert=tls.crt   --key=tls.key

# Confirm secret creation
kubectl get secret $SECRET_NAME -n $NAMESPACE
``
