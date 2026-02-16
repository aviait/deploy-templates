# Pipeline Specs by Application Type

## 1) Frontend (Web)
- Pipeline file: `.github/workflows/template-frontend-web.yml`
- Docs: `docs/frontend-web.md`
- Vars/inputs: `aws_role_arn_stage|prod`, `s3_bucket_stage|prod`, `smoke_url_stage|prod`, `cloudfront_distribution_id_*` (opcional)
- Secrets: sem credencial AWS estática (OIDC); `GITHUB_TOKEN` automático
- Gates: PR->CI, `main`->stage, tag `vX.Y.Z`->prod (manual approval via `production` env)
- Versionamento: tags semânticas `vMAJOR.MINOR.PATCH`
- Checklist:
  - [x] quality/build/security/release/deploy
  - [x] artifact de build
  - [x] CDN invalidation
  - [x] smoke check

## 2) Backend (API)
- Pipeline file: `.github/workflows/template-backend-api.yml`
- Docs: `docs/backend-api.md`
- Vars/inputs: `image_name`, `k8s_namespace_stage|prod`, `k8s_deployment_name`, `k8s_container_name`, `migration_cmd`, `healthcheck_url_*`
- Secrets: `KUBE_CONFIG_STAGE`, `KUBE_CONFIG_PROD`, `GIT_TOKEN` (opcional; fallback `github.token`)
- Gates: PR->CI, `main`->stage, tag `vX.Y.Z`->release+prod (manual approval)
- Versionamento: imagem `sha-<commit>` para stage, `vX.Y.Z` + `latest` para release/prod
- Checklist:
  - [x] quality/build/security/release/deploy
  - [x] migrations seguras por Job
  - [x] publish em GHCR
  - [x] healthcheck pós-deploy
  - [x] rollback automático (`rollout undo`)

## 3) Worker (Jobs/Filas/Cron)
- Pipeline file: `.github/workflows/template-worker-jobs.yml`
- Docs: `docs/worker-jobs.md`
- Vars/inputs: `worker_kind`, `k8s_workload_name`, `k8s_container_name`, `queue_config_map_*`, `cron_schedule_*`, `graceful_shutdown_seconds`
- Secrets: `KUBE_CONFIG_STAGE`, `KUBE_CONFIG_PROD`
- Gates: PR->CI, `main`->stage, tag `vX.Y.Z`->release+prod (manual approval)
- Versionamento: `sha-<commit>` (stage), `vX.Y.Z` + `latest` (prod)
- Checklist:
  - [x] quality/build/security/release/deploy
  - [x] rollout sem duplicidade (`Recreate`/cron concurrency policy)
  - [x] validações de graceful shutdown
  - [x] config por ambiente (queue/schedule)
  - [x] rollback automático

## 4) Mobile (React Native)
- Pipeline file: `.github/workflows/template-mobile-react-native.yml`
- Docs: `docs/mobile-react-native.md`
- Vars/inputs: comandos build Android/iOS, `firebase_app_id_android|ios`, lanes Fastlane
- Secrets: `FIREBASE_TOKEN`, signing Android/iOS, `APP_STORE_CONNECT_*`, `GOOGLE_PLAY_SERVICE_ACCOUNT_JSON`
- Gates: PR->CI, `main`->QA distribution, tag `vX.Y.Z`->prod release (manual approval)
- Versionamento: tags semânticas `vX.Y.Z` para release e publicação em store
- Checklist:
  - [x] quality/build/security/release/deploy
  - [x] build Android/iOS
  - [x] distribuição QA (Firebase App Distribution)
  - [x] release prod com Fastlane
  - [x] artefatos de release anexados
