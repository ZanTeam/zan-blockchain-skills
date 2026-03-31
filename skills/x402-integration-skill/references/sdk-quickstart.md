# x402 SDK Quick Start Reference

Step-by-step guide: install, authenticate, call RPC, manage credits.

## Prerequisites

| Item | Requirement |
|---|---|
| Node.js | >= 18 |
| Package manager | npm / pnpm / yarn |
| Wallet key | EVM `0x` hex private key **or** Solana Base58 secret key |

> Private keys are used locally for SIWE/SIWS signing + EIP-3009 payment signatures. They are never sent to the gateway.

---

## 1. Install

### From npm

```bash
npm install @zan_team/x402 viem
```

Solana optional (only for SVM auth/payment):

```bash
npm install @solana/web3.js @solana/spl-token bs58 tweetnacl
```

### From source

```bash
git clone <repo-url> zanx402-sdk
cd zanx402-sdk
npm install
npm run build        # ESM + CJS → dist/
```

Reference locally:

**npm link:**

```bash
cd zanx402-sdk && npm link
cd your-project  && npm link @zan_team/x402
```

**Local path dependency:**

```jsonc
// package.json
{
  "dependencies": {
    "@zan_team/x402": "file:../zanx402-sdk",
    "viem": ">=2.0.0"
  }
}
```

---

## 2. Environment Variables

```bash
# Gateway URL (default: https://x402.zan.top)
export GATEWAY_URL="https://x402.zan.top"

# EVM mode
export PRIVATE_KEY="0xYOUR_EVM_PRIVATE_KEY"

# OR Solana mode (auto-switches to SVM)
export SVM_PRIVATE_KEY="YOUR_BASE58_SOLANA_SECRET_KEY"

# Payment network (CAIP-2) — which chain settles USDC
export PAYMENT_NETWORK="eip155:84532"   # Base Sepolia (testnet)

# Credit bundle for auto-purchase
export BUNDLE="default"
```

### Payment Network Values

| CAIP-2 | Chain | Notes |
|---|---|---|
| `eip155:8453` | Base Mainnet | EVM production |
| `eip155:84532` | Base Sepolia | EVM testnet |
| `solana:5eykt4UsFv8P8NJdTREpY1vzqKqZKvdp` | Solana Mainnet | SVM production |
| `solana:EtWTRABZaYq6iMfeYKouRu166VU2xqa1` | Solana Devnet | SVM testnet |

---

## 3. Initialize Client

### EVM — Factory + Pre-auth (Recommended)

```typescript
import { createX402Client } from '@zan_team/x402';

const client = await createX402Client({
  gatewayUrl: 'https://x402.zan.top',
  privateKey: process.env.PRIVATE_KEY as `0x${string}`,
  autoPayment: true,
  preAuth: true,   // SIWE + JWT completed before return
});
```

`preAuth: true` completes authentication before the factory returns — ideal for long-running services and Agent processes.

### EVM — viem WalletClient

```typescript
import { createWalletClient, http } from 'viem';
import { privateKeyToAccount } from 'viem/accounts';
import { mainnet } from 'viem/chains';
import { X402Client } from '@zan_team/x402';

const wallet = createWalletClient({
  account: privateKeyToAccount('0x...'),
  chain: mainnet,
  transport: http(),
});

const client = new X402Client({
  gatewayUrl: 'https://x402.zan.top',
  wallet,
  autoPayment: true,
});
```

### Solana (SVM)

```typescript
const client = await createX402Client({
  gatewayUrl: 'https://x402.zan.top',
  svmPrivateKey: process.env.SVM_PRIVATE_KEY!,
  autoPayment: true,
  preAuth: true,
});
```

SDK auto-detects `chainType` from key format.

---

## 4. RPC Calls

### Single Call

```typescript
const block = await client.call('eth', 'mainnet', 'eth_blockNumber');
console.log('Latest block:', block.result);

const balance = await client.call('eth', 'mainnet', 'eth_getBalance', [
  '0x742d35Cc6634C0532925a3b844Bc9e7595f2bD18', 'latest',
]);
console.log('ETH balance (wei):', balance.result);
```

