# Managed Services Account-Leadership Agent (Copilot Studio)

> A Microsoft Copilot Studio agent blueprint that automates the routine account-leadership work of a managed-services executive: customer onboarding, SOW pricing drafts, delivery-manager delegation, and QBR preparation. Human-in-the-loop approval gates every customer-facing or financially binding action.

![Copilot Studio](https://img.shields.io/badge/Microsoft-Copilot%20Studio-0078D4) ![Power Automate](https://img.shields.io/badge/Power%20Automate-flows-0066FF) ![Dataverse](https://img.shields.io/badge/Dataverse-audit%20log-742774) ![Vendor neutral](https://img.shields.io/badge/vendor-neutral-lightgrey)

## Why this exists

Service-delivery leaders spend a large share of their week assembling QBR decks, re-pricing SOWs, and routing work to the right delivery manager. This POC shows how an enterprise agent can take on that load **without** giving up control: it drafts, proposes and summarizes, and a person approves.

## What's in the kit

| Path | Purpose |
|---|---|
| `01-setup-guide.md` | Free dev-tenant setup (M365, Power Platform, Salesforce, ServiceNow) and connectors |
| `02-agent-instructions.md` | Production-style system prompt with guardrails |
| `03-topics/` | Four topic specs: onboarding, SOW pricing, manager delegation, QBR prep |
| `04-adaptive-cards/` | Teams approval cards: pricing, delegation, customer email |
| `05-catalog-mock/` | Power Automate flow that mocks a vendor product-catalog API |
| `06-sample-data/` | Fully synthetic "Acme Corp" customer record |

## Architecture

```
Teams / M365 Copilot ──▶ Copilot Studio agent ──▶ Topics (4)
                                │                    │
                                ▼                    ▼
                 Power Automate flows ──▶ Salesforce (account) · ServiceNow (delivery) · Catalog mock
                                │
                    Adaptive Card approval in Teams  ◀── human decision
                                │
                       Dataverse AgentActionLog (full audit trail)
```

## Governance guardrails (by design)

- Never sends external email without a Teams approval card.
- Never commits SOW pricing; it produces drafts only, with a target-margin model.
- Never assigns a *named* person; it proposes, and the manager confirms.
- Every action is logged to Dataverse for audit and review.

## Demo script

1. "Onboard Acme Corp as a new managed services customer, primary contact Jordan Lee, kickoff next Tuesday."
2. "Model a 3-year SOW for Acme covering a server refresh and 24×7 platinum support, target 35% margin."
3. "Assign the Acme storage migration to a delivery manager with storage-array experience."
4. "Generate the Q2 QBR deck for Acme."

## Value

Demonstrates **responsible AI adoption** in a leadership workflow: real automation, clear approval boundaries, and auditability. This is the pattern enterprises need before they let agents near customers or contracts.

## Roadmap

- Record and embed a 4-minute demo video or GIF
- Export as a Power Platform solution package for one-click import
- Add KPI tiles (time saved per QBR, approval turnaround)

## Topics
`copilot-studio` `power-automate` `ai-agents` `human-in-the-loop` `ai-governance` `managed-services` `servicenow` `salesforce` `microsoft-365`
