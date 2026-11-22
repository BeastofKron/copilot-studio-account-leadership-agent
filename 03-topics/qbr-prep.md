# Topic: QBR / Account Health Prep

## Trigger phrases

- generate QBR for {customer}
- account health report for {customer}
- quarterly review deck for {customer}
- how is {customer} doing this quarter

## Inputs to collect

| Variable | Type | Prompt if missing |
|---|---|---|
| `CustomerName` | Text | "Which customer?" |
| `Quarter` | Text (Q1/Q2/Q3/Q4) | default = current quarter |
| `Year` | Number | default = current year |

## Flow logic

```
1. Resolve date range:
     QStart = first day of the quarter
     QEnd = last day of the quarter (or today if quarter still open)

2. Pull data in parallel via Power Automate "Apply to each":

   a. ServiceNow incidents
      - filter: company = CustomerName, opened_at BETWEEN [QStart, QEnd]
      - capture: count by priority, count by category, MTTR, top 5 by impact
      - SLA breach count

   b. ServiceNow change_request
      - filter: u_customer = CustomerName, end_date BETWEEN [QStart, QEnd]
      - capture: count completed, count in-flight, count failed
      - hours delivered (sum of u_actual_hours)

   c. Salesforce opportunities
      - filter: AccountId = SF Account for CustomerName
      - capture: stage progression in quarter, new opps opened, opps closed
        won/lost with TCV, pipeline movement

   d. SharePoint project status docs
      - search /sites/ms-{slug(CustomerName)}/Shared Documents/status-reports/
        for docs modified in quarter
      - extract executive summary text from each (Open XML)

   e. Dataverse AgentActionLog
      - filter: CustomerAccount = CustomerName, Timestamp in quarter
      - capture: count of agent actions taken, broken out by ActionType

3. Compute account health score (0–100):
     health = 100
            - (sla_breach_count × 5)
            - (failed_change_count × 8)
            - (lost_opp_count × 3)
            + min(20, hours_delivered / target_hours × 20)
     Cap at 100, floor at 0.

4. Generate a 12-slide PowerPoint via Microsoft Graph or a Power Automate
   "Create file" + Office Scripts call. Template lives at
   /sites/templates/qbr-template.pptx. Slide order:

      1. Title — Customer name, quarter, prepared by
      2. Executive summary — 4 bullets auto-generated from data
      3. Health score with traffic-light visual
      4. Incident summary (counts + MTTR + top 5 table)
      5. SLA performance
      6. Change delivery (completed/in-flight/failed bar chart)
      7. Project status updates (from SharePoint executive summaries)
      8. Salesforce pipeline movement
      9. Wins this quarter (closed-won opps)
      10. Risks / open items
      11. Recommendations (auto-drafted: top 3 themes from incident data)
      12. Next quarter focus areas (placeholder for human edit)

5. Save the deck to SharePoint /sites/ms-{slug(CustomerName)}/QBRs/
   filename: "{CustomerName}-QBR-{Quarter}{Year}-DRAFT.pptx"

6. Log to Dataverse AgentActionLog:
        ActionType = QBR
        Status = Executed (no approval needed — it's a draft)
        Payload = { deckURL, healthScore, dataSources: {...counts...} }

7. Reply in the account channel:
     "QBR draft for {CustomerName} {Quarter}{Year} ready.
      Health: {healthScore}/100 ({color})
      Deck: {deckURL}
      Sources stitched: {N} incidents, {M} changes, {K} opps, {J} status docs.
      Slides 11–12 need your input before this goes to the customer."
```

## Hard rule

- Draft only. Filename includes `-DRAFT`. Agent does not send to the customer.
- Health score formula is encoded as a constant — change it in one place if you want to demo different weights.

## What to demo

Run the prompt. Show the parallel data pulls in Power Automate run history (visual fan-out). Open the generated deck. Show the health score with traffic light. Show the auto-generated executive summary bullets. Point out slides 11–12 are placeholders.
