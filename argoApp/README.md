# ArgoCD Application - Nginx Helm Chart

Deploy da aplicação nginx via Helm chart gerenciado pelo ArgoCD.

## Estrutura

```
helmPlaygroud/
├── L&L/                        # Helm chart da aplicação
│   ├── Chart.yaml
│   ├── values.yaml
│   └── templates/
│       ├── deployment.yaml
│       ├── service.yaml
│       ├── configmap.yaml
│       └── _helpers.tpl
└── argoApp/
    ├── yaml.yaml               # ArgoCD Application manifest
    ├── namespace.yaml          # Namespace da aplicação
    └── README.md
```

## Pré-requisitos

- Cluster Kubernetes rodando
- ArgoCD instalado no namespace `argocd`
- Repositório git acessível pelo ArgoCD

## Configuração

### 1. Subir o repositório para o git

Certifique-se que o projeto está em um repositório git acessível pelo ArgoCD.

### 2. Atualizar o repoURL

Edite [yaml.yaml](yaml.yaml) e substitua o `repoURL` pelo endereço do seu repositório:

```yaml
source:
  repoURL: https://github.com/<seu-usuario>/helmPlaygroud.git
```

### 3. Aplicar os manifests

```bash
# Criar o namespace
kubectl apply -f argoApp/namespace.yaml

# Criar a ArgoCD Application
kubectl apply -f argoApp/yaml.yaml
```

### 4. Verificar o deploy

```bash
# Via kubectl
kubectl get application nginx-helm-app -n argocd

# Via ArgoCD CLI
argocd app get nginx-helm-app

# Sincronizar manualmente (se necessário)
argocd app sync nginx-helm-app
```

## Sync Policy

A aplicação está configurada com `automated sync`:

| Opção        | Valor | Descrição                                      |
|--------------|-------|------------------------------------------------|
| `prune`      | true  | Remove recursos deletados do git               |
| `selfHeal`   | true  | Reverte mudanças manuais no cluster            |
| `CreateNamespace` | true | Cria o namespace automaticamente          |

## Customizar valores do Helm

Para sobrescrever valores do chart, adicione a seção `helm.values` no [yaml.yaml](yaml.yaml):

```yaml
spec:
  source:
    helm:
      values: |
        replicaCount: 2
        configmap:
          html: |
            <html><body><h1>Minha App</h1></body></html>
```

## Verificar a aplicação no cluster

```bash
kubectl get pods -n nginx-app
kubectl get svc -n nginx-app

# Port-forward para testar localmente
kubectl port-forward svc/nginx-nginx-app 8080:80 -n nginx-app
# Acesse: http://localhost:8080
```
