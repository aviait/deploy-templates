# CI/CD Templates v2 (estáveis)

Este repositório publica reusable workflows para:
- `web` (static via SSH/rsync)
- `bff` (docker via SSH)
- `worker` (docker via SSH)
- `app` (React Native)

## Estrutura
- Workflows genéricos por environment:
  - `.github/workflows/v2-web.yml`
  - `.github/workflows/v2-bff.yml`
  - `.github/workflows/v2-worker.yml`
  - `.github/workflows/v2-app.yml`
- `v2/callers/by-environment/*.yml`: 3 callers por ambiente (`development|homologation|production`).
- `v2/variables-secrets.md`: catálogo de `vars`/`secrets`.
- `v2/implantacao-por-tipo.md`: implantação prática por tipo (`web`, `bff`, `worker`, `app`).
- `v2/stage-strategy.md`: estratégia de promoção e toggles.
- `v2/pr-validate.md`: padrão de validação de PR.

## Fluxo padrão
1. `pull_request` para `development|homologation|production`: roda `quality`, `build`, `security`.
2. `push` em `development`: deploy `dev`.
3. `push` em `homologation`: deploy `hml`.
4. `tag vX.Y.Z`: `release` + deploy `prd`.
5. `production` branch só deploya `prd` quando habilitado.

## Padrão de deploy
- Web: deploy para diretório remoto com `rsync --delete` + smoke check; diretório pode ser derivado de DNS (`/var/www/<DNS_NAME>`).
- BFF: `docker pull`, `docker stop/rm`, `docker run`, healthcheck opcional e rollback automático.
- Worker: mesmo padrão do BFF, com startup checks e rollback automático.
- App: distribuição Firebase e/ou release via Fastlane.

## app_name automático
Nos workflows `v2-*.yml`, `app_name` é opcional.
Se omitido, o template usa automaticamente o nome do repositório.

## Injeção automática de vars
- BFF/Worker: `vars` do GitHub podem ser injetadas automaticamente no container (env-file), sem mapeamento explícito no caller.

## Gates e segurança
- Fail-fast: lint/typecheck/tests/audit/pre-deploy-config quebram o pipeline.
- Segurança: `npm|yarn|pnpm audit`, `gitleaks`, `CodeQL`.
- PR validate: `pre-deploy-config` valida variáveis/segredos mínimos de deploy antes do merge.
- Produção: usar `environment protection` com aprovação manual no environment `prd`.
- Cada pipeline publica resumo em `GITHUB_STEP_SUMMARY`, incluindo tabela de resultados e caminho de execução da pipeline.
