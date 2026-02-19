# Changelog: V2 Alignment + Tracer E2E Rollout

## Data de publicacao
- 2026-02-19

## Escopo
- Alinhamento canonico de callers v2 (`worker`, `bff`, `app`, `web`) por ambiente.
- Correcao de resolucao de variaveis de deploy no `v2-worker.yml` com fallback por sufixo de ambiente (`*_DEV|*_HML|*_PRD`) e fallback final para chave default.
- Padronizacao de tracer ponta a ponta (HTTP, fila, logs), mantendo compatibilidade legada temporaria.

## Ordem de rollout recomendada
1. `deploy-templates` (fonte canonica)
2. `template-*` (geracao futura)
3. `cloudprovision-*` (repos provisionados atuais)

## Compatibilidade legada de tracer
- Ate **2026-05-31**: payloads legados sem `payload.tracer` continuam aceitos.
- A partir de **2026-06-01**: alvo para remocao dos campos legados fora de `tracer`, mantendo apenas contrato canonico:
  - `tracer.transactionId`
  - `tracer.correlationId`
  - `tracer.identifier`
  - `tracer.source` (opcional)

## Contrato canonico de fila
- `messageId = tracer.transactionId`
- `correlationId = tracer.correlationId`
- `headers.tracer_identifier = tracer.identifier`
