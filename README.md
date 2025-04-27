# 🚀 AviaIT Deploy Templates

Templates de GitHub Actions para automação de deploys de projetos:

- 📦 Projetos **estáticos** (build gerando arquivos tipo React, Vue, HTML)
- 🐳 Projetos **Docker** (build, push e atualização de containers)

Organizados para serem reutilizados entre múltiplos projetos e ambientes, com flexibilidade total.

# 📚 Templates disponíveis

| Template | Descrição |
|:---|:---|
| `deploy-static-template.yml` | Deploy de projetos estáticos para servidor remoto |
| `deploy-docker-template.yml` | Deploy de aplicações Docker para servidor remoto |

# 📦 Deploy Static Project (`deploy-static-template.yml`)

## 🚀 Como utilizar

Exemplo `.github/workflows/deploy-prd.yml` no projeto:

```yaml
name: Deploy PRD Static

on:
  push:
    branches:
      - main

jobs:
  deploy:
    uses: aviait/deploy-templates/.github/workflows/deploy-static-template.yml@main
    with:
      ENVIRONMENT: "PRD"
      PROJECT_NAME: "imoflex-webapp"
      REMOTE_DIR: "/var/www/imoflex.aviait.com.br"
      SSH_USER: "deploy"
      SSH_HOST: "dns.aviait.com.br"
      NODE_VERSION: "18.16.1"
    secrets:
      PEM_KEY: ${{ secrets.PEM_KEY }}
```

## 📥 Inputs esperados

| Nome | Tipo | Obrigatório | Descrição |
|:---|:---|:---|:---|
| `ENVIRONMENT` | string | ✅ | Ambiente lógico (DEV, HML, PRD) |
| `PROJECT_NAME` | string | ✅ | Nome do projeto |
| `REMOTE_DIR` | string | ✅ | Pasta de destino no servidor |
| `SSH_USER` | string | ✅ | Usuário SSH para login |
| `SSH_HOST` | string | ✅ | Host/DNS do servidor |
| `NODE_VERSION` | string | ❌ | Versão do Node (default: 18.16.1) |

## 🔒 Secrets esperados

| Nome | Descrição |
|:---|:---|
| `PEM_KEY` | Chave privada para conexão SSH |

## 📋 Fluxo de execução

1. Checkout do repositório
2. Instala dependências (`yarn install`)
3. Roda build (`yarn build`)
4. Cria pasta remota (caso necessário)
5. Envia arquivos da pasta `build/` usando `rsync`
6. Apaga chave SSH após deploy

# 🐳 Deploy Docker Project (`deploy-docker-template.yml`)

## 🚀 Como utilizar

Exemplo `.github/workflows/deploy-prd.yml`:

```yaml
name: Deploy PRD Docker

on:
  push:
    branches:
      - main

jobs:
  deploy:
    uses: aviait/deploy-templates/.github/workflows/deploy-docker-template.yml@main
    with:
      ENVIRONMENT: "PRD"
      PROJECT_NAME: "imoflex-bff"
      SSH_USER: "deploy"
      SSH_HOST: "dns.aviait.com.br"
      DOCKER_IMAGE: "imoflex-bff"
      DOCKER_USERNAME: "tiaviait"
      DOCKER_PASSWORD: "token-ou-senha-dockerhub"
      DOCKER_RUN_ENVS: "-p 3008:80 -e NODE_ENV=production -e ENVIRONMENT=production -e APP_API_URL=https://api.imoflex.aviait.com.br ..."
    secrets:
      PEM_KEY: ${{ secrets.PEM_KEY }}
```

## 📥 Inputs esperados

| Nome | Tipo | Obrigatório | Descrição |
|:---|:---|:---|:---|
| `ENVIRONMENT` | string | ✅ | Ambiente lógico (DEV, HML, PRD) |
| `PROJECT_NAME` | string | ✅ | Nome do projeto |
| `SSH_USER` | string | ✅ | Usuário SSH |
| `SSH_HOST` | string | ✅ | Host SSH |
| `DOCKER_IMAGE` | string | ✅ | Nome da imagem Docker |
| `DOCKER_USERNAME` | string | ✅ | Usuário Docker Hub |
| `DOCKER_PASSWORD` | string | ✅ | Token ou senha Docker Hub |
| `DOCKER_RUN_ENVS` | string | ✅ | Variáveis de ambiente para o container (todas na mesma linha) |
| `SSH_REMOTE_PORT` | number | ❌ | Porta SSH (default: 22) |

## 🔒 Secrets esperados

| Nome | Descrição |
|:---|:---|
| `PEM_KEY` | Chave privada para conexão SSH |

## 📋 Fluxo de execução

1. Checkout do repositório
2. Instala dependências (`yarn install`)
3. Roda build (`yarn build`)
4. Extrai versão do `package.json`
5. Faz login no Docker Hub
6. Builda e push da imagem Docker
7. Acessa servidor remoto via SSH
8. Executa `docker pull`, `docker stop`, `docker rm` e `docker run` novo container
9. Apaga chave SSH após deploy

# 🔥 Observações importantes

- O campo `DOCKER_RUN_ENVS` deve ser passado como uma única linha.
- As imagens Docker são tagueadas com a versão do `package.json`.
- A chave SSH (`deploy-key.pem`) é removida após o deploy.
- Variáveis sensíveis nunca ficam persistidas.

# 📈 Melhorias futuras

- Suporte automático a múltiplos ambientes (DEV, HML, PRD)
- Armazenamento de versão buildada como artefato
- Deploy para Kubernetes
- Deploy em múltiplos servidores simultaneamente

> Mantido com 💙 por [AviaIT](https://github.com/aviait)
