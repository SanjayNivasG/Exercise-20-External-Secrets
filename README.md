# Exercise 20 – External Secrets Integration

## Objective

Integrate AWS Secrets Manager with Kubernetes using the External Secrets Operator. Store application secrets in AWS Secrets Manager and automatically synchronize them into Kubernetes Secrets.

---

## Scenario

In modern cloud-native applications, sensitive information such as database credentials and JWT secrets should not be stored directly inside Kubernetes manifests.

This exercise demonstrates how to securely manage secrets using AWS Secrets Manager and automatically synchronize them into Kubernetes with the External Secrets Operator.

---

## Architecture

```
                 AWS Secrets Manager
                         │
                         │
          Stores Application Secrets
                         │
                         ▼
         External Secrets Operator
                         │
                         │
                 SecretStore
                         │
                         ▼
                ExternalSecret
                         │
                         ▼
             Kubernetes Secret
                         │
                         ▼
                Application Pod
```

---

## Technologies Used

- AWS EC2 (Ubuntu)
- Kubernetes (Minikube)
- Docker
- Helm
- AWS CLI
- AWS Secrets Manager
- External Secrets Operator
- kubectl
- Git
- GitHub

---

## Project Structure

```
Exercise-20-External-Secrets/
│
├── README.md
├── secretstore.yaml
└── externalsecret.yaml
```

---

## Prerequisites

- AWS Account
- IAM User with Secrets Manager permissions
- AWS CLI configured
- Docker
- Minikube
- Kubernetes
- Helm

---

## AWS Secret Created

Secret Name

```
payment-app-secret
```

Secret Values

```
DB_USERNAME
DB_PASSWORD
JWT_SECRET
```

Example

```json
{
  "DB_USERNAME": "admin",
  "DB_PASSWORD": "password123",
  "JWT_SECRET": "myjwtsecret"
}
```

---

## Installation Steps

### 1. Configure AWS CLI

```bash
aws configure
```

Verify

```bash
aws sts get-caller-identity
```

---

### 2. Install External Secrets Operator

```bash
kubectl create namespace external-secrets

helm repo add external-secrets https://charts.external-secrets.io

helm repo update

helm install external-secrets external-secrets/external-secrets \
-n external-secrets \
--set installCRDs=true
```

---

### 3. Create Namespace

```bash
kubectl create namespace payment
```

---

### 4. Create AWS Credentials Secret

```bash
kubectl create secret generic aws-secret \
-n payment \
--from-literal=access-key=<AWS_ACCESS_KEY> \
--from-literal=secret-access-key=<AWS_SECRET_ACCESS_KEY>
```

---

### 5. Create SecretStore

```bash
kubectl apply -f secretstore.yaml
```

---

### 6. Create ExternalSecret

```bash
kubectl apply -f externalsecret.yaml
```

---

## Validation

Verify SecretStore

```bash
kubectl get secretstore -n payment
```

Expected

```
READY   True
STATUS  Valid
```

---

Verify ExternalSecret

```bash
kubectl get externalsecret -n payment
```

Expected

```
STATUS         READY
SecretSynced   True
```

---

Verify Kubernetes Secret

```bash
kubectl get secret -n payment
```

Expected

```
aws-secret
payment-secret
```

---

Decode Secret Values

Database Username

```bash
kubectl get secret payment-secret \
-n payment \
-o jsonpath="{.data.DB_USERNAME}" | base64 -d
```

Database Password

```bash
kubectl get secret payment-secret \
-n payment \
-o jsonpath="{.data.DB_PASSWORD}" | base64 -d
```

JWT Secret

```bash
kubectl get secret payment-secret \
-n payment \
-o jsonpath="{.data.JWT_SECRET}" | base64 -d
```

---

## Project Outcome

- Successfully created secrets in AWS Secrets Manager.
- Installed External Secrets Operator using Helm.
- Configured SecretStore for AWS authentication.
- Created an ExternalSecret resource.
- Automatically synchronized AWS secrets into Kubernetes.
- Verified secret synchronization using kubectl.
- Decoded Kubernetes secrets successfully.

---

## Troubleshooting

### SecretStore Not Found

Check CRDs

```bash
kubectl get crd | grep external
```

---

### SecretSyncedError

Verify AWS credentials

```bash
aws sts get-caller-identity
```

Restart Operator

```bash
kubectl rollout restart deployment external-secrets -n external-secrets
```

---

### Verify Pods

```bash
kubectl get pods -n external-secrets
```

---

## Skills Demonstrated

- Kubernetes Secrets
- AWS Secrets Manager
- External Secrets Operator
- Helm
- Kubernetes Custom Resources (CRDs)
- Cloud Secret Management
- DevOps Security
- Kubernetes Troubleshooting
- AWS CLI
- Git & GitHub

---

## Key Learnings

- Centralized secret management using AWS Secrets Manager.
- Secure synchronization of secrets into Kubernetes.
- Avoid storing credentials inside Kubernetes manifests.
- Validate and troubleshoot External Secrets synchronization.
- Manage Kubernetes secrets using industry-standard practices.

---

## Interview Questions

### Why use AWS Secrets Manager?

To securely store and centrally manage sensitive application credentials.

---

### Why use External Secrets Operator?

It automatically synchronizes secrets from external secret stores into Kubernetes.

---

### What is SecretStore?

SecretStore defines how the External Secrets Operator authenticates with the external secret provider.

---

### What is ExternalSecret?

ExternalSecret maps data from AWS Secrets Manager into a Kubernetes Secret.

---

### Why not store passwords directly in Kubernetes?

Because Kubernetes Secrets are only Base64 encoded and are not intended to be the primary secure storage solution. Using AWS Secrets Manager improves security and simplifies secret rotation.

---

## Conclusion

This exercise demonstrates secure secret management in Kubernetes by integrating AWS Secrets Manager with the External Secrets Operator. Secrets are maintained centrally in AWS and synchronized automatically into Kubernetes, providing a scalable and production-oriented approach to managing sensitive application data.
