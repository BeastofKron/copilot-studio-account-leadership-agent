# Setup Guide — Managed Services VP Agent POC

Total time: ~3 hours including email verifications. All tenants free.

## 1. Microsoft 365 Developer Tenant (foundation)

This is the host for Copilot Studio, Teams, SharePoint, Outlook, and Dataverse.

1. Go to https://developer.microsoft.com/en-us/microsoft-365/dev-program and click **Join now**.
2. Sign in with a personal Microsoft account (not your work account).
3. Choose **Instant sandbox** with sample data pre-loaded. You get an E5 dev tenant for 90 days, auto-renewable while you stay active.
4. Note your tenant admin credentials — format `admin@<yourname>.onmicrosoft.com`.
5. Sign in to https://admin.microsoft.com with that admin account.
6. Under **Users → Active users**, confirm the sample users exist (Adele Vance, Megan Bowen, etc. — useful as fake delivery managers later).

## 2. Power Platform + Copilot Studio

1. Sign in to https://make.powerapps.com with your dev tenant admin.
2. Top right → **Environments** → confirm a default environment exists. Create a new one named `MSVP-POC` with **Dataverse enabled** (region: your nearest).
3. Go to https://copilotstudio.microsoft.com — accept the trial prompt. You'll get a 30-day Copilot Studio trial that you can extend.
4. In Copilot Studio, top right → switch environment to `MSVP-POC`.

## 3. Dataverse — AgentActionLog table (audit trail)

In https://make.powerapps.com → `MSVP-POC` environment → **Tables → New table**:

| Column | Type |
|---|---|
| `ActionType` | Choice: Onboarding, Pricing, Delegation, QBR, EmailSend |
| `CustomerAccount` | Text |
| `RequestedBy` | Text |
| `Status` | Choice: Pending, Approved, Rejected, Executed |
| `Payload` | Multiline text |
| `ApprovedBy` | Text (optional) |
| `Timestamp` | Date and Time |

This table is what the agent writes to every time it does anything — the portfolio demo will tail this table to show the audit trail.

## 4. Salesforce Developer Edition

1. Go to https://developer.salesforce.com/signup, sign up with a unique email (Gmail `+` aliases work: `you+sfdev@gmail.com`).
2. You'll get a free Dev Org that never expires while in use.
3. Sign in to your Dev Org. Confirm you can see **Accounts**, **Opportunities**, **Contacts** tabs.
4. **Setup → Apps → App Manager → New Connected App** to enable OAuth (you'll wire this into the Copilot Studio Salesforce connector). Note the Consumer Key and Secret.
5. Load `06-sample-data/acme-corp-customer.json` as an Account record (or just create "Acme Corp" manually with billing address, primary contact, and one open opportunity worth $750K).

## 5. ServiceNow Personal Developer Instance (PDI)

1. Go to https://developer.servicenow.com and sign up.
2. Request a Personal Developer Instance — pick the latest release. Takes ~10 min to provision.
3. Once live, sign in to `https://devXXXXXX.service-now.com` with admin credentials emailed to you.
4. **System Definition → Tables**: confirm `incident`, `change_request`, `sys_user` exist (they do by default).
5. Create 4 sample users to act as delivery managers: assign them department `Managed Services` and skill tags `Compute`, `Storage`, `Virtualization`, `Networking` (use the `u_skills` custom field if needed).
6. **System OAuth → Application Registry → New** → "Create an OAuth API endpoint for external clients". Note Client ID and Secret.

## 6. Connectors in Copilot Studio

In your `MSVP-POC` environment, **Power Automate → Connections → New connection**:

| Connector | Auth | Notes |
|---|---|---|
| Microsoft Teams | OAuth (use tenant admin) | For posting adaptive cards |
| Outlook 365 | OAuth | For drafting (not sending without approval) emails |
| SharePoint | OAuth | For onboarding site creation |
| Salesforce | OAuth (Consumer Key from step 4) | Pulls account/opportunity data |
| ServiceNow | Basic Auth or OAuth | Tasks, incidents, user lookup |
| Dataverse | Auto (current environment) | For AgentActionLog |
| HTTP | — | For catalog mock — see `05-catalog-mock/` |

## 7. Catalog mock

Follow `05-catalog-mock/README.md` to stand up a Power Automate HTTP-triggered flow that returns the mock catalog. Save the trigger URL — you'll plug it into the SOW pricing topic.

## 8. Create the agent shell

1. Copilot Studio → **Create → New agent**.
2. Choose **Skip to configure** (don't use the conversational wizard).
3. Name: `MS VP Agent`. Description: `Automates account leadership work for Managed Services on a customer account. Handles onboarding, SOW pricing assistance, delivery manager delegation, and QBR prep with human-in-the-loop approval.`
4. Icon: upload your brand color square (or skip).
5. Open **Settings → Generative AI** → set **Orchestration** to **Generative (preview)**. This lets the agent pick topics dynamically based on intent.
6. Open **Settings → Security** → set authentication to **Authenticate with Microsoft** so only signed-in tenant users can use it.
7. Open **Overview → Instructions** and paste in `02-agent-instructions.md`.

## 9. Add the 4 topics

For each file in `03-topics/`:
1. **Topics → Add topic → Create from blank**.
2. Set **Trigger** = the trigger phrases listed in that file.
3. Build out nodes following the flow logic in that file.
4. Use **Call an action → Create a flow** for anything that needs a Power Automate connector.

## 10. Channels

**Channels → Microsoft Teams → Turn on**. Once published, sideload the bot to a Teams channel called `#ms-acme-account-team`.

## 11. Smoke test

In Teams, message the bot: `Onboard Acme Corp`. You should get a clarifying question about kickoff date. Once you answer, the bot should drop an adaptive card asking you to approve creating the SharePoint site + Teams channel. Approve and watch them appear.

## 12. Record the demo

Use Microsoft Clipchamp (free, in your dev tenant) or OBS. Show all four prompts, the approval cards firing, and finally the Dataverse `AgentActionLog` table populated with the audit trail.
