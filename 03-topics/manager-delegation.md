# Topic: Delivery Manager Task Delegation

## Trigger phrases

- assign {work} to a delivery manager
- delegate {work} for {customer}
- who should run the {work}
- find a manager for {work}

## Inputs to collect

| Variable | Type | Prompt if missing |
|---|---|---|
| `CustomerName` | Text | "Which customer?" |
| `WorkSummary` | Text | "What's the work in one sentence?" |
| `RequiredSkills` | Multi-select | "Which skills are needed? (Compute, Storage, Virtualization, Networking, Migration, Security)" |
| `TargetStartDate` | Date | "When should this start?" |
| `EstimatedHours` | Number | "Rough effort estimate?" |

## Flow logic

```
1. Query ServiceNow sys_user table:
     - department = "Managed Services"
     - u_skills contains any of RequiredSkills
   For each candidate, query open change_request count assigned to them
   with state in ("New", "Assess", "Authorize", "Scheduled", "Implement").
   Compute workload score = open_changes × avg_complexity.

2. Rank candidates by:
     a. Skill match count (descending)
     b. Workload score (ascending — less loaded first)
     c. Days since last new assignment (descending — round-robin tie-break)

3. Take top 3 candidates. For each, build a small profile:
     - Name, role, skills
     - Current open changes
     - Last 3 completed projects (from change_request closed_at last 90 days)

4. Log Pending action to Dataverse AgentActionLog:
        ActionType = Delegation
        Payload = { customer, workSummary, candidates: [...] }

5. Post adaptive card (see 04-adaptive-cards/delegation-approval.json) to the
   account team channel. The card shows:
     - Work summary + customer + required skills
     - Top 3 ranked candidates with workload + recent work
     - Radio: select which candidate to assign (default = top ranked)
     - Approve / Reject / Reassign-to-someone-else buttons

6. On Approve with a selected candidate:
     - Create ServiceNow change_request:
         short_description = "{Customer}: {WorkSummary}"
         assigned_to = selected candidate
         requested_by = signed-in user
         start_date = TargetStartDate
         u_estimated_hours = EstimatedHours
         description = full work brief (auto-drafted by agent from inputs)
     - Update Dataverse row Status = Executed, ApprovedBy = approver
     - Post in account channel: "Assigned to {Candidate}. SN change:
       {ChangeURL}. They've been notified via SN's standard assignment email."

7. On Reject: update Dataverse row Status = Rejected, post in channel:
     "Assignment cancelled. Reply with an alternative or ask me to re-rank."

8. On Reassign-to-someone-else: open a follow-up turn — "Who would you like?"
   Then create the SN change_request with that override and log Status =
   Approved with payload.overrideReason captured.
```

## Hard rule

The agent **proposes**, the human **assigns**. No SN change_request is created until the human picks a name in the adaptive card. If the human just dismisses the card, no change is made.

## What to demo

Run the prompt. Show the SN query in Power Automate run history. Show the adaptive card with 3 ranked candidates. Pick #2 instead of #1 to show the override path works. Show the SN change_request opened with the right assignee.
