# PR Validate e Garantia de Código

## Objetivo
Todo PR para `development`, `homologation` e `production` deve executar CI completa antes de merge.

## Trigger recomendado
- `pull_request` em `development|homologation|production`
- `push` em `development|homologation|production`
- `push` com `tags: v*` para release de produção

## Gates mínimos (required checks)
Configure branch protection exigindo os jobs:
- `quality`
- `build`
- `security`
- `pre-deploy-config` (recomendado como obrigatório)
- `summary` (opcional, recomendado para visão consolidada)

## O que cada gate valida
- `quality`: install reproduzível + lint + typecheck + tests
- `build`: build determinístico + artifact/imagem
- `security`: audit de dependências + gitleaks + CodeQL
- `pre-deploy-config`: valida configuração mínima de deploy no PR (vars/secrets críticos e regras de ambiente), sem executar deploy
- `summary`: consolidado dos resultados, caminho percorrido da pipeline e enforcement de gates

## Garantias operacionais
- Fail-fast em qualidade e segurança
- Deploy por ambiente (`dev/hml/prd`) só quando branch/tag correspondente
- Produção por tag semântica + aprovação manual de environment
- Rollback automático em `bff/worker` quando startup/healthcheck falha
- Resumo publicado no `GITHUB_STEP_SUMMARY`
