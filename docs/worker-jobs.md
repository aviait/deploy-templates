# Worker Template (GHCR + Kubernetes)

Escolha técnica: `GHCR + Kubernetes`, igual API, para manter padrão operacional e rollback uniforme.

## Como usar
1. No repositório do worker, configure triggers PR/main/tag.
2. Chame: `uses: aviait/deploy-templates/.github/workflows/template-worker-jobs.yml@main`.
3. Defina `worker_kind`:
   - `deployment` para consumidores de fila.
   - `cronjob` para agendados.

## Inputs mínimos
- `image_name`
- `k8s_namespace_stage|prod`
- `k8s_workload_name`, `k8s_container_name`
- opcionais por ambiente:
  - `queue_config_map_stage|prod`
  - `cron_schedule_stage|prod`

## Segredos
- `KUBE_CONFIG_STAGE`
- `KUBE_CONFIG_PROD`

## Gates
- PR: CI (`quality/build/security`).
- Main: deploy stage automático.
- Tag `vX.Y.Z`: release + deploy prod com aprovação manual.

## Rollback
- `deployment`: rollback automático com `rollout undo` em falha.
- `cronjob`: rollback para imagem anterior se update falhar.

## Garantias específicas
- Evita duplicidade em fila com estratégia `Recreate` (sem overlap de pods).
- Valida `terminationGracePeriodSeconds` e suporta validação de `preStop`.
