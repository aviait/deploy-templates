# AviaIT Deploy Templates

[![Tests/CI](https://github.com/aviait/deploy-templates/actions/workflows/v2-web.yml/badge.svg)](https://github.com/aviait/deploy-templates/actions/workflows/v2-web.yml)
[![Node](https://img.shields.io/badge/node_consumers-%3E%3D22.0.0-339933?logo=node.js&logoColor=white)](#versoes-e-compatibilidade)
[![Yarn](https://img.shields.io/badge/yarn_consumers-1.22.x-2C8EBB?logo=yarn&logoColor=white)](#versoes-e-compatibilidade)
[![Parametros](https://img.shields.io/badge/parametros-documentados-1f6feb)](#parametros-principais)

Templates reutilizáveis de CI/CD para GitHub Actions, com foco em `development/homologation/production` e ambientes `dev/hml/prd`.

## Padrao de Projeto (Apresentacao e Qualidade)

### Snapshot de checks

| Categoria                        | Onde validar                                         | Resultado esperado             |
| -------------------------------- | ---------------------------------------------------- | ------------------------------ |
| Sintaxe de workflows             | PRs com `.github/workflows/v2-caller-guardrails.yml` | YAML valido e sem regressao    |
| Padrao de callers v2             | `scripts/validate-v2-callers.mjs` (CI)               | sem divergencias de contrato   |
| Qualidade nos repos consumidores | `yarn run check` em cada app que usa os templates    | pipeline verde antes de deploy |

### Versoes e compatibilidade

- GitHub Actions runner: `ubuntu-latest`
- Node recomendado nos repos consumidores: `>= 22.0.0`
- Yarn recomendado nos repos consumidores: `1.22.x` (Classic)

### Padroes obrigatorios

1. Toda alteracao em workflow deve manter os callers `development/homologation/production`.
2. Nenhum segredo hardcoded em workflows.
3. Validar guardrails antes de liberar nova revisao dos templates.

## Parametros principais

Fontes oficiais de parametros:

- `v2/variables-secrets.md`
- `v2/stage-strategy.md`
- `v2/callers/by-environment/*.yml`

Regra de manutencao:

1. Toda variavel nova precisa estar em `v2/variables-secrets.md`.
2. Toda mudanca de estrategia por ambiente precisa refletir `v2/stage-strategy.md`.
3. O README principal deve apontar para essas fontes.

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

- `web`: build estático + deploy SSH/rsync (diretório remoto derivável por DNS: `/var/www/<DNS_NAME>`)
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
