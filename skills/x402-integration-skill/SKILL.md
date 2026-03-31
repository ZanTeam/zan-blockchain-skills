---
name: x402-integration-skill
description: >-
  x402 protocol integration — pay-per-call blockchain RPC via USDC on-chain settlement.
  TypeScript SDK (@zan_team/x402) provides SIWE/SIWS wallet authentication, transparent
  402 auto-payment flow, JSON-RPC calls across 20+ chains, credits balance/purchase/usage
  management, and gateway capability discovery. No API Key registration required — authenticate
  with any EVM or Solana wallet and pay with USDC.
  Triggers on: x402, USDC, credits, pay-per-call, 即用即付, 购买额度, wallet auth, SIWE,
  SIWS, createX402Client, client.call, autoPayment, 402 payment, x402 gateway.
license: Apache-2.0
metadata:
  author: zan-team
  version: "1.0"
---

# x402 Protocol Integration

## Overview

x402 is an HTTP-native payment protocol that extends HTTP 402 (Payment Required) for machine-to-machine micropayments. The `@zan_team/x402` TypeScript SDK enables AI Agents and applications to access blockchain RPC services through a unified gateway, settling in **on-chain USDC** — no API Key registration needed.

**Key differentiator vs API Key**: x402 is wallet-native. Authenticate with any EVM or Solana wallet, pay per use with USDC, and start calling RPCs immediately.

## Intake Questions

- Which wallet type? EVM (0x private key) or Solana (Base58 secret key)?
- What chain and network do you want to query? (eth/mainnet, bsc/mainnet, solana/mainnet, etc.)
- Do you want automatic payment when credits run out? (`autoPayment: true`)
- Which payment network for USDC settlement? (Base, Base Sepolia, Solana Mainnet, Solana Devnet)
- Need the full bootstrap example or a specific API (auth, credits, RPC, discovery)?

## Safety Defaults

- Default gateway: `https://x402.zan.top`
- Default `autoPayment: false` — explicit opt-in for on-chain payments
- Never ask for or log private keys; always use environment variables
- Private keys are used locally for SIWE/SIWS signing only — never sent to the gateway
- All data is for reference only; never provide investment or trading advice

## Quick Reference

| Capability | SDK Method | Description |
|---|---|---|
| **Initialize** | `createX402Client(config)` | Factory with optional pre-auth |
| **RPC Call** | `client.call(ecosystem, network, method, params?)` | JSON-RPC to any supported chain |
| **Batch RPC** | `client.rpc.batch(ecosystem, network, requests[])` | Multiple RPCs in one call |
| **Balance** | `client.getBalance()` | Check credits balance and tier |
| **Purchase** | `client.purchaseCredits(bundle)` | Buy credits with USDC |
| **Usage** | `client.getUsage({ limit })` | Recent usage records |
| **Health** | `client.health()` | Gateway health (no auth) |
| **Networks** | `client.listNetworks()` | Available chains (no auth) |
| **Bundles** | `client.listBundles()` | Credit packages (no auth) |
| **x402 Cap** | `client.getX402Capability()` | Payment capability info (no auth) |
| **Fetch** | `client.fetch(path, init)` | Raw HTTP with auto JWT + 402 handling |

## Installation

```bash
npm install @zan_team/x402 viem
```

Solana optional (only for SVM auth/payment):

```bash
npm install @solana/web3.js @solana/spl-token bs58 tweetnacl
```

Requires **Node.js >= 18**.

## Quick Start — EVM

```typescript
import { createX402Client } from '@zan_team/x402';

const client = await createX402Client({
  gatewayUrl: 'https://x402.zan.top',
  privateKey: process.env.PRIVATE_KEY as `0x${string}`,
  autoPayment: true,
  preAuth: true,
});

const block = await client.call('eth', 'mainnet', 'eth_blockNumber');
console.log('Latest block:', block.result);

const balance = await client.getBalance();
console.log(`Credits: ${balance.balance}  Tier: ${balance.tier}`);
```

## Quick Start — Solana

```typescript
import { createX402Client } from '@zan_team/x402';

const client = await createX402Client({
  gatewayUrl: 'https://x402.zan.top',
  svmPrivateKey: process.env.SVM_PRIVATE_KEY!,
  autoPayment: true,
  preAuth: true,
});

const slot = await client.call('solana', 'mainnet', 'getSlot');
console.log('Current slot:', slot.result);
```

SDK auto-detects `chainType` from the provided key.

## Configuration

```typescript
interface X402ClientConfig {
  gatewayUrl: string;
  // EVM
  wallet?: WalletClient;
  privateKey?: `0x${string}`;
  // SVM
  svmPrivateKey?: string;       // Base58
  paymentNetwork?: string;      // CAIP-2, e.g. "eip155:8453"
  solanaRpcUrl?: string;
  // Behavior
  chainType?: 'EVM' | 'SVM';   // auto-detected from keys
  autoPayment?: boolean;        // default: false
  defaultBundle?: BundleType;   // default: 'default'
  preAuth?: boolean;            // pre-authenticate on creation
  timeout?: number;             // ms, default: 30000
  fetch?: typeof fetch;         // custom fetch impl
}
```

