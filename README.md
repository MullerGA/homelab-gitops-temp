# Homelab GitOps - Repo Temporaire

Dépôt temporaire pour tester le workflow GitOps avec ArgoCD.

## Structure

```
homelab-gitops-temp/
├── apps/                        # Applications déployées
│   ├── gitea/                  # Serveur Git local
│   │   ├── deployment.yaml     # Gitea (sans DB)
│   │   ├── service.yaml        # Services NodePort
│   │   └── pvc.yaml           # Stockage Longhorn
│   └── test-app/              # App de test
│       └── nginx-test.yaml    # Nginx simple
└── infrastructure/            # Infrastructure partagée
    ├── postgresql/            # Base PostgreSQL centralisée
    │   ├── deployment.yaml    # PostgreSQL multi-DB
    │   ├── service.yaml       # Service ClusterIP
    │   ├── pvc.yaml          # Stockage 20Gi
    │   └── configmap.yaml    # Init des bases
    └── argocd-apps/          # Définitions ArgoCD
        ├── postgresql-app.yaml # App PostgreSQL
        ├── gitea-app.yaml     # App Gitea
        └── test-app.yaml      # App test
```

## Accès aux services

| Service | URL | Notes |
|---------|-----|-------|
| **Gitea** | http://192.168.1.22:30200 | Serveur Git local |
| **Gitea SSH** | ssh://192.168.1.22:30222 | Clone via SSH |
| **Test Nginx** | http://192.168.1.22:30300 | Test de déploiement |

## Configuration ArgoCD

### 1. Connecter le repo GitHub
```bash
# Dans l'interface ArgoCD (http://192.168.1.22:30100)
# Settings > Repositories > + CONNECT REPO
# Type: git
# URL: https://github.com/TON_USERNAME/homelab-gitops-temp
```

### 2. Déployer les applications
```bash
# Appliquer les définitions ArgoCD
kubectl apply -f infrastructure/argocd-apps/test-app.yaml
kubectl apply -f infrastructure/argocd-apps/gitea-app.yaml
```

### 3. Vérifier les déploiements
```bash
# Voir les apps ArgoCD
kubectl get applications -n argocd

# Voir les pods déployés
kubectl get pods -n test
kubectl get pods -n gitea
```

## Migration vers Gitea local

Une fois Gitea déployé :
1. Créer le repo `homelab-gitops` dans Gitea
2. Migrer ce contenu vers Gitea
3. Reconfigurer ArgoCD pour pointer vers Gitea
4. Supprimer ce repo GitHub temporaire

## Ressources

- **Stockage** : PVC Longhorn (8Gi Gitea + 10Gi PostgreSQL) - **Extensibles**
- **Réseau** : NodePort (30200-30300)
- **Namespaces** : `gitea`, `test`, `infrastructure`
- **Sécurité** : Secrets K8s pour tous les mots de passe

## Architecture PostgreSQL Centralisée

La base PostgreSQL dans le namespace `infrastructure` contient :
- **gitea** : Base pour Gitea (credentials via secret K8s)

**Connexion** : `postgresql.infrastructure.svc.cluster.local:5432`

## Secrets K8s

| Secret | Namespace | Contenu |
|---------|-----------|---------|
| `postgresql-admin` | infrastructure | Admin PostgreSQL |
| `postgresql-gitea` | infrastructure, gitea | Credentials Gitea DB |

## Extension de volumes

Les PVC Longhorn sont **extensibles** :
```bash
# Exemple pour étendre Gitea à 20Gi
kubectl patch pvc gitea-data -n gitea -p '{"spec":{"resources":{"requests":{"storage":"20Gi"}}}}'
``` 