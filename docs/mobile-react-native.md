# Mobile React Native Template (Firebase QA + Fastlane Prod)

Escolha técnica:
- QA: `Firebase App Distribution` (Android+iOS no mesmo fluxo, simples para squads QA).
- Prod: `Fastlane` (padrão para automação de stores com versionamento por tag).

## Como usar
1. No app mobile, configure triggers PR/main/tag.
2. Chame: `uses: aviait/deploy-templates/.github/workflows/template-mobile-react-native.yml@main`.
3. Ative environment protection para `production`.

## Inputs mínimos
- `build_android`, `build_ios`
- comandos de build (`android_build_cmd`, `ios_build_cmd`, `ios_export_cmd`)
- `firebase_app_id_android|ios`
- lanes fastlane (`fastlane_android_lane`, `fastlane_ios_lane`)

## Segredos
- QA: `FIREBASE_TOKEN`
- Android signing: `ANDROID_KEYSTORE_BASE64`, `ANDROID_KEYSTORE_PASSWORD`, `ANDROID_KEY_ALIAS`, `ANDROID_KEY_PASSWORD`
- iOS signing: `IOS_P12_BASE64`, `IOS_P12_PASSWORD`, `IOS_MOBILEPROVISION_BASE64`, `IOS_KEYCHAIN_PASSWORD`
- Store release: `APP_STORE_CONNECT_*`, `GOOGLE_PLAY_SERVICE_ACCOUNT_JSON`

## Gates
- PR: quality/security/build.
- Main: distribuição QA (Firebase).
- Tag `vX.Y.Z`: release + publicação prod (Fastlane) com aprovação manual.

## Rollback
- Mobile não tem rollback instantâneo universal; estratégia: interromper rollout na loja e republicar versão anterior.
- Artefatos de release são anexados ao GitHub Release para reenvio rápido.
