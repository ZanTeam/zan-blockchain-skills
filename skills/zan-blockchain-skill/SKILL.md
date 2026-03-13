---
name: zan-blockchain-skill
description: >-
  ZAN blockchain infrastructure — RPC endpoints (20+ EVM and non-EVM chains),
  Advanced Data API (NFT metadata/holders/transfers, Token metadata/holders/balances,
  Debug trace, Simulation), Solana Trading Boost (SWQoS-based acceleration via Tips
  and Points modes with MEV protection), and Operational APIs (quota queries).
  Use when querying balances, blocks, transactions, calling contracts, estimating gas,
  analyzing NFT/Token holders, tracing transactions, accelerating Solana trades,
  or checking service quotas.
  Triggers on mentions of ZAN, eth_blockNumber, eth_getBalance, eth_call, eth_estimateGas,
  zan_getNFTMetadata, zan_getNFTHolders, zan_getTokenMetadata, zan_getTokenHolders,
  debug_traceTransaction, zan_simulateAssetChanges, Trading Boost, SWQoS, Tips, Points,
  Solana acceleration, MEV protection, or blockchain RPC.
license: Apache-2.0
metadata:
  author: zan-team
  version: "1.0"
---

# ZAN Blockchain Infrastructure

## Intake Questions
- Which chain and network? (Ethereum, Base, Polygon, Arbitrum, Optimism, Solana, etc.)
- Is this a read-only query (balance, block, transaction) or data analysis (NFT/Token holders, traces)?
- Do you need Solana transaction acceleration (Trading Boost)?
- What API Key should I use? (env: `ZAN_API_KEY`)

## Safety Defaults
- Default to mainnet unless testnet is explicitly requested.
- Prefer read-only operations (queries, traces, simulations).
- Never ask for or accept private keys or secret keys.
- All data is for reference only; never provide investment or trading advice.

## Quick Reference

| Product | Endpoint Pattern | Description |
|---------|-----------------|-------------|
| **Chain RPC** | `https://api.zan.top/node/v1/{chain}/{network}/{API_KEY}` | Standard JSON-RPC for 20+ chains |
| **Advanced Data API** | `https://api.zan.top/data/v1/{chain}/{network}/{API_KEY}` | NFT, Token, Debug, Simulation APIs |
| **Trading Boost (Tips)** | `http://booster-sg.zan.top/node/v1/solana/mainnet/{API_KEY}/tips` | Solana acceleration via SOL tips |
| **Trading Boost (Points)** | `https://api.zan.top/node/v1/solana/mainnet/{API_KEY}/prime` | Solana acceleration via Points credits |
| **Operational** | `https://api.zan.top/operational/v1/quota?apiKey={API_KEY}` | Quota and usage queries |

## Authentication

All APIs use **API Key** authentication, passed as a URL path parameter:

```bash
curl -X POST https://api.zan.top/node/v1/eth/mainnet/{YOUR_API_KEY} \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}'
```

## Chain RPC

Standard JSON-RPC 2.0 interface for blockchain queries. Supports 20+ chains including Ethereum, BNB Smart Chain, Polygon, Arbitrum, Optimism, Base, Avalanche, Fantom, Starknet, Sui, Aptos, Bitcoin, Solana, TON, and more.

### Key Methods

| Method | Description | Example params |
|--------|-------------|---------------|
| `eth_blockNumber` | Latest block height | `[]` |
| `eth_getBalance` | Native token balance (wei) | `[address, "latest"]` |
| `eth_getTransactionByHash` | Transaction details | `[txHash]` |
| `eth_call` | Read-only contract call | `[{to, data}, "latest"]` |
| `eth_estimateGas` | Estimate gas for a tx | `[{from, to, value, data}]` |

### Quick Example

```typescript
const API_KEY = process.env.ZAN_API_KEY!;
const endpoint = `https://api.zan.top/node/v1/eth/mainnet/${API_KEY}`;

const res = await fetch(endpoint, {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({
    jsonrpc: "2.0", method: "eth_getBalance",
    params: ["0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb", "latest"], id: 1,
  }),
});
const { result } = await res.json();
const balanceEth = Number(BigInt(result)) / 1e18;
```

See [references/chain-rpc.md](references/chain-rpc.md) for all methods, parameters, response schemas, and code samples.

## Advanced Data API

JSON-RPC 2.0 interface for structured data queries: NFT metadata/holders/transfers, Token metadata/holders/balances, transaction tracing, and execution simulation.

### API Groups

| Group | Methods | Key Methods |
|-------|---------|-------------|
| **NFT API** | 8 | `zan_getNFTMetadata`, `zan_getNFTHolders`, `zan_getNftTransfers`, `zan_getNFTsByOwner` |
| **Token API** | 7 | `zan_getTokenMetadata`, `zan_getTokenHolders`, `zan_getTokensByOwner`, `zan_getTokenBalanceByOwner` |
| **Debug API** | 5 | `debug_traceTransaction`, `debug_traceCall`, `debug_traceBlockByHash` |
| **Simulation API** | 2 | `zan_simulateAssetChanges`, `zan_simulateExecution` |

### Quick Example

```typescript
const endpoint = `https://api.zan.top/data/v1/eth/mainnet/${API_KEY}`;

const res = await fetch(endpoint, {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({
    id: 1, jsonrpc: "2.0",
    method: "zan_getNFTHolders",
    params: ["0xBC4CA0EdA7647A8aB7C2061c2E118A18a936f13D", 10],
  }),
});
const { result } = await res.json(); // array of holder addresses
```

See [references/data-api.md](references/data-api.md) for all 22 methods with parameters and examples.

## Solana Trading Boost

SWQoS-based transaction acceleration for Solana, with two integration modes:

| Mode | Mechanism | Cost |
|------|-----------|------|
| **Tips** | Pay ≥0.0003 SOL to designated account | Variable (bid-based) |
| **Points** | Pre-purchase credits, deducted on success | Prime $0.5, Turbo $0.3, Standard $0.1 |

Both modes support free **MEV Protection** via `?anti-mev=true`.

### Quick Example (Points)

```typescript
import { Connection, Transaction } from "@solana/web3.js";
const conn = new Connection(
  `https://api.zan.top/node/v1/solana/mainnet/${API_KEY}/prime`
);
// Use conn.sendRawTransaction() as normal — acceleration is automatic
```

See [references/trading-boost.md](references/trading-boost.md) for Tips/Points integration details, endpoints, and code samples.

## Operational API

Query API usage quotas and service status.

```typescript
const res = await fetch(
  `https://api.zan.top/operational/v1/quota?apiKey=${API_KEY}`
);
const { totalQuota, usedQuota, remainingQuota } = await res.json();
```

See [references/operational.md](references/operational.md) for details.

## Skill Selection Guide

| User Intent | Recommended API |
|-------------|----------------|
| Balance, block, transaction, gas, contract call | **Chain RPC** |
| NFT/Token holders, metadata, transfers | **Data API (NFT/Token)** |
| Transaction tracing, call simulation | **Data API (Debug/Simulation)** |
| Solana transaction acceleration | **Trading Boost** |
| Quota, usage, service status | **Operational** |

## Ambiguity Resolution

If user intent is unclear, ask:
- Which chain? (Ethereum, Base, Polygon, Solana, etc.)
- Native balance or token balance? (`eth_getBalance` vs `zan_getTokenBalanceByOwner`)
- Raw result or structured analysis?
- Need code examples?

## Risk Disclaimer

All data provided is for reference only and does not constitute investment or trading advice. Blockchain data may have delays; always verify on-chain finality.
