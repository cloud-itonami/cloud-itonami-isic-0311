# Governance

`cloud-itonami-isic-0311` is an OSS open-business blueprint. Governance
covers both code and the operator model.

## Maintainers

Maintainers may merge changes that preserve these invariants:

- The advisor (FishingOps-LLM) cannot directly navigate a vessel, operate
  fishing gear, or make a catch decision.
- `MarineFishingOperationsGovernor` remains independent of the advisor.
- Hard governor violations (unverified vessel/permit, non-`:propose`
  effect, blocked navigation/command/catch-decision op, quota exceedance)
  cannot be overridden by human approval.
- Navigation, gear-operation, and catch-decision ops are never added to
  the closed op allowlist (`fishing.governor/op->effect`).
- Every commit, hold, and escalation is auditable.
- Real vessel, permit, crew, or catch data stays outside Git.

## Decision Records

Architecture decisions live in `docs/adr/`. Changes to the trust model,
storage contract, actuation invariant, public business model, or license
should add or update an ADR.