### Payment Networks (CAIP-2)

| CAIP-2 ID | Chain | Type |
|---|---|---|
| `eip155:8453` | Base Mainnet | EVM |
| `eip155:84532` | Base Sepolia | EVM Testnet |
| `solana:5eykt4UsFv8P8NJdTREpY1vzqKqZKvdp` | Solana Mainnet | SVM |
| `solana:EtWTRABZaYq6iMfeYKouRu166VU2xqa1` | Solana Devnet | SVM Testnet |

## Error Handling

All errors extend `X402Error` with typed `code` and optional `statusCode`.

| Error | HTTP | When |
|---|---|---|
| `AuthenticationError` | 401 | SIWE/SIWS rejected |
| `SessionExpiredError` | 401 | JWT expired |
| `InsufficientCreditsError` | 402 | Not enough credits |
| `InsufficientFundsError` | 402 | On-chain USDC too low |
| `PaymentRejectedError` | 402 | Facilitator rejected payment |
| `MethodNotAllowedError` | 403 | Method not in current tier |
| `ProviderNotFoundError` | 404 | No matching provider |
| `UpstreamError` | 504 | Provider failure |
| `NetworkError` | — | Transport / timeout |

```typescript
import { InsufficientCreditsError, SessionExpiredError } from '@zan_team/x402';

try {
  await client.call('eth', 'mainnet', 'eth_blockNumber');
} catch (err) {
  if (err instanceof InsufficientCreditsError) {
    console.error(`Need ${err.required}, have ${err.balance}`);
  } else if (err instanceof SessionExpiredError) {
    await client.authenticate();
  }
}
```

## Auto-Payment Flow

When `autoPayment: true`, the SDK handles 402 challenges transparently:

```
Client                   Gateway                  Facilitator
  │── POST /rpc/eth/main ─>│
  │<── 402 ────────────────│
  │── POST /purchase/default ──────────────────────>│
  │<── 402 + PAYMENT-REQUIRED ─────────────────────│
  │  [sign EIP-3009 / Solana SPL locally]
  │── POST /purchase/default (+ signature) ────────>│
  │                        │── POST /verify ───────>│
  │                        │<── valid, txHash ──────│
  │<── 200 + credits ──────│
  │── POST /rpc/eth/main ─>│  [retry]
  │<── 200 + result ───────│
```

See [references/auto-payment.md](references/auto-payment.md) for detailed flow.

## Skill Selection Guide

| User Intent | Recommended Action |
|---|---|
| Call blockchain RPC without API Key | **x402 RPC** (`client.call`) |
| Check/buy credits with USDC | **Credits** (`getBalance`, `purchaseCredits`) |
| Discover available chains and bundles | **Discovery** (`listNetworks`, `listBundles`) |
| Transparent HTTP with JWT + 402 handling | **Fetch** (`client.fetch`) |
| Batch multiple RPC calls | **Batch RPC** (`client.rpc.batch`) |

## vs API Key Access

| Dimension | API Key | x402 |
|---|---|---|
| Registration | Required (ZAN Console) | Not required |
| Authentication | API Key in URL | Wallet signature (SIWE/SIWS) |
| Payment | Subscription / prepaid | Pay-per-call USDC |
| Setup time | Minutes | Seconds (have wallet = ready) |
| Best for | Stable workloads | AI Agents, experiments, dynamic |

## Ambiguity Resolution

If user intent is unclear, ask:
- Are you using an EVM wallet (0x key) or Solana wallet (Base58)?
- Which chain do you want to query?
- Do you want auto-payment enabled? (on-chain USDC will be spent)
- Is this a testnet experiment or mainnet usage?
- Do you need the SDK code or a curl example?

## Alternative Access: API Key

If the user already has a **ZAN API Key** or prefers traditional subscription-based access, use the **[ZAN Blockchain Skill](../zan-blockchain-skill/SKILL.md)** instead. It provides the same RPC capabilities plus Advanced Data API (NFT/Token), Solana Trading Boost, and Operational APIs — all authenticated via API Key.

| Dimension | This Skill (x402) | API Key Skill |
|---|---|---|
| Registration | Not required | Required (ZAN Console) |
| Authentication | Wallet signature (SIWE/SIWS) | API Key in URL |
| Payment | Pay-per-call USDC | Subscription / prepaid |
| Extra capabilities | — | Data API, Trading Boost, Operational |
| Best for | AI Agents, experiments, dynamic | Stable workloads, enterprise |

## References

- [SDK Quick Start](references/sdk-quickstart.md) — installation, init, first RPC call
- [Auto Payment](references/auto-payment.md) — 402 flow, EIP-3009, Solana SPL
- [Credits API](references/credits-api.md) — balance, purchase, usage, bundles
- [Discovery API](references/discovery-api.md) — health, networks, bundles, x402 capability

## Risk Disclaimer

1. All data is for reference only; does not constitute investment or trading advice.
2. `autoPayment: true` will spend on-chain USDC automatically — use with caution on mainnet.
3. Private keys are used for local signing only and never transmitted to the gateway.
4. Use blockchain services in compliance with applicable laws and regulations.
