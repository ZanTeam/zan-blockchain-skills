# x402 Credits API Reference

Technical reference for credits management via the `@zan_team/x402` SDK. Credits are the unit of account for RPC calls — each method costs a defined number of credits. Credits are purchased with on-chain USDC.

## Overview

| Operation | SDK Method | Auth | Description |
|---|---|---|---|
| Check balance | `client.getBalance()` | JWT | Current credits and tier |
| Purchase credits | `client.purchaseCredits(bundle)` | JWT + Payment | Buy credits with USDC |
| Usage history | `client.getUsage(options)` | JWT | Recent usage records |
| Payment status | `client.credits.getPaymentStatus(paymentId)` | JWT | Check payment confirmation |

---

## getBalance

Query current credits balance and account tier.

### SDK Usage

```typescript
const balance = await client.getBalance();
```

### Response

| Field | Type | Description |
|---|---|---|
| `balance` | number | Available credits |
| `tier` | string | Account tier (e.g. `'free'`, `'basic'`, `'pro'`) |

### Example

```typescript
const { balance, tier } = await client.getBalance();
console.log(`Credits: ${balance}  Tier: ${tier}`);

if (balance < 10) {
  console.warn('Low credits — consider purchasing more');
}
```

### HTTP Equivalent

```http
GET /credits/balance HTTP/1.1
Authorization: Bearer <jwt>
```

```json
{
  "balance": 95,
  "tier": "basic"
}
```

---

## purchaseCredits

Purchase credits with on-chain USDC. The SDK handles the full 402 payment flow.

### SDK Usage

```typescript
const receipt = await client.purchaseCredits(bundle);
```

### Parameters

| Param | Type | Required | Description |
|---|---|---|---|
| `bundle` | `'default'` \| `'standard'` \| `'bulk'` | Yes | Credit package to purchase |

### Bundle Types

| Bundle | Description |
|---|---|
| `default` | Standard package (recommended for most use cases) |
| `standard` | Mid-tier package |
| `bulk` | Large package for high-volume usage |

Use `client.listBundles()` to discover available bundles and pricing dynamically.

### Response

| Field | Type | Description |
|---|---|---|
| `creditsPurchased` | number | Credits added |
| `txHash` | string | On-chain transaction hash |
| `balance` | number | New total balance |

### Example

```typescript
const receipt = await client.purchaseCredits('default');
console.log(`Purchased: ${receipt.creditsPurchased} credits`);
console.log(`Tx hash: ${receipt.txHash}`);
console.log(`New balance: ${receipt.balance}`);
```

### HTTP Equivalent

```http
POST /purchase/default HTTP/1.1
Authorization: Bearer <jwt>
Content-Type: application/json
```

Returns 402 with payment challenge → client signs → resubmits with payment → 200 with receipt.

### Errors

| Error | When |
|---|---|
| `InsufficientFundsError` | Wallet USDC balance too low for the bundle price |
| `PaymentRejectedError` | Facilitator rejected the payment signature |

---

## getUsage

Query recent usage records showing method calls, costs, and latency.

### SDK Usage

```typescript
const usage = await client.getUsage(options?);
```

### Parameters

| Param | Type | Required | Default | Description |
|---|---|---|---|---|
| `options.limit` | number | No | 20 | Max records to return |
| `options.offset` | number | No | 0 | Pagination offset |

### Response

| Field | Type | Description |
|---|---|---|
| `records` | UsageRecord[] | Usage entries |
| `total` | number | Total record count |

### UsageRecord

| Field | Type | Description |
|---|---|---|
| `methodName` | string | RPC method called (e.g. `eth_blockNumber`) |
| `ecosystem` | string | Chain ecosystem (e.g. `eth`, `bsc`) |
| `network` | string | Network (e.g. `mainnet`, `testnet`) |
| `creditCost` | number | Credits consumed |
| `latencyMs` | number | Response time in ms |
| `timestamp` | string | ISO 8601 timestamp |
| `status` | string | `success` or `error` |

### Example

```typescript
const usage = await client.getUsage({ limit: 10 });
console.log(`Total records: ${usage.total}`);

usage.records.forEach((r) => {
  console.log(
    `${r.timestamp} | ${r.methodName} | cost=${r.creditCost} | ${r.latencyMs}ms | ${r.status}`
  );
});
```

### HTTP Equivalent

```http
GET /credits/usage?limit=10 HTTP/1.1
Authorization: Bearer <jwt>
```

---

## getPaymentStatus

Check the on-chain confirmation status of a purchase.

### SDK Usage

```typescript
const status = await client.credits.getPaymentStatus(paymentId);
```

### Parameters

| Param | Type | Required | Description |
|---|---|---|---|
| `paymentId` | string | Yes | Payment ID from purchase receipt |

### Response

| Field | Type | Description |
|---|---|---|
| `status` | string | `pending`, `confirmed`, `failed` |
| `txHash` | string | On-chain tx hash |
| `confirmations` | number | Block confirmations |

---

## Credits Lifecycle

```
┌─────────────┐     ┌──────────────┐     ┌─────────────┐
│  Wallet has  │────>│  Purchase    │────>│  Credits     │
│  USDC        │     │  Credits     │     │  Available   │
└─────────────┘     └──────────────┘     └──────┬──────┘
                                                │
                                          RPC calls
                                          deduct credits
                                                │
                                         ┌──────▼──────┐
                                         │  Credits     │
                                         │  Depleted    │
                                         └──────┬──────┘
                                                │
                              autoPayment: true │ false
                              ┌─────────────────┼─────────────┐
                              ▼                               ▼
                    Auto re-purchase            InsufficientCreditsError
                    + retry                     (manual handling)
```

---

## Agent Notes

- **Balance check first**: Before batch operations, call `getBalance()` to verify sufficient credits.
- **Bundle discovery**: Use `listBundles()` to show users available packages and pricing before purchasing.
- **Usage analysis**: Use `getUsage()` to help users understand their consumption patterns.
- **Auto-payment vs manual**: Recommend `autoPayment: true` for unattended Agents; manual for interactive use where cost awareness matters.
- **Testnet USDC**: On testnet payment networks, USDC can be obtained from faucets — guide users accordingly.
