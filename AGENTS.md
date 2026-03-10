# Deploy Templates Engineering Guide

## Mission

`deploy-templates` is not a side repository. It is the runtime contract between provisioning, templates and consuming applications. Changes here must be treated as architecture changes.

## Mandatory Standards

- Workflow contracts must stay explicit and documented.
- Variable/secret names must have a single official naming policy.
- Environment behavior must be predictable by caller and stage strategy.
- Changes that can break consumers must be documented as breaking changes.

## Contract Rules

- Reusable workflows and callers must stay in sync with documented variables/secrets.
- Drift between docs, callers and workflows is not acceptable.
- Validation scripts are part of the contract, not optional tooling.

## Compatibility Rules

- Any change must be reviewed against:
  - `cloudprovision`
  - backend templates
  - web/app templates where applicable
- Breaking contract changes must be called out explicitly and versioned with discipline.

## Testing Expectations

- Guardrails and validation scripts must remain authoritative.
- Caller/workflow compatibility should be validated before release.

## Transfer Rules from CloudProvision

- Provisioning decisions may influence deploy contracts, but `deploy-templates` must not inherit internal platform assumptions that are not stable enough for shared consumption.
