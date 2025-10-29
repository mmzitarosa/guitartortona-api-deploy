# Production Secrets

Secrets per ambiente production, gestiti tramite External Secrets Operator.

## Secrets

- `ghcr-login-secret` - Credentials GitHub Container Registry
- `mariadb-credentials` - Credentials database production

## Vault Paths
```
secret/guitartortona/production/ghcr/username
secret/guitartortona/production/ghcr/password
secret/guitartortona/production/mariadb/username
secret/guitartortona/production/mariadb/password
```

## Setup

1. Popolare Vault con i secrets:
```bash
kubectl exec -it vault-0 -n vault -- sh -c "
export VAULT_TOKEN=<ROOT_TOKEN>

vault kv put secret/guitartortona/production/ghcr \
  username='mmzitarosa' \
  password='ghp_XXX...'

vault kv put secret/guitartortona/production/mariadb \
  username='guitartortona' \
  password='QQuXxGldv0P1y2Xh'
"
```

2. Verificare sincronizzazione:
```bash
kubectl get externalsecret -n default
kubectl get secret ghcr-login-secret -n default
kubectl get secret mariadb-credentials -n default
```
