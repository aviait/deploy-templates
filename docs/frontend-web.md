# Frontend Web Template (S3 + CloudFront)

Escolha técnica: `S3 + CloudFront` porque é padrão de mercado para estático, barato, escalável, e permite invalidação de cache por deploy.

## Como usar
1. No repositório da aplicação, crie um workflow caller com gatilhos:
   - `pull_request` para `main` (CI completo).
   - `push` em `main` (deploy stage automático).
   - `push` em tag `v*` (release + deploy prod).
2. Chame: `uses: aviait/deploy-templates/.github/workflows/template-frontend-web.yml@main`.
3. Configure `environment` protection para `production` (aprovação manual).

## Inputs mínimos
- `aws_role_arn_stage`, `aws_role_arn_prod`
- `s3_bucket_stage`, `s3_bucket_prod`
- `smoke_url_stage`, `smoke_url_prod`
- opcionais: `cloudfront_distribution_id_stage|prod`

## Segredos
- Não requer segredo AWS estático; use OIDC com `aws-actions/configure-aws-credentials`.
- `GITHUB_TOKEN` é usado automaticamente para release/scans.

## Gates
- CI: PR para `main`.
- Stage: merge em `main`.
- Prod: tag `vX.Y.Z` + aprovação manual no environment `production`.

## Rollback
- Reexecutar deploy com artifact/tag anterior.
- Para rollback rápido: redeploy da release anterior no bucket + invalidação CloudFront.

## Observações
- Jobs padrão: `quality`, `build`, `security`, `release`, `deploy_stage`, `deploy_prod`.
- Fail-fast ativo em lint/typecheck/tests.
