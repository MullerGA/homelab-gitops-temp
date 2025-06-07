# Configuration Certificats SSL

## 🔑 Prérequis : Clés API OVH

**⚠️ IMPORTANT** : Les clés API OVH ne sont **PAS** dans ce repo public pour des raisons de sécurité.

### 1. Générer les clés API OVH

1. Aller sur : https://eu.api.ovh.com/createToken/
2. **Application name** : `K3s-Cert-Manager`
3. **Application description** : `Certificats SSL pour homelab`
4. **Validity** : `Unlimited`
5. **Rights** :
   ```
   GET    /domain/zone/gmladata.fr/record
   GET    /domain/zone/gmladata.fr/record/*
   POST   /domain/zone/gmladata.fr/record
   DELETE /domain/zone/gmladata.fr/record/*
   POST   /domain/zone/gmladata.fr/refresh
   ```

### 2. Créer le secret dans K8s

```bash
# SSH sur le serveur K3s
kubectl create namespace cert-manager-webhook-ovh

kubectl create secret generic ovh-credentials \
  --namespace cert-manager-webhook-ovh \
  --from-literal=applicationKey="TA_CLE_APPLICATION" \
  --from-literal=applicationSecret="TON_SECRET_APPLICATION" \
  --from-literal=consumerKey="TA_CLE_CONSUMER" \
  --from-literal=endpoint="ovh-eu"
```

### 3. Vérifier le secret

```bash
kubectl get secret ovh-credentials -n cert-manager-webhook-ovh -o yaml
```

## 🚀 Déploiement

Une fois le secret créé manuellement, ArgoCD peut déployer le reste :
- OVH Webhook
- ClusterIssuer 
- Certificat wildcard 