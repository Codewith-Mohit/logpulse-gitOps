# Sealed Secrets for LogiPulse

This folder shows how to move from plain Kubernetes Secret manifests to Sealed Secrets for a safer GitOps workflow.

## Why this is better

- Secrets are encrypted before they are committed to Git.
- The cluster-side controller decrypts them when applying to Kubernetes.
- Your deployment templates can continue using the same Secret name and keys.

## Recommended choice for this project

For learning and small/self-hosted setups, Sealed Secrets is the best first step.

If you later want cloud-native secret management, you can move to:
- Azure Key Vault + External Secrets Operator
- HashiCorp Vault

## Step-by-step setup

### 1. Install the Sealed Secrets controller

```bash
kubectl apply -f https://github.com/bitnami-labs/sealed-secrets/releases/download/v0.27.0/controller.yaml
```

Verify it is running:

```bash
kubectl get pods -n kube-system -l app.kubernetes.io/name=sealed-secrets
```

### 2. Install the kubeseal CLI

On Linux:

```bash
wget https://github.com/bitnami-labs/sealed-secrets/releases/download/v0.27.0/kubeseal-0.27.0-linux-amd64.tar.gz

tar -xvf kubeseal-0.27.0-linux-amd64.tar.gz
sudo install -m 755 kubeseal /usr/local/bin/kubeseal
```

### 3. Create a temporary plaintext secret

```bash
kubectl create secret generic logipulse-secrets \
  --from-literal=sql-connection='YOUR_SQL_CONNECTION_STRING' \
  --from-literal=asb-connection='YOUR_ASB_CONNECTION_STRING' \
  --from-literal=jwt-secret='YOUR_JWT_SECRET' \
  --namespace logipulse \
  --dry-run=client -o yaml > /tmp/logipulse-secret.yaml
```

### 4. Encrypt it with kubeseal

```bash
kubeseal --format yaml < /tmp/logipulse-secret.yaml > base/sealed-secrets/logipulse-secrets.yaml
```

### 5. Commit the generated sealed secret

The generated file can be committed to Git because it contains encrypted data, not plaintext secrets.

### 6. Apply it to the cluster

```bash
kubectl apply -f base/sealed-secrets/logipulse-secrets.yaml
```

The controller will decrypt it and create the regular Secret object.

## Important note

Your existing deployment files already reference the secret name and keys used by the app. No change is required in the workload manifests if the Secret ends up with the same name and keys.

## Azure Key Vault option (later)

If you want cloud-based secret management, the next step is:
- External Secrets Operator
- Azure Key Vault
- Azure workload identity or service principal

This is more production-ready, but it requires more setup than Sealed Secrets.
