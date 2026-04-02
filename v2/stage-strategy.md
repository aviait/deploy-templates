# Estrategia de Stages (dev/hml/prd)

## Mapeamento recomendado
- `development` -> `dev`
- `homologation` -> `hml`
- `production` -> `prd` (opcional por branch)
- `tag vX.Y.Z` -> `prd` oficial de release

## Como funciona no v2

No modelo v2, cada ambiente possui um **caller dedicado** (workflow YAML no repositorio consumidor) que referencia o reusable workflow correspondente (`v2-bff.yml`, `v2-web.yml`, `v2-worker.yml`, `v2-app.yml`).

O controle de qual ambiente e quando deploya e feito exclusivamente pelo **caller**:

- `on: push` com filtro de branches/tags determina quando o caller dispara.
- `inputs.environment` informa ao reusable workflow qual GitHub Environment usar para resolver vars/secrets.
- `PIPELINE_ENABLE_PROD_FROM_BRANCH` (`true`/`false`) controla se o caller de producao aceita push de branch alem de tag.
- `PIPELINE_SEMVER_PREFIX` (default `v`) define o prefixo de tags aceito para release.
- `PIPELINE_ENV_DEV`, `PIPELINE_ENV_HML`, `PIPELINE_ENV_PRD` permitem renomear os GitHub Environments.

## Variaveis legadas (DEPRECATED - nao usadas no v2)

As variaveis abaixo existiam em versoes anteriores e **nao tem efeito nos workflows v2**:

- ~~`PIPELINE_ENABLE_DEV`~~ - nao existe nos callers v2
- ~~`PIPELINE_ENABLE_HML`~~ - nao existe nos callers v2
- ~~`PIPELINE_ENABLE_PRD`~~ - nao existe nos callers v2
- ~~`PIPELINE_ENABLE_PROD_FROM_TAG`~~ - substituido pelo modelo de callers dedicados

Para desabilitar um ambiente no v2, desative ou remova o caller correspondente no repositorio consumidor.

## Operacao recomendada
1. PR aprovado + CI verde em `development`.
2. Merge em `development` deploya `dev`.
3. Promocao para `homologation` com validacao funcional.
4. Tag semantica (`vX.Y.Z`) para producao.
5. `environment prd` com aprovacao manual obrigatoria.

## Modelo recomendado
- 3 callers por app (um por branch/alvo): isolamento forte de credenciais, DNS e janela de deploy por ambiente.
- Todos chamam workflows `v2-*.yml`, com resolucao por GitHub Environment (`dev`, `hml`, `prd`).

## Congelamento de ambiente
- Para congelar um ambiente sem mudar YAML do reusable workflow:
  - Desative o caller do ambiente no repositorio consumidor, ou
  - Adicione uma `if: false` no caller correspondente.
