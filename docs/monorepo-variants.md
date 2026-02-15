# Monorepo Variants (paths-filter + matrix)

Use esse padrão no workflow caller para evitar builds desnecessários.

```yaml
jobs:
  changes:
    runs-on: ubuntu-latest
    outputs:
      frontend: ${{ steps.filter.outputs.frontend }}
      api: ${{ steps.filter.outputs.api }}
    steps:
      - uses: actions/checkout@v6
      - id: filter
        uses: dorny/paths-filter@v3
        with:
          filters: |
            frontend: ['apps/web/**']
            api: ['apps/api/**']

  deploy_api:
    if: needs.changes.outputs.api == 'true'
    uses: aviait/deploy-templates/.github/workflows/template-backend-api.yml@main

  deploy_web:
    if: needs.changes.outputs.frontend == 'true'
    uses: aviait/deploy-templates/.github/workflows/template-frontend-web.yml@main
```

Para múltiplos packages do mesmo tipo, combine com `strategy.matrix` no caller.
