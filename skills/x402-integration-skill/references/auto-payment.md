# x402 Auto-Payment Flow Reference

Technical reference for the x402 protocol's transparent payment mechanism. When `autoPayment: true`, the SDK automatically handles HTTP 402 challenges, signs on-chain payment authorizations, and retries the original request.

## Protocol Overview

x402 extends HTTP status 402 (Payment Required) for machine-to-machine micropayments. The gateway returns a structured 402 response when credits are insufficient, and the client resolves it by signing an on-chain payment authorization.

```
Client                   Gateway                  Facilitator
  │── POST /rpc/eth/main ─>│
  │<── 402 ────────────────│  "credits insufficient"
  │── POST /purchase/default ──────────────────────>│
  │<── 402 + PAYMENT-REQUIRED ─────────────────────│  "sign payment"
  │  [sign EIP-3009 / Solana SPL locally]
  │── POST /purchase/default (+ signature) ────────>│
  │                        │── POST /verify ───────>│
  │                        │<── valid, txHash ──────│
  │<── 200 + credits ──────│
  │── POST /rpc/eth/main ─>│  [auto-retry]
  │<── 200 + result ───────│
```

## Step-by-Step Flow

### Step 1: Initial RPC Request

Client sends a standard JSON-RPC request with JWT authorization:

```http
POST /rpc/eth/mainnet HTTP/1.1
Authorization: Bearer <jwt>
Content-Type: application/json

{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}
```

### Step 2: 402 Response

Gateway returns 402 with structured challenge:

```http
HTTP/1.1 402 Payment Required
Content-Type: application/json

{
  "error": "insufficient_credits",
  "required": 1,
  "balance": 0,
  "accepts": {
    "scheme": "exact",
    "network": "eip155:84532",
    "maxAmountRequired": "100000",
    "resource": "https://x402.zan.top/purchase/default",
    "payTo": "0x...",
    "extra": {
      "name": "USDC",
      "version": "2"
    }
  }
}
```

### Step 3: Payment Authorization

**EVM (EIP-3009 — TransferWithAuthorization):**

The SDK signs an EIP-3009 `TransferWithAuthorization` message locally. This authorizes the USDC contract to transfer tokens from the user's wallet to the facilitator — no separate approve transaction needed.

```typescript
const signature = await walletClient.signTypedData({
  domain: {
    name: 'USDC',
    version: '2',
    chainId: 84532,
    verifyingContract: USDC_CONTRACT,
  },
  types: {
    TransferWithAuthorization: [
      { name: 'from', type: 'address' },
      { name: 'to', type: 'address' },
      { name: 'value', type: 'uint256' },
      { name: 'validAfter', type: 'uint256' },
      { name: 'validBefore', type: 'uint256' },
      { name: 'nonce', type: 'bytes32' },
    ],
  },
  primaryType: 'TransferWithAuthorization',
  message: { from, to, value, validAfter, validBefore, nonce },
});
```

**Solana (SPL Token Transfer):**

The SDK constructs and signs an SPL Token transfer instruction for USDC on Solana.

### Step 4: Purchase with Signature

```http
POST /purchase/default HTTP/1.1
Authorization: Bearer <jwt>
Content-Type: application/json

{
  "payment": {
    "signature": "0x...",
    "from": "0x...",
    "to": "0x...",
    "value": "100000",
    "validAfter": "0",
    "validBefore": "115792...",
    "nonce": "0x..."
  }
}
```

### Step 5: Verification & Credits

The facilitator verifies the payment on-chain, then the gateway issues credits:

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "creditsPurchased": 100,
  "txHash": "0x...",
  "balance": 100
}
```

### Step 6: Auto-Retry

The SDK automatically retries the original RPC request with the refreshed credits.

---

## Configuration

| Config | Type | Default | Effect |
|---|---|---|---|
| `autoPayment` | boolean | `false` | Enable transparent 402 → purchase → retry |
| `defaultBundle` | `'default'` \| `'standard'` \| `'bulk'` | `'default'` | Bundle used for auto-purchase |
| `paymentNetwork` | string (CAIP-2) | auto | Override USDC settlement chain |

### Enabling Auto-Payment

```typescript
const client = await createX402Client({
  gatewayUrl: 'https://x402.zan.top',
  privateKey: process.env.PRIVATE_KEY as `0x${string}`,
  autoPayment: true,       // enable
  defaultBundle: 'default', // bundle to purchase
});
```

### Manual Payment

When `autoPayment: false` (default), handle 402 explicitly:

```typescript
import { InsufficientCreditsError } from '@zan_team/x402';

try {
  await client.call('eth', 'mainnet', 'eth_blockNumber');
} catch (err) {
  if (err instanceof InsufficientCreditsError) {
    console.log(`Need ${err.required} credits, have ${err.balance}`);
    const receipt = await client.purchaseCredits('default');
    console.log(`Bought ${receipt.creditsPurchased} credits`);
    // Retry manually
    const block = await client.call('eth', 'mainnet', 'eth_blockNumber');
  }
}
```

---

## Payment Errors

| Error | HTTP | Cause | Resolution |
|---|---|---|---|
| `InsufficientCreditsError` | 402 | Not enough credits for RPC call | Purchase credits or enable autoPayment |
| `InsufficientFundsError` | 402 | Wallet USDC balance too low | Add USDC on the payment network |
| `PaymentRejectedError` | 402 | Facilitator rejected the signature | Check key, network, USDC balance |

---

## Security Notes

- **Local signing only**: Private keys never leave the client. EIP-3009 / SPL signatures are computed locally.
- **Time-bounded**: EIP-3009 `validBefore` / `validAfter` fields limit the authorization window.
- **Single-use nonce**: Each payment uses a unique nonce to prevent replay.
- **USDC only**: Payments are in USDC stablecoin — no volatile token exposure.

---

## Agent Notes

- Always warn users before enabling `autoPayment: true` — it will spend USDC.
- Recommend testnet (`eip155:84532` or `solana:EtWTRABZaYq6iMfeYKouRu166VU2xqa1`) for first-time setup.
- If `InsufficientFundsError` occurs, guide user to fund their wallet with USDC on the correct network.
- The SDK handles the entire 402 flow internally — no manual HTTP interaction needed.