Path `/rpc/{ecosystem}/{network}` is built automatically; JWT injected. On insufficient credits with `autoPayment: true`, auto-purchases and retries.

### Batch RPC

```typescript
const results = await client.rpc.batch('eth', 'mainnet', [
  { method: 'eth_blockNumber' },
  { method: 'eth_gasPrice' },
  { method: 'net_version' },
]);
results.forEach((r) => console.log(r.result));
```

### Transparent Fetch

`client.fetch()` matches `globalThis.fetch` signature, auto-injects JWT and handles 402:

```typescript
const res = await client.fetch('/rpc/eth/mainnet', {
  method: 'POST',
  body: JSON.stringify({
    jsonrpc: '2.0', id: 1,
    method: 'eth_blockNumber', params: [],
  }),
});
console.log(await res.json());
```

---

## 5. Credits Management

```typescript
const { balance, tier } = await client.getBalance();
console.log(`Balance: ${balance}  Tier: ${tier}`);

const receipt = await client.purchaseCredits('default');
console.log(`Purchased ${receipt.creditsPurchased} credits  tx: ${receipt.txHash}`);

const usage = await client.getUsage({ limit: 10 });
usage.records.forEach((r) =>
  console.log(`${r.methodName}  cost=${r.creditCost}  ${r.latencyMs}ms`)
);
```

---

## 6. Discovery (No Auth Required)

```typescript
const health = await client.health();
console.log('Gateway status:', health.status);

const { networks } = await client.listNetworks();
console.log(`Available networks: ${networks.length}`);

const { bundles } = await client.listBundles();
bundles.forEach(b => console.log(`${b.name}: ${b.credits} credits @ ${b.price}`));

const capability = await client.getX402Capability();
console.log('x402 capability:', capability);
```

---

## 7. Full Bootstrap Example

```typescript
import { createX402Client } from '@zan_team/x402';

async function main() {
  const client = await createX402Client({
    gatewayUrl: process.env.GATEWAY_URL || 'https://x402.zan.top',
    privateKey: process.env.PRIVATE_KEY as `0x${string}`,
    autoPayment: true,
    preAuth: true,
  });

  // Discover
  const { networks } = await client.listNetworks();
  console.log(`Available networks: ${networks.length}`);

  // Check balance
  const { balance } = await client.getBalance();
  console.log(`Credits: ${balance}`);

  // Call RPC
  const block = await client.call('eth', 'mainnet', 'eth_blockNumber');
  console.log(`Latest block: ${block.result}`);

  // Batch call
  const results = await client.rpc.batch('eth', 'mainnet', [
    { method: 'eth_blockNumber' },
    { method: 'eth_gasPrice' },
  ]);
  results.forEach(r => console.log(r.result));
}

main().catch(console.error);
```

---

## Troubleshooting

| Symptom | Fix |
|---|---|
| `PRIVATE_KEY required` | Export `PRIVATE_KEY=0x...` or `SVM_PRIVATE_KEY=...` |
| `NetworkError: fetch failed` | Check gateway URL and network connectivity |
| `AuthenticationError` | Verify private key format (EVM: 0x hex, SVM: Base58) |
| `InsufficientCreditsError` | Call `purchaseCredits()` or enable `autoPayment: true` |
| `InsufficientFundsError` | Add USDC to your wallet on the payment network |
| `SessionExpiredError` | Call `client.authenticate()` to refresh JWT |

---

## Agent Notes

- **Key safety**: Always pass keys via env vars. Never hardcode or log them.
- **Testnet first**: Use `eip155:84532` (Base Sepolia) or `solana:EtWTRABZaYq6iMfeYKouRu166VU2xqa1` (Solana Devnet) for testing.
- **Auto-payment awareness**: Warn users that `autoPayment: true` will spend USDC automatically.
- **Pre-auth**: Use `preAuth: true` for Agent processes to avoid lazy-auth latency on first call.
