# ZAN Solana Trading Boost — Consolidated Reference

> Technical reference for agents. SWQoS-based Solana transaction acceleration service.

## Overview

ZAN Solana Trading Boost uses **SWQoS (Stake-Weighted Quality of Service)** technology and ZAN’s staked validators to improve Solana transaction success rate and confirmation speed. It provides priority bandwidth allocation, low-latency confirmation (e.g. Prime: 95% of transactions confirmed within 1 second), and optional MEV protection.

## Authentication

| Auth Method | Supported |
|-------------|-----------|
| **API Key** | ✅ (only supported method) |

API Key is passed in the URL path as `{API_KEY}`. No other auth methods are supported.

---

## SWQoS Technology

Trading Boost is built on Solana’s [SWQoS (Stake-Weighted Quality of Service)](https://solana.com/developers/guides/advanced/stake-weighted-qos). ZAN uses staked validators to obtain priority transaction channels. Bandwidth is allocated by stake weight, giving users faster and more reliable confirmations.

---

## Integration Modes

| Mode | Description | Best For |
|------|-------------|----------|
| **Tips** | Pay SOL tips (≥0.0003 SOL) to a designated account; priority is bid-based | Custom strategies, bidding, dynamic priority |
| **Points** | Pre-purchase Points; fees deducted after successful confirmation; priority by tier | Simple workflows, fixed pricing, stable throughput |

---

## Tips Mode

### Endpoint

| Region | Endpoint |
|--------|----------|
| Singapore | `http://booster-sg.zan.top/node/v1/solana/mainnet/{API_KEY}/tips` |

### Requirements

- **Minimum tip**: 0.0003 SOL (300,000 lamports)
- **Recipient**: `CEWud6sUrg85auffmvG55cVhGjDbFCGC2HvhGeagd7St`
- **Payment**: Add a `SystemProgram.transfer` instruction to the transaction
- **Permission**: Tips mode must be enabled for your account; contact ZAN before first use

### Request

- **Method**: `POST`
- **Body**: Standard Solana `sendTransaction` JSON-RPC

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `jsonrpc` | string | Yes | `"2.0"` |
| `method` | string | Yes | `"sendTransaction"` |
| `params` | array | Yes | `[base64_encoded_tx, options]` |
| `id` | number | Yes | Request ID |

### curl Example

```bash
curl -X POST 'http://booster-sg.zan.top/node/v1/solana/mainnet/{YOUR_API_KEY}/tips' \
  -H 'Content-Type: application/json' \
  -d '{
    "jsonrpc": "2.0",
    "id": 1,
    "method": "sendTransaction",
    "params": [
      "<base64_encoded_tx>",
      { "encoding": "base64" }
    ]
  }'
```

### TypeScript Example

```typescript
import {
  Connection,
  Keypair,
  SystemProgram,
  Transaction,
  PublicKey,
} from "@solana/web3.js";
import bs58 from "bs58";

const API_KEY = process.env.ZAN_API_KEY!;
const TIPS_ENDPOINT = `http://booster-sg.zan.top/node/v1/solana/mainnet/${API_KEY}/tips`;
const TIPS_ACCOUNT = new PublicKey("CEWud6sUrg85auffmvG55cVhGjDbFCGC2HvhGeagd7St");
const TIPS_AMOUNT = 300_000; // 0.0003 SOL = 300,000 lamports

const connection = new Connection(TIPS_ENDPOINT, "confirmed");
const payer = Keypair.fromSecretKey(bs58.decode(process.env.SOLANA_PRIVATE_KEY!));

async function sendWithTips(transaction: Transaction) {
  transaction.add(
    SystemProgram.transfer({
      fromPubkey: payer.publicKey,
      toPubkey: TIPS_ACCOUNT,
      lamports: TIPS_AMOUNT,
    })
  );

  const { blockhash } = await connection.getLatestBlockhash();
  transaction.recentBlockhash = blockhash;
  transaction.feePayer = payer.publicKey;

  transaction.sign(payer);
  const rawTx = transaction.serialize();

  const signature = await connection.sendRawTransaction(rawTx, {
    skipPreflight: false,
    preflightCommitment: "confirmed",
  });

  return signature;
}
```

### Error Codes (Tips)

| HTTP Status | Description | Action |
|-------------|-------------|--------|
| 401 | Invalid or expired API Key | Verify API Key |
| 403 | Tips permission not enabled | Contact ZAN to enable |
| 429 | TPS exceeded | Reduce rate; excess falls back to standard |
| 500 | Server error | Retry later |

---

## Points Mode

### Endpoints

| Region | Base URL | Example (Prime) |
|--------|----------|-----------------|
| Singapore | `https://api.zan.top/node/v1/solana/mainnet/{API_KEY}/prime` | SG endpoint |
| US (Virginia) | `https://api-us.zan.top/node/v1/solana/mainnet/{API_KEY}/prime` | US endpoint |

### Service Tiers

| Tier | Path Segment | Priority | Points | Cost |
|------|---------------|----------|--------|------|
| **Prime** | `prime` | Dedicated channel | 5 Points | $0.5 |
| **Turbo** | `turbo` | High priority | 3 Points | $0.3 |
| **Standard** | `standard` | Shared channel | 1 Point | $0.1 |

- 1 Point = $0.1 USD
- Points are deducted only after successful on-chain confirmation
- If balance is insufficient, transactions fall back to standard (non-accelerated) handling

### URL Parameters

| Parameter | Description | Values |
|-----------|-------------|--------|
| `level` | Tier (via path) | `prime`, `turbo`, `standard` |
| `anti-mev` | MEV protection | `true` (enable), omit or `false` (disable) |

If no tier is specified, transactions are processed as standard (non-accelerated).

### Request

- **Method**: `POST`
- **Body**: Standard Solana `sendTransaction` JSON-RPC (no extra params)

### curl Examples

**Prime tier:**

```bash
curl -X POST 'https://api.zan.top/node/v1/solana/mainnet/{YOUR_API_KEY}/prime' \
  -H 'Content-Type: application/json' \
  -d '{
    "jsonrpc": "2.0",
    "id": 1,
    "method": "sendTransaction",
    "params": [
      "<base64_encoded_tx>",
      { "encoding": "base64" }
    ]
  }'
```

**Turbo tier + MEV protection:**

```bash
curl -X POST 'https://api.zan.top/node/v1/solana/mainnet/{YOUR_API_KEY}/turbo?anti-mev=true' \
  -H 'Content-Type: application/json' \
  -d '{
    "jsonrpc": "2.0",
    "id": 1,
    "method": "sendTransaction",
    "params": [
      "<base64_encoded_tx>",
      { "encoding": "base64" }
    ]
  }'
```

### TypeScript Example

```typescript
import { Connection, Keypair, SystemProgram, Transaction, PublicKey } from "@solana/web3.js";
import bs58 from "bs58";

const API_KEY = process.env.ZAN_API_KEY!;
const LEVEL = "prime"; // prime | turbo | standard
const ENDPOINT = `https://api.zan.top/node/v1/solana/mainnet/${API_KEY}/${LEVEL}`;

// With MEV protection:
// const ENDPOINT = `https://api.zan.top/node/v1/solana/mainnet/${API_KEY}/${LEVEL}?anti-mev=true`;

const connection = new Connection(ENDPOINT, "confirmed");
const payer = Keypair.fromSecretKey(bs58.decode(process.env.SOLANA_PRIVATE_KEY!));

async function sendWithBoost(transaction: Transaction) {
  const { blockhash } = await connection.getLatestBlockhash();
  transaction.recentBlockhash = blockhash;
  transaction.feePayer = payer.publicKey;

  transaction.sign(payer);
  const rawTx = transaction.serialize();

  const signature = await connection.sendRawTransaction(rawTx, {
    skipPreflight: false,
    preflightCommitment: "confirmed",
  });

  return signature;
}

function getBoostEndpoint(
  apiKey: string,
  level: "prime" | "turbo" | "standard",
  region: "sg" | "us" = "sg",
  antiMev = false
): string {
  const base = region === "us"
    ? `https://api-us.zan.top/node/v1/solana/mainnet/${apiKey}/${level}`
    : `https://api.zan.top/node/v1/solana/mainnet/${apiKey}/${level}`;
  return antiMev ? `${base}?anti-mev=true` : base;
}
```

### Error Codes (Points)

| HTTP Status | Description | Action |
|-------------|-------------|--------|
| 401 | Invalid or expired API Key | Verify API Key |
| 429 | TPS exceeded | Reduce rate; excess falls back to standard |
| 500 | Server error | Retry later |

---

## MEV Protection

Both Tips and Points support free MEV protection. Append `?anti-mev=true` to the endpoint URL.

**Examples:**

- Tips: `http://booster-sg.zan.top/node/v1/solana/mainnet/{API_KEY}/tips?anti-mev=true`
- Points: `https://api.zan.top/node/v1/solana/mainnet/{API_KEY}/prime?anti-mev=true`

**Notes:**

- MEV protection may add small latency during peak periods
- Recommended for high-value transactions; optional for low-value ones
- Monitor latency via [ZAN Dashboard](https://zan.top/service/sol/overview)

---

## Throughput Limits

- Trading Boost shares API Key and throughput limits with Chain RPC
- Requests beyond limits are automatically handled as standard (non-accelerated)
- For higher throughput: upgrade Chain RPC plan in [ZAN Console](https://zan.top/personal/price-plan?product=NODE_SERVICE) or [contact solutions](https://zan.top/home/contact)

---

## Monitoring

### Dashboard

- ZAN Portal → SOL Trading → Overview
- Switch between Points and Tips views

### Metrics

| Metric | Description |
|--------|-------------|
| **Success Rate** | Confirmation rate per tier |
| **On-Chain Time** | Time from submit to on-chain (excludes client→ZAN) |
| **Remaining Points** | Current Points balance (Points mode only) |

### Transaction Verification

- Search by transaction hash in SOL Trading Boost → “View consumption details”
- Only last 30 days are shown
- If a transaction does not appear, it was not accelerated

---

## Response Schema

Both modes return standard Solana `sendTransaction` response:

| Field | Type | Description |
|-------|------|-------------|
| `jsonrpc` | string | `"2.0"` |
| `id` | number | Same as request ID |
| `result` | string | Transaction signature |

**Example:**

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": "5VERv8NMhJzWpLvQhKmRwKbPjKp3yP3RGi7Nk8tCWqNhR7EhXkQvZLBnCsJkzKPwDNFgvJHb4kza2NHe7KwbFgS"
}
```

---

## FAQ

### Why might transaction acceleration fail?

- TPS limit exceeded
- Points balance insufficient (Points mode)

### How do I verify a transaction was accelerated?

Search by transaction hash in “View consumption details”. If it does not appear, it was not accelerated.

### Is testnet supported?

No. Only Solana mainnet is supported.

### What if Points balance is insufficient?

Transactions are automatically processed as standard (non-accelerated). They do not fail due to Points shortage.

### Do Tips get refunded if the transaction fails?

No. Tips are on-chain SOL transfers and are not refunded regardless of transaction outcome.

---

## Risk Disclaimer

1. All data and documentation are for reference only and do not constitute investment or trading advice.
2. Transaction acceleration involves on-chain transactions and fees (SOL tips or Points). Users should assess risk and cost.
3. MEV protection may add small latency; balance security and speed for your use case.
4. Use blockchain services in compliance with applicable laws and regulations.

---

## Support

- [Telegram](https://t.me/ZANTeam_official)
- [Discord](https://discord.gg/yZzn9BEtFU)
- [Contact](https://zan.top/home/contact)

---

## References

- [ZAN Trading Boost Docs](https://docs.zan.top/docs/trading-boost)
- [Solana SWQoS](https://solana.com/developers/guides/advanced/stake-weighted-qos)
- [ZAN sendTransaction API](https://docs.zan.top/reference/sendtransaction-solana)
