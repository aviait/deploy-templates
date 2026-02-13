# AviaIT Deploy Templates

Templates de GitHub Actions reutilizáveis para padronizar build e deploy de projetos.

## Templates disponíveis

| Template | Descrição |
|:---|:---|
| `deploy-static-template.yml` | Build e deploy de projeto estático para servidor remoto via SSH + rsync |
| `deploy-docker-template.yml` | Build/push de imagem Docker + update de container remoto com rollback |
| `deploy-worker-template.yml` | Build/push de imagem Docker + deploy de worker remoto com rollback |
| `deploy-mobile-template.yml` | Pipeline de build/publicação para apps Android e iOS |

## Observações gerais

- Os templates usam `actions/checkout@v6` e `actions/setup-node@v6`.
- Esses actions exigem runner de Actions `>= 2.327.1`.
- Para `checkout@v6` com comandos git autenticados dentro de Docker container action, use runner `>= 2.329.0`.

## Deploy Static (`deploy-static-template.yml`)

### Exemplo de uso

```yaml
name: Deploy PRD Static

on:
  push:
    branches: [main]

jobs:
  deploy:
    uses: aviait/deploy-templates/.github/workflows/deploy-static-template.yml@main
    with:
      ENVIRONMENT: PRD
      PROJECT_NAME: my-web
      REMOTE_DIR: /var/www/my-web
      REMOTE_OWNER: "__AUTO__"
      SSH_USER: deploy
      SSH_HOST: my-host.example.com
      # SSH_KNOWN_HOSTS: "my-host.example.com ssh-ed25519 AAAA..."
      NODE_VERSION: "20"
      PACKAGE_MANAGER: yarn
      INSTALL_CMD: yarn install --immutable
      BUILD_CMD: yarn build
      BUILD_DIR: build
      CACHE_DEPENDENCY_PATH: yarn.lock
      SSH_REMOTE_PORT: 22
      RSYNC_EXCLUDES: |
        *.map
        node_modules
      UPLOAD_ARTIFACT: true
      ARTIFACT_NAME: my-web-build
    secrets:
      PEM_KEY: ${{ secrets.PEM_KEY }}
```

### Inputs principais

| Input | Tipo | Obrigatório | Default |
|:---|:---|:---:|:---|
| `ENVIRONMENT` | string | sim | - |
| `PROJECT_NAME` | string | sim | - |
| `REMOTE_DIR` | string | sim | - |
| `REMOTE_OWNER` | string | não | `__AUTO__` |
| `SSH_USER` | string | sim | - |
| `SSH_HOST` | string | sim | - |
| `SSH_KNOWN_HOSTS` | string | não | `""` |
| `NODE_VERSION` | string | não | `20` |
| `PACKAGE_MANAGER` | string (`npm`, `yarn`, `pnpm`) | não | `yarn` |
| `INSTALL_CMD` | string | não | `yarn install --immutable` |
| `BUILD_CMD` | string | não | `yarn build` |
| `BUILD_DIR` | string | não | `build` |
| `CACHE_DEPENDENCY_PATH` | string | não | `""` |
| `SSH_REMOTE_PORT` | number | não | `22` |
| `RSYNC_EXCLUDES` | string | não | `""` |
| `UPLOAD_ARTIFACT` | boolean | não | `false` |
| `ARTIFACT_NAME` | string | não | `static-build` |

### Secrets

| Secret | Obrigatório |
|:---|:---:|
| `PEM_KEY` | sim |

## Deploy Docker (`deploy-docker-template.yml`)

### Exemplo de uso

```yaml
name: Deploy PRD Docker

on:
  push:
    branches: [main]

jobs:
  deploy:
    uses: aviait/deploy-templates/.github/workflows/deploy-docker-template.yml@main
    with:
      ENVIRONMENT: PRD
      PROJECT_NAME: my-api
      SSH_USER: deploy
      SSH_HOST: my-host.example.com
      # SSH_KNOWN_HOSTS: "my-host.example.com ssh-ed25519 AAAA..."
      NODE_VERSION: "20"
      VERSION_CMD: node -p "require('./package.json').version"
      PACKAGE_MANAGER: yarn
      INSTALL_CMD: yarn install --immutable
      BUILD_CMD: yarn build
      CACHE_DEPENDENCY_PATH: yarn.lock
      DOCKER_IMAGE: my-api
      CONTAINER_NAME: my-api
      REGISTRY: docker.io
      IMAGE_NAMESPACE: my-org
      DOCKER_PLATFORM: linux/amd64
      DOCKER_BUILD_ARGS: |
        NODE_ENV=production
      DOCKER_RUN_ENVS: "-p 3000:3000 -e NODE_ENV=production"
      # DOCKER_RUN_ENVS_FILE: |-
      #   NODE_ENV=production
      IMAGE_RETENTION_COUNT: 5
      SSH_REMOTE_PORT: 22
      REMOTE_DOCKER_LOGIN: true
      HEALTHCHECK_URL: https://my-api.example.com/health
      HEALTHCHECK_RETRIES: 12
      HEALTHCHECK_DELAY: 5
      PRUNE_RUNNER: true
    secrets:
      PEM_KEY: ${{ secrets.PEM_KEY }}
      DOCKER_USERNAME: ${{ secrets.DOCKER_USERNAME }}
      DOCKER_PASSWORD: ${{ secrets.DOCKER_PASSWORD }}
```

