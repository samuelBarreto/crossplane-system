# Crossplane System

<div align="center">

![Crossplane](https://crossplane.io/images/crossplane-icon-color.svg)

**Repositório GitOps para gerenciamento de infraestrutura Crossplane**

[![Crossplane](https://img.shields.io/badge/Crossplane-v1.14.5-blue)](https://crossplane.io)
[![ArgoCD](https://img.shields.io/badge/ArgoCD-compatible-green)](https://argoproj.github.io/cd/)
[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](LICENSE)

</div>

---

## 📋 Sobre o Projeto

Este repositório contém os manifestos Kubernetes para instalação e configuração do **Crossplane** e seus providers, gerenciados via **GitOps** com ArgoCD.

O Crossplane é um framework de orquestração de infraestrutura que permite provisionar e gerenciar recursos cloud (AWS, Azure, GCP) usando APIs nativas do Kubernetes.

## 🏗️ Estrutura do Repositório

```
crossplane-system/
├── install/                      # Instalação do Crossplane Core
│   └── crossplane-install.yaml   # Deployment, RBAC, ServiceAccount
│
├── providers/                    # Providers por cloud provider
│   ├── aws/                      # Providers AWS
│   │   └── provider-aws.yaml     # EC2, RDS, S3, VPC, IAM
│   ├── gcp/                      # Providers GCP
│   │   └── provider-gcp.yaml     # Compute, SQL, Storage
│   └── azure/                    # Providers Azure
│       └── provider-azure.yaml   # Compute, DB, Storage, Network
│
└── provider-configs/             # Configurações de credenciais
    ├── provider-config-aws.yaml
    ├── provider-config-azure.yaml
    └── README.md                 # Guia de segurança para secrets
```

## 🚀 Como Usar

### Opção 1: Via ArgoCD (Recomendado)

Este repositório foi projetado para ser usado com **ArgoCD Applications**.

```yaml
---
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: crossplane-core
  namespace: argocd
spec:
  project: platform
  source:
    repoURL: https://github.com/samuelBarreto/crossplane-system.git
    targetRevision: main
    path: install/
  destination:
    server: https://kubernetes.default.svc
    namespace: crossplane-system
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
    - CreateNamespace=true
```

Veja exemplos completos em: [Repositório Principal](https://github.com/your-org/crossplane-platform)

### Opção 2: Kubectl Direto

```bash
# 1. Instalar Crossplane Core
kubectl apply -f install/crossplane-install.yaml

# 2. Aguardar Crossplane ficar pronto
kubectl wait --for=condition=Ready pod -l app=crossplane \
  -n crossplane-system --timeout=300s

# 3. Instalar Providers (escolha seu cloud)
kubectl apply -f providers/aws/provider-aws.yaml
# ou
kubectl apply -f providers/gcp/provider-gcp.yaml
# ou
kubectl apply -f providers/azure/provider-azure.yaml

# 4. Aguardar Providers ficarem HEALTHY
kubectl get providers -w
```

## 🔧 Providers Disponíveis

### AWS Providers
- **provider-aws-ec2** `v1.1.0` - EC2, EBS, AMIs
- **provider-aws-rds** `v1.1.0` - RDS, Aurora, Snapshots
- **provider-aws-s3** `v1.1.0` - S3 Buckets, Objects
- **provider-aws-vpc** `v1.1.0` - VPC, Subnets, Security Groups
- **provider-aws-iam** `v1.1.0` - IAM Roles, Policies

### GCP Providers
- **provider-gcp-compute** `v1.0.0` - Compute Engine, Disks
- **provider-gcp-sql** `v1.0.0` - Cloud SQL
- **provider-gcp-storage** `v1.0.0` - Cloud Storage

### Azure Providers
- **provider-azure-compute** `v1.0.0` - Virtual Machines
- **provider-azure-dbforpostgresql** `v1.0.0` - PostgreSQL Database
- **provider-azure-storage** `v1.0.0` - Storage Accounts
- **provider-azure-network** `v1.0.0` - Virtual Networks

## 🔐 Configuração de Credenciais

⚠️ **IMPORTANTE**: Este repositório **NÃO** contém credenciais em plain text.

Para configurar credenciais, consulte: [`provider-configs/README.md`](provider-configs/README.md)

**Métodos suportados:**
1. ✅ External Secrets Operator (Recomendado para produção)
2. ✅ Sealed Secrets
3. ✅ HashiCorp Vault via ArgoCD Vault Plugin
4. ⚠️ kubectl create secret (apenas desenvolvimento local)

## 📦 Componentes

### Crossplane Core (v1.14.5)

O Crossplane Core inclui:
- **CRDs**: XRD, Composition, Claim
- **Controllers**: Composition, Claim, Package Manager
- **Webhooks**: Validação e conversão
- **Features habilitadas**:
  - ✅ Composition Webhook Schema Validation
  - ✅ Composition Functions

### Resources Limits

```yaml
resources:
  requests:
    cpu: 100m
    memory: 256Mi
  limits:
    cpu: 500m
    memory: 512Mi
```

## 🔄 Versionamento e Updates

### Versões Atuais
- **Crossplane Core**: `v1.14.5`
- **AWS Providers**: `v1.1.0`
- **GCP Providers**: `v1.0.0`
- **Azure Providers**: `v1.0.0`

### Como Atualizar Providers

```bash
# Editar versão no arquivo
vim providers/aws/provider-aws.yaml

# Atualizar a versão do package
# De: xpkg.upbound.io/upbound/provider-aws-rds:v1.1.0
# Para: xpkg.upbound.io/upbound/provider-aws-rds:v1.2.0

# Commit e push (ArgoCD fará o sync automático)
git add providers/aws/provider-aws.yaml
git commit -m "chore: update AWS RDS provider to v1.2.0"
git push
```

## 🎯 Integração com ArgoCD

Este repositório funciona como **source** para 3 Applications ArgoCD:

1. **crossplane-core** → `install/`
2. **crossplane-providers** → `providers/`
3. **provider-configs** → `provider-configs/`

### Sync Waves Recomendadas

```yaml
# 1. Core (wave 0)
argocd.argoproj.io/sync-wave: "0"

# 2. Providers (wave 1)
argocd.argoproj.io/sync-wave: "1"

# 3. Provider Configs (wave 2)
argocd.argoproj.io/sync-wave: "2"

# 4. XRDs e Compositions (wave 3) - outro repositório
argocd.argoproj.io/sync-wave: "3"
```

## 📊 Health Checks

### Crossplane Core

```bash
# Verificar pod
kubectl get pods -n crossplane-system -l app=crossplane

# Verificar logs
kubectl logs -n crossplane-system -l app=crossplane --tail=50

# Verificar readiness
kubectl get deployment crossplane -n crossplane-system
```

### Providers

```bash
# Listar todos providers
kubectl get providers

# Status detalhado
kubectl describe provider provider-aws-rds

# Expected Output:
# NAME               INSTALLED   HEALTHY   PACKAGE
# provider-aws-rds   True        True      xpkg.upbound.io/upbound/provider-aws-rds:v1.1.0
```

## 🐛 Troubleshooting

### Provider não fica Healthy

```bash
# Ver logs do provider
kubectl logs -n crossplane-system -l pkg.crossplane.io/provider=provider-aws-rds

# Verificar ProviderConfig
kubectl get providerconfig
kubectl describe providerconfig default

# Verificar secret de credenciais
kubectl get secret aws-credentials -n crossplane-system
```

### Crossplane Core crashando

```bash
# Ver eventos
kubectl get events -n crossplane-system --sort-by='.lastTimestamp'

# Logs detalhados
kubectl logs -n crossplane-system deployment/crossplane --previous

# Verificar recursos
kubectl top pod -n crossplane-system
```

## 🔒 Segurança

### Princípios

1. ✅ **Least Privilege**: Credenciais com permissões mínimas necessárias
2. ✅ **Secrets Management**: Nunca commitar credenciais no Git
3. ✅ **Audit Logging**: Habilitar logs de auditoria
4. ✅ **Rotation**: Rotacionar credenciais regularmente
5. ✅ **RBAC**: Controle de acesso baseado em roles

### Boas Práticas

```yaml
# ❌ NUNCA FAZER
apiVersion: v1
kind: Secret
metadata:
  name: aws-credentials
data:
  credentials: BASE64_ENCODED_CREDS  # Não commitar!

# ✅ USAR
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
spec:
  secretStoreRef:
    name: aws-secrets-manager
```

## 📚 Referências

- [Documentação Crossplane](https://docs.crossplane.io)
- [Upbound Provider Docs](https://marketplace.upbound.io)
- [ArgoCD Best Practices](https://argo-cd.readthedocs.io/en/stable/user-guide/best_practices/)
- [Crossplane Community](https://github.com/crossplane/crossplane)

## 🤝 Contribuindo

### Workflow

1. Fork o repositório
2. Crie uma branch: `git checkout -b feature/new-provider`
3. Faça suas alterações
4. Teste localmente
5. Commit: `git commit -m "feat: add AWS EKS provider"`
6. Push: `git push origin feature/new-provider`
7. Abra um Pull Request

### Commits Convention

Seguimos [Conventional Commits](https://www.conventionalcommits.org/):

- `feat:` nova funcionalidade
- `fix:` correção de bug
- `chore:` mudanças em ferramentas/configurações
- `docs:` mudanças em documentação
- `refactor:` refatoração de código

## 📄 Licença

Apache License 2.0 - veja [LICENSE](LICENSE) para detalhes.

## 📧 Suporte

- **Issues**: [GitHub Issues](https://github.com/samuelBarreto/crossplane-system/issues)
- **Discussions**: [GitHub Discussions](https://github.com/samuelBarreto/crossplane-system/discussions)
- **Email**: platform-team@example.com

---

<div align="center">

**Construído com ❤️ pela Platform Engineering Team**

</div>

