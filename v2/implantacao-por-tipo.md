# Implantação nos Repositórios Template (com callers)

Este guia define como implantar os templates `v2` nos seus repositórios template consumidores, usando callers por ambiente.

## Padrão adotado
- 3 callers por tipo: `development`, `homologation`, `production`.
- 1 GitHub Environment por estágio: `dev`, `hml`, `prd`.
- Mesmos nomes de `vars/secrets` em todos os environments (mudando só os valores).
- Workflows reutilizáveis usados pelos callers:
  - `v2-web.yml`
  - `v2-bff.yml`
  - `v2-worker.yml`
  - `v2-app.yml`

## Passos comuns (todos os tipos)
1. Copie os 3 callers do tipo desejado de `v2/callers/by-environment/` para o repositório template consumidor.
2. Ajuste `paths` dos gatilhos e `working_directory` para a estrutura real do repo.
3. Crie os environments `dev`, `hml`, `prd` em `Settings > Environments`.
4. Cadastre `vars` e `secrets` por environment.
5. Ative proteção no environment `prd` (aprovação manual obrigatória).
6. Configure branch protection com checks obrigatórios: `quality`, `build`, `security`, `pre-deploy-config`.
7. Faça um `workflow_dispatch` em cada caller para validar configuração inicial.

## Web
Arquivos callers:
- `web-development.yml`
- `web-homologation.yml`
- `web-production.yml`

Ajustes obrigatórios no caller:
- `paths`: incluir apenas arquivos da aplicação web.
- `working_directory`: pasta do frontend (ou `.` para repo único).
- `build_output_dir`: pasta de saída de build (`dist`, `build`, etc).

Vars mínimas por environment:
- `SSH_USER`: usuário SSH do host web.
- `SSH_HOST`: host/IP do servidor.
- `DNS_NAME`: domínio da aplicação (ex.: `backoffice.aviait.com.br`).

Diretório remoto final:
- o deploy publica em `/var/www/<DNS_NAME>`.

Secrets mínimos por environment:
- `PEM_KEY`: chave SSH privada.

Comportamento:
- PR/Push: roda quality + build + security.
- Push em `development` e `homologation`: deploy para `dev`/`hml`.
- Produção: caller de produção permite tag `v*` e respeita `PIPELINE_ENABLE_PROD_FROM_BRANCH`.
- Pós deploy: smoke check via `SMOKE_URL` ou automaticamente `https://<DNS_NAME>` quando `SMOKE_URL` não for informado.

## BFF
Arquivos callers:
- `bff-development.yml`
- `bff-homologation.yml`
- `bff-production.yml`

Ajustes obrigatórios no caller:
- `paths` e `working_directory`.
- `registry`: registry de imagem (`ghcr.io` ou `docker.io`).
- `image_namespace` (opcional): namespace quando `image_name` nao tiver `/` (Docker Hub).
- `image_name`: caminho do repositorio de imagem sem o host do registry.
- `dockerfile`: caminho do Dockerfile.

Vars mínimas por environment:
- `SSH_USER`
- `SSH_HOST`
- `HOST_PORT`: porta publicada no host (obrigatória para deploy).

Secrets mínimos por environment:
- `PEM_KEY`
- `DEPLOY_REGISTRY_USERNAME`
- `DEPLOY_REGISTRY_PASSWORD`

Vars recomendadas:
- `CONTAINER_PORT` (default 3000).
- `CONTAINER_NAME` (se não definir, usa `app_name`/repo).
- `HEALTHCHECK_URL`, `HEALTHCHECK_RETRIES`, `HEALTHCHECK_DELAY_SECONDS`.
- `BFF_INJECT_ALL_GITHUB_VARS`, `BFF_GITHUB_VARS_PREFIX`, `BFF_GITHUB_VARS_EXCLUDE`.

Secrets opcionais:
- `DOCKER_RUN_ENVS_FILE` para enviar env-file multiline.

