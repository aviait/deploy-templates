# Backend API Template (GHCR + Kubernetes)

Escolha técnica: `GHCR` (integração nativa com GitHub e permissões de pacote) + `Kubernetes` (rollout/undo nativo, padrão enterprise).

## Como usar
1. No repositório da API, crie workflow caller com:
   - `pull_request` para `main`.
   - `push` em `main`.
   - `push` em tag `v*`.
2. Chame: `uses: aviait/deploy-templates/.github/workflows/template-backend-api.yml@main`.
3. Configure approvals no environment `production`.

## Inputs mínimos
- `image_name` (ex.: `org/api`)
- `k8s_namespace_stage`, `k8s_namespace_prod`
- `k8s_deployment_name`, `k8s_container_name`
- `healthcheck_url_stage`, `healthcheck_url_prod`
- `migration_cmd` (backward-compatible)

## Segredos
- `KUBE_CONFIG_STAGE`
- `KUBE_CONFIG_PROD`
- `GIT_TOKEN` (opcional; se ausente, usa `github.token`)

## Gates
- CI completo em PR.
- Deploy stage automático em `main`.
- Release + deploy prod apenas em tag `vX.Y.Z` com aprovação manual.

## Rollback
- Falha de rollout executa `kubectl rollout undo` automaticamente.
- Imagem release fica versionada por tag semântica para reverter facilmente.

## Observações
- Migrations rodam em Job dedicado antes do deploy.
- Jobs padrão: `quality`, `build`, `security`, `release`, `migrate_*`, `deploy_*`.
