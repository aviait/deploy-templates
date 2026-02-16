# Variaveis e Segredos (v2)

Este documento detalha o que cada variavel/segredo controla nos templates `v2-web.yml`, `v2-bff.yml`, `v2-worker.yml` e `v2-app.yml`.

## Como os valores sao resolvidos
1. Valor enviado no `with:` do caller.
2. `vars` do GitHub Environment (`dev`, `hml`, `prd`).
3. Default do workflow (quando existir).

## Onde cadastrar
- `vars`: parametros de configuracao (host, portas, comandos, toggles).
- `secrets`: dados sensiveis (chaves SSH, senha de registry, certificados mobile).

## Segredos comuns (cross-template)
| Segredo | Obrigatorio | Onde usa | O que faz |
| --- | --- | --- | --- |
| `CHECKOUT_TOKEN` | nao | `v2-web.yml`, `v2-bff.yml`, `v2-worker.yml`, `v2-app.yml` | Token alternativo para `actions/checkout` quando o `GITHUB_TOKEN` nao consegue clonar o repo (ex.: orchestrator repo rodando pipeline para outro repo privado, submodules privados, restricoes de permissao). Recomendado: Fine-grained PAT com acesso apenas ao(s) repo(s) necessario(s) e permissao `Contents: Read`. |

Nota (auto-tag):
- O exemplo `v2/callers/examples/auto-tag-production.yml` usa `secrets.GITHUB_TOKEN` (token automatico do GitHub Actions) para criar/push de tag.
- Se voce precisa que o push da tag dispare outros workflows (`on: push tags`), considere usar PAT dedicado no lugar do `GITHUB_TOKEN` por causa da protecao anti-recursao do GitHub.

## Variaveis comuns (usadas pelos callers)
| Variavel | Tipo | Default | O que faz |
| --- | --- | --- | --- |
| `PIPELINE_ENV_DEV` | string | `dev` | Nome do GitHub Environment usado no caller de development. |
| `PIPELINE_ENV_HML` | string | `hml` | Nome do GitHub Environment usado no caller de homologation. |
| `PIPELINE_ENV_PRD` | string | `prd` | Nome do GitHub Environment usado no caller de production. |
| `PIPELINE_SEMVER_PREFIX` | string | `v` | Prefixo aceito para tags de release (`vX.Y.Z`). |
| `PIPELINE_ENABLE_PROD_FROM_BRANCH` | boolean string | `false` | No caller de producao, permite deploy em push de branch `production` sem tag (`true`) ou apenas por tag (`false`). |

## Variaveis legadas (nao usadas no modelo atual by-environment)
- `PIPELINE_BRANCH_DEV`, `PIPELINE_BRANCH_HML`, `PIPELINE_BRANCH_PRD`
- `PIPELINE_ENABLE_DEV`, `PIPELINE_ENABLE_HML`, `PIPELINE_ENABLE_PRD`
- `PIPELINE_ENABLE_PROD_FROM_TAG`

## Web
Workflow: `v2-web.yml`

### Variaveis de caller (CI/build)
| Variavel | Default | O que faz |
| --- | --- | --- |
| `WEB_NODE_VERSION` | `20` | Versao do Node usada em quality/build/security. |
| `WEB_PACKAGE_MANAGER` | `pnpm` (nos callers) | Define `npm`, `yarn` ou `pnpm`. |
| `WEB_PNPM_VERSION` | `9` | Versao do pnpm quando `WEB_PACKAGE_MANAGER=pnpm`. |
| `WEB_CACHE_DEPENDENCY_PATH` | `apps/web/pnpm-lock.yaml` | Lockfile usado para cache de dependencias. |
| `WEB_LINT_CMD` | `npm run lint` | Comando de lint (fail-fast). |
| `WEB_TYPECHECK_CMD` | `npm run typecheck` | Comando de typecheck (fail-fast). |
| `WEB_TEST_CMD` | `npm test -- --ci` | Comando de testes (fail-fast). |
| `WEB_BUILD_CMD` | `npm run build` | Comando de build de producao. |
| `WEB_BUILD_OUTPUT_DIR` | `dist` | Pasta do output que vira artifact e deploy. |
| `WEB_ARTIFACT_NAME` | `web-build` | Nome do artifact de build publicado no Actions. |
| `WEB_DNS` | vazio | DNS principal da aplicação web (ex.: `backoffice.aviait.com.br`) usado para resolver `/var/www/<DNS_NAME>` e smoke URL padrão. |

