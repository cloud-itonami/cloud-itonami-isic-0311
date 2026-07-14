# Business Model

## The Problem

Commercial fishing fleets operate under multiple jurisdictional regulations:
- Catch logging and quota compliance (Japan FAA, EU CFP)
- Vessel maintenance requirements
- Supply chain coordination
- Crew safety reporting

Today's fleets use paper logs, email spreadsheets, and verbal handoffs, creating:
- Audit trail gaps
- Delayed safety escalations
- Quota compliance risk
- Supply shortages due to poor coordination

## The Solution

This actor provides an open-source fleet operations coordinator that any qualified fishing operator can fork, deploy, and customize:

1. **Vessel & Permit Registry** — Authoritative registry of vessels, permits, and quota allocations
2. **Catch Logging** — Structured catch records with species, quantity, location, timestamp
3. **Quota Tracking** — Compliance reporting against jurisdiction-specific quotas
4. **Maintenance Scheduling** — Proposals for preventive and reactive maintenance
5. **Supply Ordering** — Automated procurement proposals (fuel, gear, provisions)
6. **Safety Escalation** — Immediate notification of safety concerns to crew/captain

## Revenue Models for Implementers

Fishing fleet operators who deploy this actor can monetize it:

1. **Fleet Management SaaS** — Host the actor on cloud infrastructure, provide a UI, charge per vessel
2. **Compliance Auditing** — Audit fleets' catch logs and quotas, certify compliance to regulators
3. **Supply Chain Integration** — Connect to fuel/gear suppliers, earn commission on orders
4. **Insurance/Risk Management** — Bundle with vessel insurance, use audit ledger for claims
5. **Consulting** — Help fleets adapt the actor to local regulations

## Technical Deployment Paths

### Self-Hosted (fishing fleet)
- Clone repo
- Run on fleet's own infrastructure
- Integrate with existing vessel systems via REST/WebSocket
- Maintain audit ledger locally or on Datomic

### Managed Deployment (service provider)
- Fork repo
- Add multi-tenant database layer (Datomic per fleet, or multi-tenant Datomic)
- Deploy on Kubernetes/cloud
- Offer white-label UI
- Provide compliance certification

### Hybrid (fleet + service provider)
- Vessel systems run the actor locally
- Sync audit ledger to central compliance auditor weekly
- Central auditor provides regulatory reporting and appeals

## Open Source + Business

This repo is AGPL-3.0-or-later, which means:
- Fishing fleets using the actor OWN their data and audit logs
- Service providers who modify the actor must open-source their changes
- Commercial deployments must contribute improvements back to the community

This "copyleft" model ensures the fishing community benefits from improvements while allowing businesses to build on top of it.

## Expected Impact

1. **Compliance**: Automated catch logging reduces regulatory violations and penalties
2. **Safety**: Immediate escalation of safety concerns saves lives at sea
3. **Sustainability**: Quota tracking prevents overfishing and supports marine conservation
4. **Efficiency**: Automated maintenance and supply ordering reduces fleet downtime
5. **Transparency**: Append-only audit ledger provides evidence for claims/disputes
