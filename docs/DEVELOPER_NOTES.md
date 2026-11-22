# Managed Services VP Agent — Portfolio POC

A Microsoft Copilot Studio agent that automates the account-leadership work of a VP of Managed Services for one customer account. Handles onboarding, SOW pricing assistance, delivery-manager task delegation, and QBR prep — with human-in-the-loop approval for anything customer-facing or financially binding.

Vendor-neutral: works for any IT services / MSP business. The product catalog is mocked, so you can demo it without a real ERP or quoting system.

## What's in this kit

| File | What it is |
|---|---|
| `01-setup-guide.md` | Step-by-step environment setup (free dev tenants, connectors, Dataverse) |
| `02-agent-instructions.md` | The system prompt you paste into the agent's "Instructions" field |
| `03-topics/*.md` | The 4 topic specs: onboarding, SOW pricing, manager delegation, QBR prep |
| `04-adaptive-cards/*.json` | Three approval cards (pricing, delegation, customer email) |
| `05-catalog-mock/` | Power Automate flow that fakes a vendor product-catalog API |
| `06-sample-data/acme-corp-customer.json` | Synthetic customer to demo against |

## Recommended build order

1. **Read `01-setup-guide.md`** end-to-end before signing up for anything.
2. Spin up the four dev tenants (allow ~2 hours including waiting on email verifications).
3. Stand up the catalog mock (`05-catalog-mock/README.md`) — 30 min.
4. Create the agent shell in Copilot Studio, paste in `02-agent-instructions.md`.
5. Build the 4 topics one at a time using `03-topics/*.md` as the blueprint.
6. Paste the adaptive-card JSON into the Power Automate "Post adaptive card" actions.
7. Load `06-sample-data/acme-corp-customer.json` into Salesforce + ServiceNow.
8. Record a 4-minute demo: one prompt per topic, showing the approval cards firing.

## Demo script suggestions

- **Onboarding:** "Onboard Acme Corp as a new managed services customer, primary contact Jordan Lee, kickoff next Tuesday."
- **SOW pricing:** "Model a 3-year SOW for Acme covering a server refresh and 24×7 platinum support, target 35% margin."
- **Delegation:** "Assign the Acme storage migration to a delivery manager — needs storage-array experience."
- **QBR:** "Generate Q2 QBR deck for Acme."

## Hard guardrails

- Never sends external email without a Teams approval card.
- Never commits SOW pricing — drafts only.
- Never assigns a *named* person — proposes assignment, manager confirms.
- Every action logged to a Dataverse `AgentActionLog` table for the portfolio walkthrough.