### Outputs

| Output | Descrição |
|:---|:---|
| `version` | Versão lida do `package.json` |
| `image_ref` | Referência completa da imagem publicada |

### Inputs importantes (adicionais)

- `VERSION_CMD`: comando para extrair a versão do build (default: `node -p "require('./package.json').version"`).
- `CONTAINER_NAME`: nome do container remoto (default: usa `DOCKER_IMAGE`).
- `DOCKER_RUN_ENVS` e `DOCKER_RUN_ENVS_FILE`: agora podem ser usados juntos.
- `SSH_KNOWN_HOSTS`: permite pinar host key além do `ssh-keyscan`.
- `REMOTE_DOCKER_LOGIN`: quando `true`, executa `docker login` também no servidor remoto antes do `pull`.

### Secrets

| Secret | Obrigatório |
|:---|:---:|
| `PEM_KEY` | sim |
| `DOCKER_USERNAME` | não |
| `DOCKER_PASSWORD` | não |

## Deploy Worker (`deploy-worker-template.yml`)

Template para publicar e atualizar workers em container remoto. Mantém rollback e retenção de imagens como no template Docker.

### Exemplo de uso

```yaml
name: Deploy Worker PRD

on:
  push:
    branches: [main]

jobs:
  deploy:
    uses: aviait/deploy-templates/.github/workflows/deploy-worker-template.yml@main
    with:
      ENVIRONMENT: PRD
      PROJECT_NAME: my-worker-service
      SSH_USER: deploy
      SSH_HOST: my-host.example.com
      # SSH_KNOWN_HOSTS: "my-host.example.com ssh-ed25519 AAAA..."
      NODE_VERSION: "20"
      VERSION_CMD: node -p "require('./package.json').version"
      PACKAGE_MANAGER: yarn
      INSTALL_CMD: yarn install --immutable
      BUILD_CMD: yarn build
      CACHE_DEPENDENCY_PATH: yarn.lock
      WORKER_NAME: queue-processor
      DOCKER_IMAGE: queue-processor
      REGISTRY: docker.io
      IMAGE_NAMESPACE: my-org
      DOCKER_PLATFORM: linux/amd64
      DOCKER_BUILD_ARGS: |
        NODE_ENV=production
      WORKER_RUN_ENVS: "-e NODE_ENV=production"
      # WORKER_RUN_ENVS_FILE: |-
      #   NODE_ENV=production
      WORKER_EXTRA_ARGS: "node dist/worker.js"
      IMAGE_RETENTION_COUNT: 5
      STARTUP_CHECK_RETRIES: 6
      STARTUP_CHECK_DELAY: 5
      REMOTE_DOCKER_LOGIN: true
      PRUNE_RUNNER: true
    secrets:
      PEM_KEY: ${{ secrets.PEM_KEY }}
      DOCKER_USERNAME: ${{ secrets.DOCKER_USERNAME }}
      DOCKER_PASSWORD: ${{ secrets.DOCKER_PASSWORD }}
```

### Outputs

| Output | Descrição |
|:---|:---|
| `version` | Versão lida do `package.json` |
| `image_ref` | Referência completa da imagem publicada |

### Inputs importantes (adicionais)

- `VERSION_CMD`: comando para extrair a versão do build.
- `WORKER_RUN_ENVS` e `WORKER_RUN_ENVS_FILE`: podem ser combinados.
- `WORKER_EXTRA_ARGS`: argumentos/comando extra no `docker run`.
- `SSH_KNOWN_HOSTS`: host keys adicionais para SSH estrito.
- `REMOTE_DOCKER_LOGIN`: login remoto no registry antes do `docker pull`.

## Deploy Mobile (`deploy-mobile-template.yml`)

Template para pipeline mobile com jobs separados para Android (`ubuntu-latest`) e iOS (`macos-latest`).

### Exemplo de uso

