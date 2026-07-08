# Azure Key Vault integration for LogiPulse

This folder shows how to use Azure Key Vault with the External Secrets Operator.

## Why this is useful

- Secrets stay in Azure Key Vault instead of Git.
- Kubernetes workloads can fetch them at runtime.
- This is a strong step up from plain Secret manifests and from Sealed Secrets.

## Prerequisites

- An Azure subscription
- An Azure Key Vault
- A Kubernetes cluster with workload identity or a service principal
- The External Secrets Operator installed

## High-level flow

1. Store secrets in Azure Key Vault.
2. Create an ExternalSecret resource in Kubernetes.
3. The controller reads the values from Azure Key Vault.
4. The controller creates or updates a Kubernetes Secret.
5. Your app continues to use the same Secret name and keys.

## Example secret names in Key Vault

- `logipulse-sql-connection`
- `logipulse-asb-connection`
- `logipulse-jwt-secret`

## Example manifest

The sample manifest in this folder creates a Kubernetes Secret named `logipulse-secrets` from Azure Key Vault values.