### Variaveis de environment (deploy)
| Variavel | Obrigatoria | Default | O que faz |
| --- | --- | --- | --- |
| `SSH_USER` | sim (deploy) | - | Usuario SSH do servidor de deploy. |
| `SSH_HOST` | sim (deploy) | - | Host/IP do servidor de deploy. |
| `SSH_REMOTE_PORT` | nao | `22` | Porta SSH remota. |
| `SSH_KNOWN_HOSTS` | condicional | - | Conteudo de known_hosts para pinning de host key. |
| `REQUIRE_SSH_KNOWN_HOSTS` | nao | `false` | Quando `true`, exige `SSH_KNOWN_HOSTS` e nao usa `ssh-keyscan` automatico. |
| `DNS_NAME` | sim | vazio | DNS principal do ambiente (ex.: `backoffice.aviait.com.br`); o deploy publica sempre em `/var/www/<DNS_NAME>`. |
| `REMOTE_OWNER` | nao | `__AUTO__` | Dono aplicado no diretorio remoto (`user:group`); `__AUTO__` tenta usar o usuario remoto. |
| `RSYNC_EXCLUDES` | nao | vazio | Exclusoes adicionais no `rsync` (lista separada por linha). |
| `SMOKE_URL` | nao | `https://<DNS_NAME>` quando DNS configurado | URL para smoke check apos deploy. |
| `SMOKE_RETRIES` | nao | `12` | Tentativas de smoke check. |
| `SMOKE_DELAY_SECONDS` | nao | `5` | Delay entre tentativas de smoke check. |

### Segredos de environment
| Segredo | Obrigatorio | O que faz |
| --- | --- | --- |
| `PEM_KEY` | sim (deploy) | Chave privada SSH usada para conectar no host remoto. |

## BFF
Workflow: `v2-bff.yml`

### Variaveis de caller (CI/build/publicacao)
| Variavel | Default | O que faz |
| --- | --- | --- |
| `BFF_NODE_VERSION` | `20` | Versao do Node na pipeline. |
| `BFF_PACKAGE_MANAGER` | `pnpm` (nos callers) | Gerenciador de pacotes. |
| `BFF_PNPM_VERSION` | `9` | Versao do pnpm. |
| `BFF_CACHE_DEPENDENCY_PATH` | `apps/bff/pnpm-lock.yaml` | Lockfile para cache. |
| `BFF_LINT_CMD` | `npm run lint` | Lint de codigo. |
| `BFF_TYPECHECK_CMD` | `npm run typecheck` | Typecheck da aplicacao. |
| `BFF_TEST_CMD` | `npm test -- --ci` | Testes automatizados. |
| `BFF_DOCKERFILE` | `Dockerfile` | Dockerfile usado no build da imagem. |
| `BFF_REGISTRY` | `ghcr.io` | Registry de imagem (ex.: `ghcr.io` ou `docker.io`). |
| `BFF_IMAGE_NAMESPACE` | vazio | Namespace quando `BFF_IMAGE_NAME` nao tiver `/` (ex.: usuario/org no Docker Hub). |
| `BFF_IMAGE_NAME` | `{owner}/{repo}/bff` | Caminho do repositorio da imagem sem o host do registry (nao inclua `:tag`/`@digest`). Exemplos: GHCR: `{owner}/{repo}/bff`; Docker Hub: `namespace/repo` (nao suporta `namespace/repo/sub`). |
| `BFF_INJECT_ALL_GITHUB_VARS` | `true` | Injeta automaticamente as `vars` do GitHub no env-file do container (`docker run`). Quando `environment` eh reconhecido como `dev/hml/prd`, chaves no formato `*_DEV`/`*_HML`/`*_PRD` sao normalizadas para a chave base (ex.: `S3_BUCKET_DEV` vira `S3_BUCKET`). |
| `BFF_GITHUB_VARS_PREFIX` | vazio | (LEGADO) Mantido por compatibilidade, mas nao eh usado nos templates v2 atuais. |
| `BFF_GITHUB_VARS_EXCLUDE` | vazio | Exclui vars da injecao automatica (`KEY` ou `PREFIX_*`). |

Notas de release (tags):
- Em tags `vX.Y.Z`, quando existe `package.json` em `working_directory`, o workflow valida que a versao do `package.json` corresponde a tag (ex.: `v1.2.3` e `package.json=1.2.3`) e usa `1.2.3` como tag principal da imagem de release (tambem publica `v1.2.3` e `latest`).