```yaml
name: Deploy Mobile PRD

on:
  push:
    branches: [main]

jobs:
  deploy_mobile:
    uses: aviait/deploy-templates/.github/workflows/deploy-mobile-template.yml@main
    with:
      ENVIRONMENT: PRD
      PROJECT_NAME: my-mobile-app
      WORKING_DIRECTORY: .
      ANDROID_WORKING_DIRECTORY: android
      IOS_WORKING_DIRECTORY: ios
      NODE_VERSION: "20"
      PACKAGE_MANAGER: yarn
      INSTALL_CMD: yarn install --immutable
      CACHE_DEPENDENCY_PATH: yarn.lock
      BUILD_ANDROID: true
      BUILD_IOS: true
      UPLOAD_ARTIFACT: true
      ANDROID_JAVA_VERSION: "17"
      ANDROID_PREPARE_CMD: chmod +x ./gradlew
      ANDROID_BUILD_CMD: ./gradlew :app:bundleRelease
      ANDROID_ARTIFACT_PATH: app/build/outputs/bundle/release/*.aab
      ANDROID_ARTIFACT_NAME: android-release
      # ANDROID_PUBLISH_CMD: bundle exec fastlane android deploy
      RUBY_VERSION: "3.3"
      IOS_USE_BUNDLER_CACHE: false
      IOS_PREPARE_CMD: pod install
      IOS_BUILD_CMD: xcodebuild -workspace Runner.xcworkspace -scheme Runner -configuration Release -archivePath build/Runner.xcarchive archive
      # IOS_EXPORT_CMD: xcodebuild -exportArchive -archivePath build/Runner.xcarchive -exportPath build -exportOptionsPlist ExportOptions.plist
      IOS_ARTIFACT_PATH: build/**/*.ipa
      IOS_ARTIFACT_NAME: ios-release
      # IOS_PUBLISH_CMD: bundle exec fastlane ios deploy
    secrets: inherit
```

### Inputs principais

| Input | Tipo | Obrigatório | Default |
|:---|:---|:---:|:---|
| `ENVIRONMENT` | string | sim | - |
| `PROJECT_NAME` | string | sim | - |
| `WORKING_DIRECTORY` | string | não | `.` |
| `ANDROID_WORKING_DIRECTORY` | string | não | `.` |
| `IOS_WORKING_DIRECTORY` | string | não | `.` |
| `NODE_VERSION` | string | não | `20` |
| `PACKAGE_MANAGER` | string (`npm`, `yarn`, `pnpm`) | não | `yarn` |
| `INSTALL_CMD` | string | não | `yarn install --immutable` |
| `CACHE_DEPENDENCY_PATH` | string | não | `""` |
| `BUILD_ANDROID` | boolean | não | `true` |
| `BUILD_IOS` | boolean | não | `true` |
| `UPLOAD_ARTIFACT` | boolean | não | `true` |
| `ANDROID_JAVA_VERSION` | string | não | `17` |
| `ANDROID_BUILD_CMD` | string | não | `./gradlew :app:bundleRelease` |
| `ANDROID_PREPARE_CMD` | string | não | `""` |
| `ANDROID_ARTIFACT_PATH` | string | não | `android/app/build/outputs/bundle/release/*.aab` |
| `ANDROID_ARTIFACT_NAME` | string | não | `android-release` |
| `ANDROID_PUBLISH_CMD` | string | não | `""` |
| `RUBY_VERSION` | string | não | `3.3` |
| `IOS_USE_BUNDLER_CACHE` | boolean | não | `false` |
| `IOS_PREPARE_CMD` | string | não | `""` |
| `IOS_BUILD_CMD` | string | não | `xcodebuild ... archive` |
| `IOS_EXPORT_CMD` | string | não | `""` |
| `IOS_ARTIFACT_PATH` | string | não | `build/**/*.ipa` |
| `IOS_ARTIFACT_NAME` | string | não | `ios-release` |
| `IOS_PUBLISH_CMD` | string | não | `""` |

## Melhorias aplicadas nos templates

- Padronização para `actions/checkout@v6` e `actions/setup-node@v6`.
- Inputs de build/instalação mais flexíveis (`PACKAGE_MANAGER`, `INSTALL_CMD`, `BUILD_CMD`, `CACHE_DEPENDENCY_PATH`).
- Uso de shell `bash` explícito nos jobs que dependem de recursos específicos de shell.
- `ssh-keyscan` com `-H`, `sort -u` e suporte a host keys pré-definidas (`SSH_KNOWN_HOSTS`).
- Fluxo Docker/Worker com extração de versão configurável (`VERSION_CMD`) e login remoto opcional no registry (`REMOTE_DOCKER_LOGIN`).
- Fluxo Docker com suporte a nome customizado de container (`CONTAINER_NAME`) e combinação de `DOCKER_RUN_ENVS` + `DOCKER_RUN_ENVS_FILE`.
- Novo template completo para worker com rollback/startup checks.
- Template mobile com diretórios por plataforma (`ANDROID_WORKING_DIRECTORY`/`IOS_WORKING_DIRECTORY`), preparo Android opcional e cache de bundler controlável no iOS.
