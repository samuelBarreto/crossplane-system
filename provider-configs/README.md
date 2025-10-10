# Provider Configurations

Este diretório contém as configurações de credenciais para os providers Crossplane.

## ⚠️ IMPORTANTE - Segurança

**NUNCA commite credenciais em plain text no Git!**

### Métodos Recomendados para Gerenciar Secrets

#### 1. External Secrets Operator (Recomendado)

```yaml
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: aws-credentials
  namespace: crossplane-system
spec:
  refreshInterval: 1h
  secretStoreRef:
    name: aws-secrets-manager
    kind: SecretStore
  target:
    name: aws-credentials
    creationPolicy: Owner
  data:
  - secretKey: credentials
    remoteRef:
      key: crossplane/aws-credentials
```

#### 2. Sealed Secrets

```bash
# Criar secret
kubectl create secret generic aws-credentials \
  --from-file=credentials=./aws-creds.txt \
  --dry-run=client -o yaml | \
  kubeseal -o yaml > sealed-secret-aws.yaml

# Commitar sealed secret
git add sealed-secret-aws.yaml
```

#### 3. ArgoCD Vault Plugin

Integração com HashiCorp Vault para gerenciar secrets.

## Configuração por Ambiente

### Development
- Use credenciais com permissões limitadas
- Prefira service accounts específicos

### Production
- Use credenciais separadas para prod
- Habilite auditoria e logging
- Rotate credenciais regularmente

## Exemplo de Configuração

### AWS

```bash
# Criar secret manualmente (apenas para dev local)
kubectl create secret generic aws-credentials \
  -n crossplane-system \
  --from-literal=credentials="$(cat <<EOF
[default]
aws_access_key_id = YOUR_KEY
aws_secret_access_key = YOUR_SECRET
EOF
)"
```

### Azure

```bash
# Criar service principal
az ad sp create-for-rbac \
  --name crossplane-sp \
  --role Contributor \
  --scopes /subscriptions/YOUR_SUBSCRIPTION_ID

# Criar secret
kubectl create secret generic azure-credentials \
  -n crossplane-system \
  --from-literal=credentials='{"clientId":"...","clientSecret":"...","subscriptionId":"...","tenantId":"..."}'
```

### GCP

```bash
# Criar service account key
gcloud iam service-accounts keys create crossplane-key.json \
  --iam-account=crossplane@PROJECT.iam.gserviceaccount.com

# Criar secret
kubectl create secret generic gcp-credentials \
  -n crossplane-system \
  --from-file=credentials=crossplane-key.json
```

