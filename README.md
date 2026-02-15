# AviaIT Deploy Templates

Templates reutilizáveis de CI/CD para GitHub Actions, com foco em `development/homologation/production` e ambientes `dev/hml/prd`.

## Versão estável (recomendada)
- Reusable workflows executáveis:
  - `.github/workflows/v2-web.yml`
  - `.github/workflows/v2-bff.yml`
  - `.github/workflows/v2-worker.yml`
  - `.github/workflows/v2-app.yml`
- Pacote pronto para cópia e uso: `v2/`
  - `v2/callers/by-environment/*.yml`
  - `v2/README.md`
  - `v2/stage-strategy.md`
  - `v2/variables-secrets.md`
  - `v2/pr-validate.md`

## Tipos suportados
- `web`: build estático + deploy SSH/rsync (default `/var/www/dominio`)
- `bff`: build Docker + publish imagem + deploy SSH com `docker run`
- `worker`: build Docker + publish imagem + deploy SSH com `docker run`
- `app`: React Native (CI + distribuição QA + release)

## Convenções
- Fail-fast em `lint`, `typecheck`, `tests`
- Build reproduzível via lockfile (`npm ci` / frozen lockfile)
- Segurança com dependency audit + gitleaks + CodeQL
- CI separado de CD, com gates por branch/tag/environment
- Sem segredos em plain text
- `app_name` opcional (fallback automático para nome do repositório)

## Branches de referência
- `development` -> `dev`
- `homologation` -> `hml`
- `production` -> `prd` (opcional por branch)
- `vX.Y.Z` -> release + deploy `prd`

## Próximo passo
Copie um caller de `v2/callers/by-environment` para o repositório da aplicação e ajuste `vars`/`secrets` conforme `v2/variables-secrets.md`.
