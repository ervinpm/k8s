# Sample NGINX app

This assumes that you already have the following:
1. cert-manager with Lets Encrypt Certificate Issuer
2. ingress controller

### Application Deployment

Deploy the nginx deployment

```bash
$ kubectl apply -f nginx-app-deployment.yaml
```

### Certificate

Create the certificate that will be used for TLS by ingress

```bash
$ kubectl apply -f nginx-app-certificate.yaml
```

### Service

Install the service

```bash
$ kubectl apply -f nginx-app-service.yaml
```

### Create the ingress configuration

```bash
$ kubectl apply -f nginx-app-ingress.yaml
```
