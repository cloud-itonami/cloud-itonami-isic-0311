# cloud-itonami-isic-0311

Open Business Blueprint for **ISIC Rev.5 0311: Marine fishing** — a fishing-fleet operations coordination actor.

## Overview

This actor supports a commercial fishing fleet's back-office coordination workflow:

- **Catch-record logging** — species, quantity, quota-tracking data (administrative, not a fishing decision)
- **Vessel maintenance scheduling** — propose engine/gear maintenance
- **Safety concern flagging** — surface a vessel-safety or weather concern (always escalates to captain/crew)
- **Supply ordering** — fuel/gear/provisions procurement proposals

## What this actor does NOT do

This is a safety-critical maritime domain. This actor is **coordination only**. It never:

- **Navigates the vessel** — no course, heading, waypoint, or autopilot commands
- **Operates fishing gear** — no net deployment, trawling, or gear hauling
- **Makes catch decisions** — no target-species, quota-allocation, or where/when-to-fish decisions
- **Commands the vessel** in any way (engine, throttle, propulsion)

Vessel navigation, gear operation, and catch decisions remain the **vessel captain's exclusive human authority at sea**, always. The governor structurally and permanently blocks any proposal touching these — there is no human-approval path that can override this block.

## Governance Model

All proposals carry `:effect :propose` — this actor is **coordination only**. Every op is drawn from a closed allowlist (`fishing.governor/op->effect`); the advisor cannot legitimately propose anything outside it.

Hard blocks (no human override):
1. **Vessel/permit verification** — vessel must be registered and `:status :active` before any action
2. **Effect integrity** — proposal effect must match the request operation
3. **Coordination-only effects** — all effects are `:propose`
4. **Forbidden fields** — vessel-id, permit-number, vessel-status cannot be touched
5. **Blocked operations** — navigation / vessel-command / catch-decision proposals rejected outright
6. **Quota exceedance** — a catch record that would push cumulative landed weight past the permit quota is a hard, permanent block (a regulatory compliance breach, not a judgment call)

Escalation triggers (human sign-off required):
- **Safety concerns** always escalate immediately
- **High-cost supply orders** (>= 10,000 units) escalate
- **Low advisor confidence** (< 0.6) escalates

## Architecture

```
Request → Advisor (LLM) → Proposal
            ↓
         Governor (independent censor)
            ↓
         Decide (hold | escalate | commit)
            ↓
         Store (persistent audit ledger)
```

All modules are portable `.cljc` (ClojureScript-first, compatible with JVM/nbb).

## Building & Testing

```bash
# Install (workspace already has langgraph-clj at ../../../kotoba-lang/langgraph)
kbb -M:test

# Run demo
kbb -M:dev:run

# Lint
kbb -M:lint
```

## Files

- `blueprint.edn` — open business blueprint metadata
- `src/fishing/` — core actor modules
  - `governor.cljc` — compliance rules and hard blocks
  - `store.cljc` — persistent store protocol + in-memory impl
  - `llm_advisor.cljc` — advisor interface (mock or real LLM)
  - `operation.cljc` — langgraph-clj state machine
  - `sim.cljc` — demo simulation
- `test/fishing/` — contract tests for governance and store

## License

AGPL-3.0-or-later

## Contributing

This actor is part of the cloud-itonami open business blueprint fleet. See CONTRIBUTING.md.
