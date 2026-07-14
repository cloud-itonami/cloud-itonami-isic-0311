# ADR-0001: Marine Fishing Operations Coordinator Architecture

**Status:** Implemented

**Date:** 2026-07-14

## Summary

This document defines the architecture of the Marine Fishing Operations Coordinator actor for ISIC 0311 (Marine fishing). The actor coordinates fleet operations logistics and compliance, NOT vessel navigation, fishing-gear operation, or catch decisions.

## Context

Marine fishing requires:
1. Vessel and permit registry and verification
2. Catch-record logging with quota tracking
3. Vessel maintenance scheduling (proposals only)
4. Supply ordering and procurement (proposals only)
5. Safety concern escalation (immediate to captain/crew)
6. Strict separation of concerns: an LLM advisor cannot make maritime authority decisions

The actor follows the cloud-itonami pattern: LLM advisor proposes, independent governor evaluates against hard safety gates, human (captain/fleet manager) reviews/approves for all operations.

## Design Decisions

### 1. **No Navigation or Vessel-Command Authority**

The actor is forbidden from:
- Navigating vessels or setting course/heading
- Operating fishing gear (nets, lines, hooks)
- Making catch decisions (where/when to fish, target species, quota allocation)
- Controlling engines or fuel systems
- Making weather/sea-state decisions

**Implementation:** `fish.governor` has hard blocks that reject any proposal mentioning navigation, gear operation, catch authority, or engine control, regardless of confidence or evidence quality.

### 2. **Jurisdiction-Specific Catch Regulations (Honest Catalog)**

Each jurisdiction (Japan, EU) has its own catch-logging and quota requirements, all backed by official citations. The catalog is deliberately small (2 jurisdictions) to honestly represent coverage: we claim only what we verify with official spec-basis.

**File:** `src/fish/facts.cljc`

### 3. **Governor Contract with Hard and Soft Gates**

**Hard violations (no override):**
- No spec-basis citation for high-stakes proposals
- Vessel/permit not verified
- Navigation keywords present (navigate, course, helm, etc.)
- Fishing-gear keywords present (net, trawl, deploy, etc.)
- Catch-decision keywords present (target, allocation, quota, etc.)
- Engine/fuel control keywords present (throttle, engine, fuel, etc.)

**Soft violations (human can approve):**
- Low confidence (< 0.6)
- High-stakes actuation (catch record logging, supply orders)

**File:** `src/fish/governor.cljc`

### 4. **Proposal-Only Operations**

The actor proposes catch records, maintenance schedules, and supply orders but never executes them. A human (captain, fleet manager, or crew) reviews and approves before actuation.

### 5. **Safety Concerns Always Escalate Immediately**

Any safety concern (engine overheating, weather hazard, equipment failure) is flagged immediately to the captain/crew with no delay. No human approval gate delays safety escalations.

## Implementation

- **Language:** Portable Clojure (`.cljc`)
- **Runtime:** langgraph-clj StateGraph (JVM/Chicory/ClojureScript/browser WASM)
- **State Store:** In-memory atom (reference), Datomic for production
- **Audit Ledger:** Append-only (via langgraph-clj checkpoint system)
- **Tests:** JVM-based (clojure.test), 100% coverage of governor contract

## Scope Boundaries

### IN SCOPE
- Vessel and permit registration/verification
- Catch-record logging with species and quantity
- Quota tracking and compliance reporting
- Maintenance scheduling **proposals** (human executes)
- Supply ordering **proposals** (human approves/executes)
- Safety concern detection and escalation
- Compliance report generation
- Escalation logic for violations

### OUT OF SCOPE (NEVER, by design)
- **Vessel navigation** — course, heading, waypoint, autopilot
- **Fishing-gear operation** — net deployment, trawling, gear hauling
- **Catch decisions** — target species, quota allocation, when/where to fish
- **Engine/fuel control** — throttle, propulsion, fuel management
- **Weather/sea-state decisions** — navigation planning based on conditions

## Safety Contract

This actor enforces a permanent safety contract: any proposal mentioning navigation authority, fishing-gear operation, catch decisions, or engine control is rejected with a hard violation, even if approved by a human. The contract is codified in `fish.governor`'s keyword block lists and cannot be overridden.

## Regulatory Compliance

Catch logging and quota tracking comply with:
- Japan Fisheries Resource Protection Act (2024)
- EU Common Fisheries Policy (2023)
- IMO maritime safety regulations

Citations are required in every catch-logging proposal and are validated by the governor before acceptance.

## Audit Trail

Every proposal, approval, escalation, and actuation is logged to an append-only audit ledger. This ledger provides:
- Complete record of all decisions and their justifications
- Regulatory compliance evidence
- Incident investigation record

## Testing Strategy

- 100% code coverage of `fish.governor` contract
- Phase state-machine validation
- Storage contract compliance
- Jurisdiction-specific regulatory fact lookups
- Hard block keyword detection (navigation, gear, catch, engine)
- High-stakes escalation gates