### Variaveis de environment (deploy/runtime)
| Variavel | Obrigatoria | Default | O que faz |
| --- | --- | --- | --- |
| `SSH_USER` | sim (deploy) | - | Usuario SSH para deploy remoto. |
| `SSH_HOST` | sim (deploy) | - | Host remoto de deploy. |
| `SSH_REMOTE_PORT` | nao | `22` | Porta SSH remota. |
| `SSH_KNOWN_HOSTS` | condicional | - | Host key fixa para conexao segura. |
| `REQUIRE_SSH_KNOWN_HOSTS` | nao | `false` | Exige pinning explicito de host key. |
| `CONTAINER_NAME` | nao | `app_name` | Nome do container `docker run`. |
| `CONTAINER_PORT` | nao | `80` (fixo) | Porta interna do container para BFF. No template v2, o deploy sempre mapeia `HOST_PORT` para `80` (`-p HOST_PORT:80`). |
| `HOST_PORT` | sim (deploy) | sem default util (`0`) | Porta publica no host (`-p HOST_PORT:80`). |
| `DOCKER_RUN_ENVS` | nao | vazio | Variaveis inline para `docker run` (formato `KEY=VALUE`, multiline). |
| `DOCKER_EXTRA_ARGS` | nao | vazio | Argumentos extras no `docker run` (volumes, network, labels etc.). |
| `MIGRATION_CMD` | nao | vazio | Comando executado no host para migration antes/apos swap (conforme script). |
| `HEALTHCHECK_URL` | nao | vazio | URL para verificar saude apos deploy. |
| `HEALTHCHECK_RETRIES` | nao | `12` | Tentativas do healthcheck HTTP. |
| `HEALTHCHECK_DELAY_SECONDS` | nao | `5` | Delay entre tentativas de healthcheck. |
| `REMOTE_DOCKER_LOGIN` | nao | `true` | Faz login no registry no host remoto antes de `docker pull`. |
| `IMAGE_RETENTION_COUNT` | nao | `5` | Quantidade de imagens antigas preservadas no host remoto. |
| `STARTUP_CHECK_RETRIES` | nao | `6` | Tentativas para validar que o container subiu. |
| `STARTUP_CHECK_DELAY_SECONDS` | nao | `5` | Delay entre tentativas de startup check. |
| `PRUNE_RUNNER` | nao | `true` | Faz limpeza de docker no runner ao final. |
| `GITHUB_VARS_PREFIX` | nao | vazio | (LEGADO) Mantido por compatibilidade, mas nao eh usado nos templates v2 atuais. |
| `GITHUB_VARS_EXCLUDE` | nao | vazio | Mesmo papel de exclusao (caso nao use caller com `BFF_*`). |

Nota de resolucao por ambiente (BFF):
- Para as variaveis de deploy/runtime acima, o template prioriza automaticamente `*_DEV`, `*_HML` ou `*_PRD` conforme o `inputs.environment` (`dev|hml|prd`), e cai para a chave base sem sufixo quando o sufixado nao existir.

### Segredos de environment
| Segredo | Obrigatorio | O que faz |
| --- | --- | --- |
| `PEM_KEY` | sim (deploy) | Chave SSH para conectar no servidor. |
| `DEPLOY_REGISTRY_USERNAME` | condicional | Usuario do registry (usado para `docker login` no runner e/ou no host remoto quando `REMOTE_DOCKER_LOGIN=true`). |
| `DEPLOY_REGISTRY_PASSWORD` | condicional | Senha/token do registry (Docker Hub: access token; GHCR: PAT com `read:packages`/`write:packages` conforme necessidade). |
| `DOCKER_RUN_ENVS_FILE` | opcional | Env-file adicional (multiline) enviado para o host e aplicado no container. |

## Worker
Workflow: `v2-worker.yml`

