# Operator Guide

## Getting Started

### Prerequisites
- Clojure CLI (latest)
- Java 11+
- A fishing vessel or fleet to manage

### Installation

```bash
git clone https://github.com/cloud-itonami/cloud-itonami-isic-0311.git
cd cloud-itonami-isic-0311
clojure -M:dev
```

### Quick Demo

```bash
clojure -M:dev:run
```

This runs a simulation of:
1. Logging a catch record (approved by governor, escalated to human)
2. Attempting navigation (rejected by governor as forbidden authority)
3. Reporting a safety concern (immediately escalated)
4. Requesting supplies (approved, escalated to human for final approval)

## Deployment

### Step 1: Register Your Vessels

Create a file `vessels.edn`:

```clojure
[{:vessel-id "my-vessel-001"
  :name "Ocean Hope"
  :flag-state "JP"
  :capacity-ton 120
  :home-port "Misawa"}]
```

Load into the actor:

```clojure
(import '[fish.store :as store])
(def state (store/empty-store))
(def state' (store/register-vessel state "my-vessel-001" ...))
```

### Step 2: Register Permits and Quotas

For each vessel, register its fishing permits:

```clojure
(store/register-permit state' "permit-001" "my-vessel-001"
  {:jurisdiction "JP"
   :species-allowed [:skipjack-tuna :yellowfin]
   :quota-ton 500
   :valid-until "2027-12-31"})
```

### Step 3: Enable Catch Logging

Train or configure the advisor to propose catch records:

```clojure
(import '[fish.advisor :as advisor])

(let [proposal (advisor/propose-catch-record
               :vessel-id "my-vessel-001"
               :species "skipjack tuna"
               :quantity-kg 150
               :location {...}
               :timestamp (java.time.Instant/now)
               :cites ["jp-fra-2024"])]
  ...)
```

### Step 4: Human Approval Workflow

For every high-stakes proposal (catch records, supply orders):

1. **Governor evaluates** the proposal against safety gates
2. **System notifies** captain/fleet manager via email or SMS
3. **Human reviews** the proposal (usually auto-approved if within baseline)
4. **Captain approves** via web UI or API
5. **Actor logs** the action to audit ledger

Safety concerns skip approval and notify immediately.

## Safety Gates

### Hard Blocks (Never Override)

These proposals are PERMANENTLY REJECTED:

- **Navigation Commands**: "navigate to zone X", "set heading to 090°"
- **Fishing Gear Operation**: "deploy trawl", "haul nets", "winch down"
- **Catch Decisions**: "target yellowfin", "allocate 100 tons to zone A"
- **Engine Control**: "increase throttle to 80%", "fuel the vessel"

If the advisor generates such proposals, it's a sign the LLM has been misconfigured. Reset and retrain.

### Soft Gates (Can Escalate)

These proposals are approved by governor but escalated to human:

- **Low Confidence** (< 60%): e.g., "probably should log 80kg tuna" (confidence: 0.55)
- **High-Stakes**: All catch logging and supply orders require human sign-off
- **Missing Evidence**: Incomplete data for the proposal

## Auditing

Every proposal, approval, escalation, and actuation is logged:

```clojure
(get state' :escalations)
;; [{:vessel-id "my-vessel-001" 
;;    :concern-type "engine-overheating" 
;;    :timestamp "2026-07-14T08:30:00Z" 
;;    :escalated-to "captain@fleet.jp"}]
```

Export the audit ledger for regulatory compliance:

```bash
clojure -M:dev:audit-export > audit-trail-2026-q3.edn
```

## Troubleshooting

### "Governor approved but still escalated?"

High-stakes operations (catch logging, supply ordering) are always escalated to human, even if governor approves. This is by design: the actor proposes, the human decides.

### "Proposal was rejected as forbidden authority?"

Check the error message for the keyword that triggered the block. Examples:
- "navigation-forbidden" → Remove mentions of "navigate", "heading", "course"
- "fishing-gear-forbidden" → Remove "net", "trawl", "deploy"
- "catch-decision-forbidden" → Remove "target", "quota", "allocation"

These are permanent blocks and cannot be overridden.

### "Vessel not verified?"

Ensure the vessel is registered in the state:

```clojure
(store/vessel-verified? state "vessel-id")
;; Should return true
```

Register it if missing:

```clojure
(store/register-vessel state "vessel-id" :verified? true)
```

## Next Steps

- Configure multi-fleet management (see `docs/deployment-patterns.md`)
- Integrate with crew management system (see `docs/crew-integration.md`)
- Set up compliance reporting (see `docs/compliance-reporting.md`)
- Deploy to production on Datomic (see `docs/production-deployment.md`)