Comportamento:
- Builda imagem Docker, publica no registry configurado e faz deploy remoto via SSH + `docker run`.
- Em tags `vX.Y.Z`: a tag principal da imagem de release usa a versao do `package.json` (validada contra a tag git) quando disponivel, e tambem publica a tag `vX.Y.Z` e `latest`.
- Em falha de startup/healthcheck, executa rollback automático para imagem anterior.
- Gera resumo completo em `GITHUB_STEP_SUMMARY`.

## Worker
Arquivos callers:
- `worker-development.yml`
- `worker-homologation.yml`
- `worker-production.yml`

Ajustes obrigatórios no caller:
- `paths`, `working_directory`, `dockerfile`, `registry`, `image_name`.
- `image_namespace` (opcional): namespace quando `image_name` nao tiver `/` (Docker Hub).
- Nome de worker opcional (`WORKER_NAME`); fallback automático para nome do repo.

Vars mínimas por environment:
- `SSH_USER`
- `SSH_HOST`

Secrets mínimos por environment:
- `PEM_KEY`
- `DEPLOY_REGISTRY_USERNAME`
- `DEPLOY_REGISTRY_PASSWORD`

Vars recomendadas:
- `WORKER_RUN_ENVS` e/ou `WORKER_RUN_ENVS_FILE`.
- `WORKER_EXTRA_ARGS`.
- `STARTUP_CHECK_RETRIES`, `STARTUP_CHECK_DELAY_SECONDS`.
- `WORKER_INJECT_ALL_GITHUB_VARS`, `WORKER_GITHUB_VARS_PREFIX`, `WORKER_GITHUB_VARS_EXCLUDE`.

Comportamento:
- Deploy remoto via SSH com `docker stop/rm` e `docker run` do novo worker.
- Em tags `vX.Y.Z`: a tag principal da imagem de release usa a versao do `package.json` (validada contra a tag git) quando disponivel, e tambem publica a tag `vX.Y.Z` e `latest`.
- Evita duplicidade permanente mantendo um único container com o mesmo nome.
- Em falha de subida, rollback automático.

## App (React Native)
Arquivos callers:
- `app-development.yml`
- `app-homologation.yml`
- `app-production.yml`

Ajustes obrigatórios no caller:
- `paths`, `working_directory`.
- `APP_BUILD_ANDROID` e `APP_BUILD_IOS` conforme plataforma alvo.
- Comandos de build (`APP_ANDROID_BUILD_CMD`, `APP_IOS_BUILD_CMD`, `APP_IOS_EXPORT_CMD`) conforme projeto.

Vars mínimas por environment:
- `FIREBASE_GROUPS` para distribuição QA.
- `DEPLOY_VIA_FASTLANE` (normalmente `false` em dev/hml e `true` em prd).

Secrets mínimos por environment (quando build/distribuição habilitados):
- `FIREBASE_TOKEN`
- Android signing: `ANDROID_KEYSTORE_BASE64`, `ANDROID_KEYSTORE_PASSWORD`, `ANDROID_KEY_ALIAS`, `ANDROID_KEY_PASSWORD`
- iOS signing: `IOS_P12_BASE64`, `IOS_P12_PASSWORD`, `IOS_MOBILEPROVISION_BASE64`, `IOS_KEYCHAIN_PASSWORD`

Secrets para release em lojas (produção):
- `APP_STORE_CONNECT_KEY_ID`
- `APP_STORE_CONNECT_ISSUER_ID`
- `APP_STORE_CONNECT_PRIVATE_KEY_BASE64`
- `GOOGLE_PLAY_SERVICE_ACCOUNT_JSON`

Comportamento:
- CI completo (lint/typecheck/tests).
- Build Android/iOS com artifacts.
- Distribuição via Firebase App Distribution quando configurado.
- Release via Fastlane em produção (quando habilitado), com gates de tag/environment.

## Estratégia recomendada de promoção
1. Merge em `development` para validar `dev`.
2. Promoção para `homologation` após validação funcional.
3. Release de `production` por tag semântica (`vX.Y.Z`) e aprovação manual no environment `prd`.

## Observações importantes
- `app_name` é opcional em todos os templates; fallback automático para nome do repositório.
- Em BFF/Worker, prefira injeção de `vars` por environment para reduzir mapeamentos fixos no caller.
- Nunca cadastre segredo em `vars`; use sempre `secrets`.