### Variaveis de caller (CI/build/publicacao)
| Variavel | Default | O que faz |
| --- | --- | --- |
| `WORKER_NODE_VERSION` | `20` | Versao do Node na pipeline. |
| `WORKER_PACKAGE_MANAGER` | `pnpm` (nos callers) | Gerenciador de pacotes. |
| `WORKER_PNPM_VERSION` | `9` | Versao do pnpm. |
| `WORKER_CACHE_DEPENDENCY_PATH` | `apps/worker/pnpm-lock.yaml` | Lockfile para cache. |
| `WORKER_LINT_CMD` | `npm run lint` | Lint de codigo. |
| `WORKER_TYPECHECK_CMD` | `npm run typecheck` | Typecheck da aplicacao. |
| `WORKER_TEST_CMD` | `npm test -- --ci` | Testes automatizados. |
| `WORKER_DOCKERFILE` | `Dockerfile` | Dockerfile do worker. |
| `WORKER_REGISTRY` | `ghcr.io` | Registry de imagem (ex.: `ghcr.io` ou `docker.io`). |
| `WORKER_IMAGE_NAMESPACE` | vazio | Namespace quando `WORKER_IMAGE_NAME` nao tiver `/` (ex.: usuario/org no Docker Hub). |
| `WORKER_IMAGE_NAME` | `{owner}/{repo}/worker` | Caminho do repositorio da imagem sem o host do registry (nao inclua `:tag`/`@digest`). Exemplos: GHCR: `{owner}/{repo}/worker`; Docker Hub: `namespace/repo`. |
| `WORKER_INJECT_ALL_GITHUB_VARS` | `true` | Injeta automaticamente `vars` no env-file do worker. |
| `WORKER_GITHUB_VARS_PREFIX` | vazio | (LEGADO) Mantido por compatibilidade, mas nao eh usado nos templates v2 atuais. |
| `WORKER_GITHUB_VARS_EXCLUDE` | vazio | Exclui chaves da injecao automatica. |

Notas de release (tags):
- Em tags `vX.Y.Z`, quando existe `package.json` em `working_directory`, o workflow valida que a versao do `package.json` corresponde a tag (ex.: `v1.2.3` e `package.json=1.2.3`) e usa `1.2.3` como tag principal da imagem de release (tambem publica `v1.2.3` e `latest`).

### Variaveis de environment (deploy/runtime)
| Variavel | Obrigatoria | Default | O que faz |
| --- | --- | --- | --- |
| `SSH_USER` | sim (deploy) | - | Usuario SSH do host remoto. |
| `SSH_HOST` | sim (deploy) | - | Host remoto onde o worker roda. |
| `SSH_REMOTE_PORT` | nao | `22` | Porta SSH remota. |
| `SSH_KNOWN_HOSTS` | condicional | - | Host key fixa para SSH. |
| `REQUIRE_SSH_KNOWN_HOSTS` | nao | `false` | Exige known_hosts definido manualmente. |
| `WORKER_NAME` | nao | `app_name` | Nome do container do worker. |
| `WORKER_RUN_ENVS` | nao | vazio | Variaveis inline para `docker run` do worker. |
| `WORKER_EXTRA_ARGS` | nao | vazio | Argumentos extras de `docker run` para worker. |
| `REMOTE_DOCKER_LOGIN` | nao | `true` | Login no registry no host remoto antes de pull. |
| `IMAGE_RETENTION_COUNT` | nao | `5` | Numero de imagens antigas retidas no host. |
| `STARTUP_CHECK_RETRIES` | nao | `6` | Tentativas para confirmar worker rodando. |
| `STARTUP_CHECK_DELAY_SECONDS` | nao | `5` | Delay entre tentativas de startup check. |
| `PRUNE_RUNNER` | nao | `true` | Limpeza de docker no runner ao final. |
| `GITHUB_VARS_PREFIX` | nao | vazio | (LEGADO) Mantido por compatibilidade, mas nao eh usado nos templates v2 atuais. |
| `GITHUB_VARS_EXCLUDE` | nao | vazio | Exclusao de chaves da injecao de vars (uso direto do workflow). |

### Segredos de environment
| Segredo | Obrigatorio | O que faz |
| --- | --- | --- |
| `PEM_KEY` | sim (deploy) | Chave SSH para deploy remoto. |
| `DEPLOY_REGISTRY_USERNAME` | condicional | Usuario do registry (usado para `docker login` no runner e/ou no host remoto quando `REMOTE_DOCKER_LOGIN=true`). |
| `DEPLOY_REGISTRY_PASSWORD` | condicional | Senha/token do registry (Docker Hub: access token; GHCR: PAT com `read:packages`/`write:packages` conforme necessidade). |
| `WORKER_RUN_ENVS_FILE` | opcional | Env-file multiline adicional aplicado no worker. |

## App (React Native)
Workflow: `v2-app.yml`

