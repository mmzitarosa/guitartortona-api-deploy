# Staging Secrets

Secrets per ambiente staging, gestiti tramite External Secrets Operator.

## Secrets

- `ghcr-login-secret` - Credentials GitHub Container Registry
- `mariadb-credentials` - Credentials database staging

## Vault Paths
```
secret/guitartortona/staging/ghcr/username
secret/guitartortona/staging/ghcr/password
secret/guitartortona/staging/mariadb/username
secret/guitartortona/staging/mariadb/password
```

## Setup

1. Popolare Vault con i secrets:
```bash
kubectl exec -it vault-0 -n vault -- sh -c "
export VAULT_TOKEN=<ROOT_TOKEN>

vault kv put secret/guitartortona/staging/ghcr \
  username='mmzitarosa' \
  password='ghp_XXX...'

vault kv put secret/guitartortona/staging/mariadb \
  username='guitartortona_staging' \
  password='01VdjEyA8R9TH1Wz'
"
```

2. Verificare sincronizzazione:
```bash
kubectl get externalsecret -n staging
kubectl get secret ghcr-login-secret -n staging
kubectl get secret mariadb-credentials -n staging
```
