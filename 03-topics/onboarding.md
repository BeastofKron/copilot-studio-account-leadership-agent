# Topic: Customer Onboarding

## Trigger phrases

- onboard a new customer
- start onboarding for {customer}
- new managed services customer
- spin up an account for {customer}

## Inputs to collect (slot filling)

| Variable | Type | Prompt if missing |
|---|---|---|
| `CustomerName` | Text | "Which customer are we onboarding?" |
| `PrimaryContactName` | Text | "Who's the primary contact at {CustomerName}?" |
| `PrimaryContactEmail` | Email | "What's their email?" |
| `KickoffDate` | Date | "When should we schedule the kickoff?" |
| `AccountTeamLead` | User (current default = signed-in user) | implicit |

## Flow logic

```
1. Confirm intent
   Bot: "I'll onboard {CustomerName} with kickoff {KickoffDate}. This will create
         a SharePoint site, Teams channel, kickoff invite, doc-request checklist,
         and a Salesforce account record. Proceed?"

2. On confirmation, log Pending action to Dataverse AgentActionLog:
      ActionType = Onboarding
      CustomerAccount = {CustomerName}
      Status = Pending
      Payload = <JSON of the inputs>

3. Call Power Automate flow: "PA_Onboard_Customer"
   The flow does, in order:
     a. SharePoint: create site "ms-{slug(CustomerName)}" under /sites/managedservices/
        - Apply Communication Site template
        - Add owners: AccountTeamLead, AccountTeamLead's manager
     b. Teams: create channel "#{slug(CustomerName)}-account" in the
        "Managed Services" team. Pin the SharePoint site as a tab.
     c. Salesforce: create Account record (or update if exists) with
        Name = CustomerName, OwnerId = AccountTeamLead
     d. Salesforce: create Contact record linked to Account
     e. Outlook: draft (NOT send) a kickoff invite for KickoffDate, 60 min,
        attendees = [PrimaryContactEmail, AccountTeamLead].
        Subject: "{CustomerName} + Managed Services — kickoff"
        Body: standard kickoff template from SharePoint /sites/templates/kickoff-email.html
     f. SharePoint: upload doc-request checklist to the new site
        from /sites/templates/onboarding-checklist.docx
     g. Dataverse: update AgentActionLog row to Status = Executed, append
        Payload with URLs of created resources

4. Bot reply: post a single Teams message in the new account channel:
     "Onboarding for {CustomerName} done. Created:
        • Site: {SharePointURL}
        • Channel: this one
        • SF Account: {SalesforceAccountURL}
        • Kickoff invite drafted (in your Outlook drafts — review and send)
        • Doc-request checklist uploaded"
```

## Approval gate

- **No external approval needed** — onboarding is entirely internal and reversible.
- Outlook invite is **drafted only**, not sent. User reviews and sends manually.

## Error handling

- If SharePoint site name collides, append `-2`, `-3` until unique.
- If Salesforce Account already exists, link to existing record and note it in the bot reply.
- If any step fails, mark the Dataverse row `Status = Rejected`, set `Payload.error = <message>`, and post the failure in the account channel with "Step X of onboarding failed: <reason>. Other steps that completed: <list>."

## What to demo

Show that one prompt produced 6 artifacts across 4 systems in ~30 seconds. Open each one. Show the Dataverse log row.