### Variaveis de caller (CI/build/distribuicao)
| Variavel | Default | O que faz |
| --- | --- | --- |
| `APP_NODE_VERSION` | `20` | Versao do Node na pipeline mobile. |
| `APP_PACKAGE_MANAGER` | `pnpm` (nos callers) | Gerenciador de pacotes do projeto mobile. |
| `APP_PNPM_VERSION` | `9` | Versao do pnpm. |
| `APP_CACHE_DEPENDENCY_PATH` | `apps/mobile/pnpm-lock.yaml` | Lockfile para cache de dependencias. |
| `APP_LINT_CMD` | `npm run lint` | Lint de codigo. |
| `APP_TYPECHECK_CMD` | `npm run typecheck` | Typecheck. |
| `APP_TEST_CMD` | `npm test -- --ci` | Testes automatizados. |
| `APP_BUILD_ANDROID` | `true` | Ativa build Android. |
| `APP_BUILD_IOS` | `true` | Ativa build iOS. |
| `APP_ANDROID_DIR` | `android` | Diretorio de build Android. |
| `APP_IOS_DIR` | `ios` | Diretorio de build iOS. |
| `APP_ANDROID_BUILD_CMD` | `./gradlew :app:bundleRelease` | Comando de build Android. |
| `APP_IOS_BUILD_CMD` | `xcodebuild ... archive` | Comando de archive iOS. |
| `APP_IOS_EXPORT_CMD` | `xcodebuild ... -exportArchive` | Comando de export do IPA. |
| `APP_FASTLANE_DIR` | `.` | Diretorio onde roda Fastlane. |
| `APP_BUNDLE_INSTALL_CMD` | `bundle install` | Comando de install do Bundler/Fastlane. |
| `APP_FASTLANE_ANDROID_LANE` | `android release` | Lane Fastlane Android. |
| `APP_FASTLANE_IOS_LANE` | `ios release` | Lane Fastlane iOS. |
| `FIREBASE_GROUPS` | `dev/hml/prd` (caller) | Grupos do Firebase App Distribution (CSV). |
| `DEPLOY_VIA_FASTLANE` | `false` (dev/hml) | Ativa release em lojas via Fastlane (tipicamente `true` em prd). |

### Variaveis de environment (deploy/runtime)
| Variavel | Obrigatoria | Default | O que faz |
| --- | --- | --- | --- |
| `FIREBASE_APP_ID_ANDROID` | condicional | vazio | App ID Android no Firebase (necessario para distribuir Android via Firebase). |
| `FIREBASE_APP_ID_IOS` | condicional | vazio | App ID iOS no Firebase (necessario para distribuir iOS via Firebase). |
| `FIREBASE_GROUPS` | nao | `qa` | Grupos de distribuicao no Firebase. |
| `DEPLOY_VIA_FASTLANE` | nao | `false` | Quando `true`, executa lanes Fastlane para publicacao. |
| `FASTLANE_WORKING_DIRECTORY` | nao | `.` | Diretorio de execucao do Fastlane. |
| `BUNDLE_INSTALL_CMD` | nao | `bundle install` | Comando de install Ruby gems antes do Fastlane. |
| `FASTLANE_ANDROID_LANE` | nao | `android release` | Lane Android para release via Fastlane. |
| `FASTLANE_IOS_LANE` | nao | `ios release` | Lane iOS para release via Fastlane. |

### Segredos de environment
| Segredo | Obrigatorio | O que faz |
| --- | --- | --- |
| `FIREBASE_TOKEN` | condicional | Token de autenticacao no Firebase CLI/action. Necessario quando houver distribuicao Firebase. |
| `ANDROID_KEYSTORE_BASE64` | condicional | Keystore Android em Base64 para assinatura. |
| `ANDROID_KEYSTORE_PASSWORD` | condicional | Senha do keystore Android. |
| `ANDROID_KEY_ALIAS` | condicional | Alias da chave Android. |
| `ANDROID_KEY_PASSWORD` | condicional | Senha da chave Android. |
| `IOS_P12_BASE64` | condicional | Certificado iOS (`.p12`) em Base64. |
| `IOS_P12_PASSWORD` | condicional | Senha do certificado `.p12`. |
| `IOS_MOBILEPROVISION_BASE64` | condicional | Provisioning profile iOS em Base64. |
| `IOS_KEYCHAIN_PASSWORD` | condicional | Senha da keychain temporaria usada no build iOS. |
| `APP_STORE_CONNECT_KEY_ID` | condicional | Identificador da API key do App Store Connect. |
| `APP_STORE_CONNECT_ISSUER_ID` | condicional | Issuer ID da API key da Apple. |
| `APP_STORE_CONNECT_PRIVATE_KEY_BASE64` | condicional | Chave privada da API Apple em Base64. |
| `GOOGLE_PLAY_SERVICE_ACCOUNT_JSON` | condicional | JSON da service account para publicacao no Google Play. |

## app_name
- `app_name` e opcional em todos os workflows `v2-*.yml`.
- Quando omitido, o template usa o nome do repositorio (`github.event.repository.name`).
- Esse valor e usado para naming padrao (ex.: nome de container/worker quando nao definido explicitamente).
