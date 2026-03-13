# ZAN Operational APIs

Operational APIs provide **quota and service status** capabilities for monitoring API usage and resource availability. Authentication: **API Key only**.

---

## query-quota

Query API call quota and usage for the current account: total quota, used amount, remaining quota, reset time, and plan info.

### Endpoint

```
GET https://api.zan.top/operational/v1/quota?apiKey={API_KEY}
```

### Parameters

| Parameter | Type   | Required | Description                    |
|-----------|--------|----------|--------------------------------|
| `apiKey`  | string | Yes      | ZAN API Key for authentication |

Pass `apiKey` as a URL query parameter or via header: `Authorization: Bearer {API_KEY}`.

### Response Schema

| Field            | Type   | Description                                      |
|------------------|--------|--------------------------------------------------|
| `totalQuota`     | number | Total quota (calls per month/day, depends on plan) |
| `usedQuota`      | number | Used quota                                       |
| `remainingQuota` | number | Remaining quota                                  |
| `resetTime`      | string | Quota reset time (ISO 8601)                      |
| `plan`           | string | Plan name, e.g. `free`, `starter`, `pro`         |

### Example Response

```json
{
  "totalQuota": 100000,
  "usedQuota": 45230,
  "remainingQuota": 54770,
  "resetTime": "2025-04-01T00:00:00Z",
  "plan": "pro"
}
```

### Error Codes

| HTTP | Code           | Description              | Action                          |
|------|----------------|--------------------------|---------------------------------|
| 401  | `UNAUTHORIZED` | Invalid or expired API Key | Check API Key and validity      |
| 429  | `RATE_LIMITED` | Request rate exceeded    | Reduce frequency, retry later   |
| 500  | `INTERNAL_ERROR` | Server error             | Retry later or contact support  |

### curl Example

```bash
curl -X GET "https://api.zan.top/operational/v1/quota?apiKey=YOUR_API_KEY"
```

### TypeScript Example

```typescript
const API_KEY = process.env.ZAN_API_KEY;
const endpoint = `https://api.zan.top/operational/v1/quota?apiKey=${API_KEY}`;

const response = await fetch(endpoint);
const data = await response.json();

if (data.totalQuota !== undefined) {
  console.log("Total:", data.totalQuota, "Used:", data.usedQuota, "Remaining:", data.remainingQuota);
  if (data.remainingQuota < 1000) {
    console.warn("Quota low — consider upgrading plan");
  }
} else {
  console.error("Error:", data.error);
}
```

---

## Notes for Agents

- **Trigger**: Use when the user asks about "quota", "remaining calls", "API usage", "plan info", or "how many calls left".
- **Auth**: API Key only. Pass via `?apiKey=` or `Authorization: Bearer {API_KEY}`.
- **Interpretation**: When `remainingQuota` is near 0, suggest upgrading or waiting for `resetTime`.
- **Timing**: Query quota before batch or automated tasks to avoid mid-task rate limits.
