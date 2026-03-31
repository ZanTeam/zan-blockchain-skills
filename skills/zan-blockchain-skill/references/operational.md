# ZAN Billing API Reference

> Technical reference for agents. Query real-time credits balance and plan info for ZAN accounts.

## Overview

Billing API allows ChainRPC users to query the real-time Credits balance under their current subscription plan via API Key. Use when you need:
- Remaining credits balance
- Total / used credits
- Current subscription plan name
- Whether auto-scaling (credit overusage) is enabled

**Auth:** API Key (same credentials used for all ZAN API calls).

**Official docs:** <https://docs.zan.top/reference/billing-api>

---

## API Methods

### credit/detail

**Description:** Query the real-time Credits balance under the current subscription plan.

**Endpoint:**

```
POST https://api.zan.top/platform/v1/credit/detail
```

**Request Parameters:**

| Parameter | Type   | Required | Description                                                                                     |
|-----------|--------|----------|-------------------------------------------------------------------------------------------------|
| `apiKey`  | String | Yes      | Client authentication key ([create API key](https://docs.zan.top/docs/quick-start-guide)) |

**Response Parameters:**

| Field            | Type    | Description                                      |
|------------------|---------|--------------------------------------------------|
| `success`        | Boolean | `true` indicates successful execution            |
| `code`           | String  | Error code (`null` when successful)              |
| `message`        | String  | Error message (`null` when successful)           |
| `data`           | Object  | Credit details container                         |
| `→ autoScaling`  | Boolean | Whether credit overusage is permitted            |
| `→ remainCredit` | Long    | Available credits balance                        |
| `→ specName`     | String  | Current subscription plan (e.g. `Free`, `Pro`)   |
| `→ totalCredit`  | Long    | Total allocated credits                          |
| `→ usedCredit`   | Long    | Consumed credits amount                          |

**Example Response:**

```json
{
  "success": true,
  "code": null,
  "message": null,
  "data": {
    "autoScaling": false,
    "remainCredit": 300000000,
    "specName": "Free",
    "totalCredit": 300000000,
    "usedCredit": 0
  }
}
```

**curl:**
```bash
curl -X POST "https://api.zan.top/platform/v1/credit/detail" \
  -H "Content-Type: application/json" \
  -d '{ "apiKey": "YOUR_API_KEY" }'
```

**TypeScript:**
```typescript
const API_KEY = process.env.ZAN_API_KEY!;

const response = await fetch("https://api.zan.top/platform/v1/credit/detail", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ apiKey: API_KEY }),
});
const { success, data } = await response.json();

if (success) {
  console.log("Plan:", data.specName);
  console.log("Used:", data.usedCredit, "/", data.totalCredit);
  console.log("Remaining:", data.remainCredit);
  console.log("Auto-scaling:", data.autoScaling);
}
```

---

## Practical Examples

### Pre-check before batch tasks (TypeScript)

```typescript
const API_KEY = process.env.ZAN_API_KEY!;

async function checkCreditsBeforeBatch(requiredCredits: number): Promise<boolean> {
  const res = await fetch("https://api.zan.top/platform/v1/credit/detail", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ apiKey: API_KEY }),
  });
  const { success, data } = await res.json();

  if (!success) {
    console.error("Failed to query billing info");
    return false;
  }

  if (data.remainCredit < requiredCredits) {
    console.warn(
      `Insufficient credits: need ${requiredCredits}, only ${data.remainCredit} left. ` +
      `Plan: ${data.specName}. Consider upgrading.`
    );
    return false;
  }

  console.log(`Credits OK: ${data.remainCredit} remaining, proceeding with batch.`);
  return true;
}

if (await checkCreditsBeforeBatch(50000)) {
  // run batch RPC calls...
}
```

### Low-credits alert (Python)

```python
import os, requests

API_KEY = os.environ["ZAN_API_KEY"]
THRESHOLD_RATIO = 0.1

resp = requests.post(
    "https://api.zan.top/platform/v1/credit/detail",
    json={"apiKey": API_KEY},
).json()

if resp.get("success"):
    data = resp["data"]
    remaining = data["remainCredit"]
    total = data["totalCredit"]
    plan = data["specName"]
    ratio = remaining / total if total > 0 else 0

    if ratio < THRESHOLD_RATIO:
        print(f"⚠ Low credits: {remaining}/{total} ({ratio:.1%}) on [{plan}] plan.")
        print("  → Upgrade at https://zan.top/pricing")
    else:
        print(f"✓ Credits healthy: {remaining}/{total} ({ratio:.1%}) remaining.")
else:
    print(f"Error: {resp.get('message')}")
```

### Agent output template

When presenting billing info to the user, follow this pattern:

```
Plan:         {specName}
Credits:      {usedCredit} / {totalCredit}
Remaining:    {remainCredit}
Auto-scaling: {autoScaling}
Status:       ✓ Healthy / ⚠ Low (< 10%) / ✗ Exhausted
Suggestion:   {actionable advice}
```

---

## Error Codes

| HTTP | Description          | Action                           |
|------|----------------------|----------------------------------|
| 200  | `success: false`     | Check `code` and `message` fields |
| 401  | Unauthorized         | Invalid or expired API Key       |
| 429  | Rate limited         | Reduce request frequency         |
| 500  | Server error         | Retry later or contact support   |

---

## Agent Notes

- **Trigger**: Use when the user asks about "credits", "remaining calls", "quota", "billing", "plan info", or "how many credits left".
- **Endpoint**: `POST https://api.zan.top/platform/v1/credit/detail` — this is NOT the Node Service endpoint; it uses the platform API.
- **Auth**: Pass `apiKey` in the JSON body, not in the URL path.
- **Interpretation**: When `remainCredit` < 10% of `totalCredit`, warn user and suggest upgrading plan.
- **Auto-scaling**: If `autoScaling` is `true`, credits can exceed `totalCredit` (overusage allowed). If `false`, service stops when credits are exhausted.
- **Operational hint**: For batch/automation tasks, check credits first to avoid mid-task failures.
- **Source of truth**: Always prefer <https://docs.zan.top/reference/billing-api> over examples in this file.
