# 🔐 Kubernetes External Secrets with AWS Secrets Manager

This project demonstrates how to securely manage application secrets in Amazon EKS using **AWS Secrets Manager**, **IAM Roles for Service Accounts (IRSA)**, and **External Secrets Operator (ESO)**.

Instead of storing sensitive credentials directly inside Kubernetes manifests, secrets are securely stored in AWS Secrets Manager and automatically synchronized into Kubernetes.

---

# 📌 Project Overview

This lab demonstrates how to:

- Store secrets inside AWS Secrets Manager
- Configure IAM Roles for Service Accounts (IRSA)
- Install External Secrets Operator
- Synchronize AWS Secrets Manager secrets into Kubernetes
- Automatically create Kubernetes Secrets
- Perform secret rotation
- Understand production secret rotation incidents

---

# 🏗 Architecture

```
                    AWS Cloud

             AWS Secrets Manager
         ----------------------------
         DB_USERNAME
         DB_PASSWORD
         JWT_SECRET
                  │
                  │
          IAM Role (IRSA)
                  │
                  ▼
      External Secrets Operator
                  │
                  ▼
        ClusterSecretStore
                  │
                  ▼
         ExternalSecret Resource
                  │
                  ▼
         Kubernetes Secret
                  │
                  ▼
            Application Pod
```

---

# 🛠 Technologies Used

- Amazon EKS
- Kubernetes
- AWS Secrets Manager
- External Secrets Operator
- Helm
- IAM
- IAM Roles for Service Accounts (IRSA)
- OIDC Provider
- AWS CLI
- eksctl

---

# 📂 Project Structure

```
ex13/
│
├── aws/
│   ├── iam/
│   │   └── secrets-policy.json
│   │
│   └── secrets/
│       └── payment-secret.json
│
├── k8s/
│   ├── clustersecretstore.yaml
│   ├── externalsecret.yaml
│   ├── deployment.yaml
│   └── service.yaml
│
└── README.md
```

---

# Exercise 20 – External Secrets Integration

## Objective

Integrate AWS Secrets Manager with Kubernetes using External Secrets Operator.

---

## Secret Stored in AWS

The following values were stored inside AWS Secrets Manager.

```json
{
    "DB_USERNAME": "admin",
    "DB_PASSWORD": "password123",
    "JWT_SECRET": "my-super-secret"
}
```

Secret Name

```
payment-secret
```

---

## Installing External Secrets Operator

Installed using Helm.

```
helm repo add external-secrets https://charts.external-secrets.io

helm install external-secrets external-secrets/external-secrets \
-n external-secrets
```

---

## Components Installed

Verified

```
external-secrets
external-secrets-webhook
external-secrets-cert-controller
```

---

## CRDs Installed

- ExternalSecret
- SecretStore
- ClusterSecretStore
- ClusterExternalSecret

and other supporting CRDs.

---

# IAM Roles for Service Accounts (IRSA)

Instead of using AWS Access Keys, the External Secrets Operator authenticates using IRSA.

Benefits

- No static AWS credentials
- Temporary AWS credentials
- More secure
- AWS Best Practice

---

## IAM Policy

Created an IAM Policy allowing only

```
secretsmanager:GetSecretValue

secretsmanager:DescribeSecret
```

Following the Principle of Least Privilege.

---

## IAM Role

Created using

```
eksctl create iamserviceaccount
```

The ServiceAccount was automatically linked with an IAM Role.

Verified

```
kubectl describe sa external-secrets
```

---

# ClusterSecretStore

Created a ClusterSecretStore to connect External Secrets Operator with AWS Secrets Manager.

Purpose

- Authenticate to AWS
- Select AWS Region
- Define Secrets Manager provider

Verified

```
kubectl get clustersecretstore
```

Status

```
Ready = True
```

---

# ExternalSecret

Created an ExternalSecret resource.

Purpose

Synchronize

```
AWS Secrets Manager

↓

Kubernetes Secret
```

Automatically.

Configured properties

```
DB_USERNAME

DB_PASSWORD

JWT_SECRET
```

---

# Validation

Verified

```
kubectl get externalsecret
```

Result

```
READY=True
```

Verified Kubernetes Secret

```
kubectl get secret
```

Result

```
payment-secret
```

---

# Secret Synchronization

Automatic synchronization flow

```
AWS Secrets Manager

↓

External Secrets Operator

↓

Kubernetes Secret
```

No manual Kubernetes Secret creation required.

---

# Secret Rotation

Updated the AWS Secret

Old

```
JWT_SECRET=my-super-secret
```

New

```
JWT_SECRET=new-super-secret
```

External Secrets Operator automatically synchronized the updated values into Kubernetes based on the configured refresh interval.

---

# Exercise 13 – Secret Rotation Outage

## Incident

Following AWS Secrets Manager rotation

```
401 Unauthorized

Token validation failed
```

---

## Root Cause

AWS Secret was rotated.

↓

External Secrets Operator updated the Kubernetes Secret.

↓

Applications continued using cached secrets or clients continued using JWT tokens signed with the previous secret.

↓

Authentication failed.

---

## Production Impact

Possible issues

- User login failures
- Invalid JWT tokens
- Authentication outage
- Service downtime

---

## Production Solutions

### Graceful Key Rotation

Support both

```
Old JWT Secret

+

New JWT Secret
```

during migration.

---

### Rolling Restart

Restart application pods after secret updates.

Example

```
kubectl rollout restart deployment
```

---

### Secret Reload

Applications should reload secrets without requiring manual intervention.

---

### Short-lived Tokens

Use

- Short JWT expiration
- Refresh Tokens

to reduce authentication failures after rotation.

---

# Validation Commands

Verify External Secret

```bash
kubectl get externalsecret
```

---

Verify ClusterSecretStore

```bash
kubectl get clustersecretstore
```

---

Verify Kubernetes Secret

```bash
kubectl get secret
```

---

Describe External Secret

```bash
kubectl describe externalsecret payment-secret
```

---

Decode Secret

```bash
kubectl get secret payment-secret -o jsonpath="{.data.JWT_SECRET}"
```

---

List AWS Secrets

```bash
aws secretsmanager list-secrets
```

---

View Secret

```bash
aws secretsmanager get-secret-value \
--secret-id payment-secret
```

---

Rotate Secret

```bash
aws secretsmanager put-secret-value \
--secret-id payment-secret \
--secret-string file://payment-secret.json
```

---

# Concepts Learned

- AWS Secrets Manager
- Secret Versioning
- Secret Rotation
- External Secrets Operator
- SecretStore
- ClusterSecretStore
- ExternalSecret
- Kubernetes Secrets
- IRSA
- OIDC Provider
- IAM Roles
- IAM Policies
- Helm Installation
- Kubernetes Controllers
- Production Secret Management

---

# Security Best Practices

✔ Do not store passwords inside Kubernetes YAML.

✔ Use AWS Secrets Manager as the source of truth.

✔ Authenticate using IRSA instead of AWS Access Keys.

✔ Follow the Principle of Least Privilege.

✔ Rotate secrets periodically.

✔ Automate secret synchronization using External Secrets Operator.

---

# Outcome

Successfully built a production-ready secret management solution for Amazon EKS using AWS Secrets Manager and External Secrets Operator.

The environment securely synchronizes secrets from AWS into Kubernetes without storing sensitive credentials inside Kubernetes manifests and supports automatic secret rotation, making it suitable for modern cloud-native applications.