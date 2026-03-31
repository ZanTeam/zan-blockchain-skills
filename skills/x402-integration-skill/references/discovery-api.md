# x402 Discovery API Reference

Technical reference for gateway discovery capabilities via the `@zan_team/x402` SDK. All discovery endpoints are **public** — no authentication required. Use them at startup to probe available networks, bundles, and gateway health.

## Overview

| Operation | SDK Method | Auth | Description |
|---|---|---|---|
| Health check | `client.health()` | None | Gateway status |
| List networks | `client.listNetworks()` | None | Available chains and networks |
| List bundles | `client.listBundles()` | None | Credit packages and pricing |
| x402 capability | `client.getX402Capability()` | None | Payment protocol details |

All discovery methods work without JWT — ideal for Agent startup probes and dynamic tool catalogs.

---

## health

Check gateway health and availability.

### SDK Usage

```typescript
const health = await client.health();
```

### Response

| Field | Type | Description |
|---|---|---|
| `status` | string | `'healthy'` or error state |
| `version` | string | Gateway version |
| `timestamp` | string | Server timestamp (ISO 8601) |

### Example

```typescript
const health = await client.health();
if (health.status === 'healthy') {
  console.log(`Gateway v${health.version} is up`);
} else {
  console.error('Gateway unhealthy:', health);
}
```

### HTTP Equivalent

```http
GET /health HTTP/1.1
```

```json
{
  "status": "healthy",
  "version": "1.0.0",
  "timestamp": "2026-03-31T10:00:00Z"
}
```

---

## listNetworks

Discover all chains and networks supported by the gateway.

### SDK Usage

```typescript
const { networks } = await client.listNetworks();
```

### Response

| Field | Type | Description |
|---|---|---|
| `networks` | NetworkInfo[] | Available networks |

### NetworkInfo

| Field | Type | Description |
|---|---|---|
| `ecosystem` | string | Chain ecosystem (e.g. `eth`, `bsc`, `solana`) |
| `network` | string | Network name (e.g. `mainnet`, `testnet`) |
| `chainId` | string | Chain ID or identifier |
| `name` | string | Human-readable name |
| `rpcPath` | string | RPC path segment |

### Example

```typescript
const { networks } = await client.listNetworks();
console.log(`${networks.length} networks available:`);

networks.forEach(n => {
  console.log(`  ${n.ecosystem}/${n.network} — ${n.name} (${n.chainId})`);
});

// Filter for a specific ecosystem
const ethNetworks = networks.filter(n => n.ecosystem === 'eth');
console.log(`Ethereum networks: ${ethNetworks.length}`);
```

### HTTP Equivalent

```http
GET /discovery/networks HTTP/1.1
```

### Use Cases

- **Agent startup**: Enumerate supported chains before accepting user queries
- **Dynamic routing**: Select ecosystem/network based on user request
- **Capability check**: Verify a specific chain is available before calling

---

## listBundles

Discover available credit packages and pricing.

### SDK Usage

```typescript
const { bundles } = await client.listBundles();
```

### Response

| Field | Type | Description |
|---|---|---|
| `bundles` | BundleInfo[] | Available packages |

### BundleInfo

| Field | Type | Description |
|---|---|---|
| `name` | string | Bundle identifier (e.g. `default`, `standard`, `bulk`) |
| `credits` | number | Credits included |
| `price` | string | USDC price |
| `description` | string | Human-readable description |

### Example

```typescript
const { bundles } = await client.listBundles();
bundles.forEach(b => {
  console.log(`${b.name}: ${b.credits} credits @ ${b.price} USDC — ${b.description}`);
});

// Find the best value bundle
const bestValue = bundles.reduce((best, b) =>
  (b.credits / parseFloat(b.price)) > (best.credits / parseFloat(best.price)) ? b : best
);
console.log(`Best value: ${bestValue.name}`);
```

### HTTP Equivalent

```http
GET /discovery/bundles HTTP/1.1
```

### Use Cases

- **Pricing display**: Show users available packages before purchasing
- **Auto-select**: Choose bundle based on expected usage volume
- **Cost estimation**: Calculate per-call cost from bundle pricing

---

## getX402Capability

Query the gateway's x402 payment protocol capabilities.

### SDK Usage

```typescript
const capability = await client.getX402Capability();
```

### Response

| Field | Type | Description |
|---|---|---|
| `supported` | boolean | Whether x402 is enabled |
| `version` | string | x402 protocol version |
| `paymentNetworks` | PaymentNetwork[] | Supported settlement chains |
| `facilitator` | string | Facilitator endpoint |

### PaymentNetwork

| Field | Type | Description |
|---|---|---|
| `network` | string | CAIP-2 identifier |
| `token` | string | Settlement token (USDC) |
| `contract` | string | Token contract address |

### Example

```typescript
const cap = await client.getX402Capability();
if (cap.supported) {
  console.log(`x402 v${cap.version} supported`);
  console.log('Payment networks:');
  cap.paymentNetworks.forEach(pn => {
    console.log(`  ${pn.network} — ${pn.token} @ ${pn.contract}`);
  });
} else {
  console.log('x402 not available — use API Key access instead');
}
```

### HTTP Equivalent

```http
GET /.well-known/x402 HTTP/1.1
```

### Use Cases

- **Protocol detection**: Check if the gateway supports x402 before attempting payment
- **Network selection**: Choose which chain to settle USDC on
- **Fallback**: If x402 unavailable, guide user to API Key access

---

## Startup Pattern for Agents

Recommended initialization sequence for AI Agents:

```typescript
import { createX402Client } from '@zan_team/x402';

async function initAgent() {
  const client = await createX402Client({
    gatewayUrl: 'https://x402.zan.top',
    privateKey: process.env.PRIVATE_KEY as `0x${string}`,
    autoPayment: true,
    preAuth: true,
  });

  // 1. Health check
  const health = await client.health();
  if (health.status !== 'healthy') {
    throw new Error(`Gateway unhealthy: ${health.status}`);
  }

  // 2. Discover networks
  const { networks } = await client.listNetworks();
  const supportedChains = networks.map(n => `${n.ecosystem}/${n.network}`);

  // 3. Discover bundles for cost awareness
  const { bundles } = await client.listBundles();

  // 4. Check x402 capability
  const cap = await client.getX402Capability();

  // 5. Check balance
  const balance = await client.getBalance();

  return {
    client,
    supportedChains,
    bundles,
    x402: cap,
    balance: balance.balance,
  };
}
```

---

## Agent Notes

- **No auth needed**: All discovery endpoints work without JWT — safe to call before authentication.
- **Cache-friendly**: Network and bundle lists change infrequently. Cache results for the session.
- **Dynamic tools**: Use `listNetworks()` to dynamically build the set of chains an Agent can query.
- **Cost transparency**: Always show bundle pricing to users before auto-purchasing credits.
- **Health monitoring**: Call `health()` periodically for long-running Agent processes.
