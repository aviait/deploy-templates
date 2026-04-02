# Contributing

Thanks for contributing to this template.

## Prerequisites

- Node.js `>= 22`
- Yarn `>= 1.22`
- Local services required by `.env.example` (PostgreSQL, Redis, RabbitMQ when needed)

## Local setup

```bash
yarn install
cp .env.example .env
yarn dev
```

## Branch and commit conventions

- Branch naming (recommended): `feat/...`, `fix/...`, `chore/...`, `refactor/...`, `docs/...`
- Commit style (recommended): Conventional Commits
  - `feat: ...`
  - `fix: ...`
  - `chore: ...`
  - `refactor: ...`
  - `docs: ...`

## Pull request requirements

Before opening a PR, run:

```bash
yarn lint
yarn typecheck
yarn build
```

PR expectations:

- Keep PR scope focused.
- Explain non-obvious technical decisions.
- Update docs when behavior changes.
- Update `.env.example` for new/changed environment variables.
- Never include secrets or production credentials.

## Architecture guidelines

- Follow existing layering (Controller -> UseCase -> Repository).
- Keep business logic out of controllers.
- Prefer explicit DTOs for request/response contracts.
- Keep side effects isolated in providers/services.

## Dependency and security hygiene

- Use Dependabot updates as input, not blind auto-merge.
- Validate dependency upgrades with lint/typecheck/build.
- Report vulnerabilities privately (see `SECURITY.md`).
