# Catalog Mock — Setup

For the POC we don't want to hook into a real ERP / quoting / product-catalog system. We fake it with a Power Automate HTTP-triggered flow that returns a realistic catalog payload. The agent's SOW pricing topic POSTs to this URL and gets back priced line items.

## Steps

1. https://make.powerautomate.com → switch to the `MSVP-POC` environment.
2. **Create → Instant cloud flow** → trigger **When an HTTP request is received**.
3. Paste this as the request body schema:

```json
{
  "type": "object",
  "properties": {
    "serviceLines": { "type": "array", "items": { "type": "string" } },
    "termYears": { "type": "integer" },
    "customerTier": { "type": "string" }
  }
}
```

4. Add a **Compose** action. Open `mock-catalog.json` (sibling file) and paste its contents as the input expression. This is the static catalog the flow will return.

5. Add a **Filter array** action: filter the catalog's `items` where `serviceLine` is in `triggerBody()?['serviceLines']`. Output → `matchedItems`.

6. Add a second **Compose**: for each matched item, multiply `unitPrice` and `internalCost` by `termYears` to roll up to a contract total. Apply the discount band that matches `customerTier`.

7. Final **Response** action:
   - Status: 200
   - Headers: `Content-Type: application/json`
   - Body: `{ "items": <matchedItems with rolled-up totals>, "currency": "USD", "validUntil": "<today + 30 days>" }`

8. **Save**. Copy the HTTP POST URL the trigger generates (it has a SAS token in the query string — keep it secret).

9. In Copilot Studio: **Settings → Generative AI → Variables** → add environment variable `CATALOG_MOCK_URL` = the URL from step 8. The SOW pricing topic reads from this.

## Test it

```powershell
$body = @{
  serviceLines = @("Servers", "Support24x7Platinum")
  termYears    = 3
  customerTier = "Silver"
} | ConvertTo-Json

Invoke-RestMethod -Method Post -Uri $env:CATALOG_MOCK_URL `
  -ContentType "application/json" -Body $body
```

You should see a JSON response with priced line items.

## Why this is good enough for a portfolio POC

The mock matches the *shape* of any real product-catalog API — line items, costs, discount bands, term math. Swapping for a real system later is just changing the URL and the auth header; the agent's pricing logic doesn't have to change.
