# Estratégia de Stages (dev/hml/prd)

## Mapeamento recomendado
- `development` -> `dev`
- `homologation` -> `hml`
- `production` -> `prd` (opcional por branch)
- `tag vX.Y.Z` -> `prd` oficial de release

## Toggles padrão
- `PIPELINE_ENABLE_DEV=true`
- `PIPELINE_ENABLE_HML=true`
- `PIPELINE_ENABLE_PRD=true`
- `PIPELINE_ENABLE_PROD_FROM_BRANCH=false`
- `PIPELINE_ENABLE_PROD_FROM_TAG=true`

## Operação recomendada
1. PR aprovado + CI verde em `development`.
2. Merge em `development` deploya `dev`.
3. Promoção para `homologation` com validação funcional.
4. Tag semântica (`vX.Y.Z`) para produção.
5. `environment prd` com aprovação manual obrigatória.

## Modelo recomendado
- 3 callers por app (um por branch/alvo): isolamento forte de credenciais, DNS e janela de deploy por ambiente.
- Todos chamam workflows `v2-*.yml`, com resolução por GitHub Environment (`dev`, `hml`, `prd`).

## Congelamento de ambiente
- Desative sem mudar YAML:
- `PIPELINE_ENABLE_DEV=false` ou `PIPELINE_ENABLE_HML=false` ou `PIPELINE_ENABLE_PRD=false`.
