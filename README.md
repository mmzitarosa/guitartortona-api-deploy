# 🎸 Guitar Tortona API - Deployment Configuration

GitOps deployment configuration per Guitar Tortona API su Kubernetes.

## 🌍 Environments

### Staging
- **Namespace:** `staging`
- **URL:** https://api-staging.guitar.lab
- **Database:** `guitartortona_staging` on `mariadb.databases.svc.cluster.local`
- **Branch Git:** `staging`
- **Deploy:** Automatico da `develop` branch dell'applicazione
- **Image tag:** Versioning automatico (es: `1.2.0-rc3`)

### Production
- **Namespace:** `default`
- **URL:** https://api.guitar.lab
- **Database:** `guitartortona` on `mariadb.databases.svc.cluster.local`
- **Branch Git:** `main`
- **Deploy:** Manuale via promozione da staging
- **Image tag:** Versione stabile (es: `1.2.0`)

## 🔄 Branch Strategy
```
staging branch → ArgoCD → namespace staging
main branch    → ArgoCD → namespace default (production)
```

## 🚀 Workflow

### Deploy automatico staging:

1. Developer pusha su `guitartortona-api` branch `develop` con commit `RELEASE:` o `MAJOR:`
2. GitHub Actions:
   - Build Maven
   - Build Docker image
   - Push su GHCR con tag versioned + `staging`
   - Clone questo repository (branch `staging`)
   - Aggiorna `staging/deployment.yaml` con nuovo tag
   - Commit e push
3. ArgoCD rileva cambio e deploya automaticamente su namespace `staging`

### Promozione a production:

1. Test su staging OK
2. Workflow manuale `promote-production` in `guitartortona-api`
3. GitHub Actions:
   - Retag immagine Docker: `X.Y.Z-rcN` → `X.Y.Z` + `latest`
   - Clone questo repository (branch `main`)
   - Aggiorna `production/deployment.yaml`
   - Commit e push
4. ArgoCD rileva cambio e deploya su namespace `default` (production)

## 📂 Struttura
```
├── staging/
│   ├── deployment.yaml           # Pod configuration
│   ├── service.yaml              # ClusterIP service
│   ├── ingress.yaml              # HTTPS ingress
│   └── secrets/
│       ├── external-secret-ghcr.yaml      # GHCR credentials
│       └── external-secret-mariadb.yaml   # DB credentials
└── production/
    └── ... (same structure)
```

## 🔐 Secrets

Gestiti tramite:
1. **HashiCorp Vault** - Storage sicuro
2. **External Secrets Operator** - Sync automatico a Kubernetes
3. **Git** - Solo definizioni ExternalSecret (NO secrets reali!)

### Secrets utilizzati:

- `ghcr-login-secret` - Credentials per pullare immagini Docker da GitHub Container Registry
- `mariadb-credentials` - Username e password per database

### Path in Vault:
```
secret/guitartortona/staging/ghcr/{username,password}
secret/guitartortona/staging/mariadb/{username,password}
secret/guitartortona/production/ghcr/{username,password}
secret/guitartortona/production/mariadb/{username,password}
```

## 🔍 Verifica Deployment
```bash
# Stato ArgoCD
kubectl get application guitartortona-api-staging -n argocd
kubectl get application guitartortona-api-production -n argocd

# Pods
kubectl get pods -n staging -l app=guitartortona-api
kubectl get pods -n default -l app=guitartortona-api

# Health check
curl -k https://api-staging.guitar.lab/actuator/health
curl -k https://api.guitar.lab/actuator/health

# Logs
kubectl logs -n staging -l app=guitartortona-api --tail=100
kubectl logs -n default -l app=guitartortona-api --tail=100
```

## 📖 Riferimenti

- Repository infrastruttura: [guitartortona-infrastructure](https://github.com/mmzitarosa/guitartortona-infrastructure)
- Repository applicazione: [guitartortona-api](https://github.com/mmzitarosa/guitartortona-api)
- ArgoCD UI: https://argocd.guitar.lab
- Grafana: https://grafana.guitar.lab
