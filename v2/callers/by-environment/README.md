# Callers por Ambiente (3 por app)

## Quando usar
Use `by-environment` quando houver isolamento forte entre `development`, `homologation` e `production`:
- times diferentes por ambiente
- DNS/host e portas diferentes
- segredos totalmente segregados por ambiente
- janelas de deploy distintas

## Arquivos
- `web-development.yml`, `web-homologation.yml`, `web-production.yml`
- `bff-development.yml`, `bff-homologation.yml`, `bff-production.yml`
- `worker-development.yml`, `worker-homologation.yml`, `worker-production.yml`
- `app-development.yml`, `app-homologation.yml`, `app-production.yml`

## Workflows genéricos por environment
- Web: `.github/workflows/v2-web.yml`
- BFF: `.github/workflows/v2-bff.yml`
- Worker: `.github/workflows/v2-worker.yml`
- App: `.github/workflows/v2-app.yml`

## Convenção de vars/secrets por environment (sem sufixo)
Em cada environment (`dev`, `hml`, `prd`), cadastre os mesmos nomes:
- Web: `SSH_USER`, `SSH_HOST`, `SSH_REMOTE_PORT`, `DNS_NAME`, `PEM_KEY`
- BFF: `SSH_USER`, `SSH_HOST`, `HOST_PORT`, `PEM_KEY`, `DEPLOY_REGISTRY_USERNAME`, `DEPLOY_REGISTRY_PASSWORD`
- Worker: `SSH_USER`, `SSH_HOST`, `PEM_KEY`, `DEPLOY_REGISTRY_USERNAME`, `DEPLOY_REGISTRY_PASSWORD`
- App: `FIREBASE_TOKEN` e credenciais mobile (`ANDROID_*`, `IOS_*`, `APP_STORE_CONNECT_*`, `GOOGLE_PLAY_SERVICE_ACCOUNT_JSON`)

## app_name automático
Nos workflows genéricos, `app_name` é opcional.
Se não for enviado, o nome da aplicação é resolvido automaticamente pelo nome do repositório.

## Injeção automática de vars no container
- BFF/Worker: as `vars` do GitHub podem ser injetadas automaticamente no `docker run` (env-file), mesmo sem mapeamento explícito no caller.

## Observação
Os arquivos são exemplos para copiar e ajustar `paths`, nomes de app e variáveis conforme o monorepo.

## Guia de implantação completo
Para implantar isso em repositórios template consumidores com passo a passo por tipo, consulte:
- `v2/implantacao-por-tipo.md`
