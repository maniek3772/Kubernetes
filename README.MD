# MatrApp Kubernetes Deployment

This project contains Kubernetes manifests to deploy **MatrApp** (a Flask application) together with a **PostgreSQL** database.  
The configuration includes security best practices, RBAC, NetworkPolicies, PersistentVolumes, and an Ingress with TLS.  

## Overview

The manifest defines the following resources:

- **Namespace** `matr` with Pod Security Standards (restricted).
- **ServiceAccount**, **Role**, and **RoleBinding** granting the app access only to its secret.
- **Secret** `matr-db-secret` storing PostgreSQL credentials.
- **Services** for the application (`matrapp`) and PostgreSQL (`postgres`).
- **StorageClass** + **PersistentVolumeClaim** for database persistence.
- **StatefulSet** for running PostgreSQL with a persistent volume.
- **Deployment** for the MatrApp Flask application.
- **NetworkPolicies**:
  - `default-deny` (blocks all traffic by default),
  - allows the app to connect to PostgreSQL,
  - egress rules for the app (e.g., internet access).
- **Ingress** with TLS (NGINX ingress controller) to expose the app externally.

## Requirements

- Kubernetes ≥ 1.25
- Installed **NGINX Ingress Controller**
- **cert-manager** if you want automatic TLS
- A cluster with **Azure Disk CSI driver** (`disk.csi.azure.com`)

## Deployment

1. Clone the repository:
   ```bash
   git clone https://github.com/maniek3772/Kubernetes.git
   cd Kubernetes
   ```

2. Edit the Ingress manifest to set your own host instead of the example:
   ```
   https://matrapp.20.253.205.154.nip.io
   ```

3. Generate or provide your own TLS certificate and create the secret:
   ```bash
   kubectl create secret tls matrapp-tls --cert=matrapp.crt --key=matrapp.key -n matr
   ```

4. Apply the manifests:
   ```bash
   kubectl apply -f deployment.yaml
   ```

5. Check if pods are running:
   ```bash
   kubectl get pods -n matr
   ```

6. Once ready, the app will be available at your chosen host:
   ```
   https://<your-domain>
   ```

## Secrets

Database credentials are stored in the Kubernetes `Secret`:
```yaml
POSTGRES_USER: matruser
POSTGRES_PASSWORD: matrpassword
POSTGRES_DB: matrdb
```

You can modify these before applying the manifests.

## Security Features

- **Pod Security Standards** (restricted) enabled at the namespace level.
- RBAC rules – the app can only access its database secret.
- **NetworkPolicies** isolate traffic – only the app can connect to PostgreSQL.
- Containers run as **non-root**, with minimal privileges.
- PostgreSQL uses a **PersistentVolume**, so data survives pod restarts.