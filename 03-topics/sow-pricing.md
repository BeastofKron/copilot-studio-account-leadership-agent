# Topic: SOW Pricing Assistant

## Trigger phrases

- model a SOW for {customer}
- price a statement of work
- what would {customer} pay for...
- draft pricing scenarios for {customer}

## Inputs to collect

| Variable | Type | Prompt if missing |
|---|---|---|
| `CustomerName` | Text | "Which customer is this SOW for?" |
| `TermYears` | Number | "How many years? (1, 3, or 5)" |
| `ServiceLines` | Multi-select | "Which services? (Servers, Storage, Support 24×7 Platinum, Professional services, Networking)" |
| `TargetMargin` | Number (%) | "What's the target margin? (Standard is 30%; deal desk allows 25–40%)" |

## Flow logic

```
1. Pull Salesforce opportunity for CustomerName (most recent open opp).
   - Capture: OpportunityId, Stage, Amount, CloseDate, OwnerId.
   - If none open: ask user "No open opportunity for {CustomerName}.
     Should I draft pricing standalone, or do you want to create the opp first?"

2. Call catalog mock (HTTP action, URL from environment variable
   CATALOG_MOCK_URL). POST body:
      { "serviceLines": <ServiceLines>, "termYears": <TermYears>,
        "customerTier": <derived from SF Account.AnnualRevenue> }
   Response shape: see 05-catalog-mock/mock-catalog.json — returns list of
   line items with internal cost, list price, and recommended discount band.

3. Compute three scenarios in the agent's "Generative answers" step or via
   Power Fx in Copilot Studio. For each scenario:

      Conservative (low risk):
        - Apply minimum discount band
        - Customer price = list × (1 - minDiscount)
        - Margin = (customer price - internal cost) / customer price
        - Target: customer price floor that keeps margin >= TargetMargin + 3pp

      Target (recommended):
        - Apply mid discount band
        - Target margin exactly TargetMargin

      Aggressive (win the deal):
        - Apply max discount band
        - Target margin = TargetMargin - 5pp, floor at 22%

   Each scenario returns:
        - Total contract value (TCV)
        - Annual recurring revenue (ARR)
        - Gross margin %
        - Per-line breakdown
        - Key assumptions used

4. Log Pending action to Dataverse AgentActionLog:
        ActionType = Pricing
        Payload = <JSON of all 3 scenarios + SF OpportunityId>

5. Post adaptive card (see 04-adaptive-cards/pricing-approval.json) to
   the deal desk Teams channel "#deal-desk". The card shows:
     - Customer + opportunity link
     - The 3 scenarios side by side
     - Approve / Reject / Request changes buttons
     - Free-text comment box

6. On Approve: update Dataverse row to Status = Approved, set ApprovedBy,
   and reply in the account channel: "Pricing approved by {ApprovedBy}.
   Drafting SOW document from Approved scenario..." [generates a Word doc
   from SharePoint /sites/templates/sow-template.docx with values merged]

7. On Reject: update Dataverse row to Status = Rejected.

8. On Request changes: parse comment, re-run scenarios with adjusted
   inputs, post new card. Max 3 iterations before escalation.
```

## Hard rule

The agent **never** writes a price into Salesforce, the SOW document, or any external comm without an Approved status in Dataverse. Even on Approve, the generated SOW Word doc lands in the SharePoint *drafts* folder, not the customer-shared folder.

## Pricing model assumptions (encode as Power Fx constants)

- `MIN_MARGIN_FLOOR` = 22%
- `STANDARD_TARGET_MARGIN` = 30%
- `DEAL_DESK_CEILING` = 40%
- `DISCOUNT_BANDS_BY_TIER` = { Gold: [5, 12, 20], Silver: [3, 8, 15], Bronze: [0, 5, 10] }
- `CUSTOMER_TIER_BY_REVENUE` = revenue > $1B → Gold; > $250M → Silver; else Bronze

## What to demo

Run the prompt. Show the agent reading the SF opp. Show it calling the catalog mock (network trace in Power Automate). Show the adaptive card landing in #deal-desk. Approve it. Show the Word doc draft in SharePoint with prices merged in.
