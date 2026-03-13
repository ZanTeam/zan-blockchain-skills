# ZAN Chain RPC API Reference

Consolidated reference for ZAN Chain RPC APIs. Use for on-chain state queries, real-time interactions, contract read calls, and gas estimation.

## Overview

Chain RPC provides raw on-chain state queries and real-time interaction. Use when you need:
- Account balance (native token)
- Block height and block details
- Transaction details and receipts
- Contract read-only calls (view/pure)
- Gas estimation

**Auth:** API Key (register at [ZAN Console](https://zan.top)).

## Endpoint Format

```
POST https://api.zan.top/node/v1/{chain}/{network}/{API_KEY}
```

Replace `{chain}/{network}` with target chain (e.g. `eth/mainnet`, `base/mainnet`, `arbitrum/mainnet`). See [Supported Chains](#supported-chains) below.

---

## API Methods

### eth_blockNumber

**Description:** Returns the latest block number the node has synced to, as a hex string.

**Params:** `[]` (empty array)

**Response:** `result` — hex block number, e.g. `"0x12a4b3c"`

**curl:**
```bash
curl -X POST https://api.zan.top/node/v1/eth/mainnet/YOUR_API_KEY \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "method": "eth_blockNumber",
    "params": [],
    "id": 1
  }'
```

**TypeScript:**
```typescript
const response = await fetch(`https://api.zan.top/node/v1/eth/mainnet/${API_KEY}`, {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({
    jsonrpc: "2.0",
    method: "eth_blockNumber",
    params: [],
    id: 1,
  }),
});
const data = await response.json();
const blockNumber = parseInt(data.result, 16);
```

---

### eth_getBalance

**Description:** Returns native token balance (ETH, BNB, etc.) for an address in wei. Does not include ERC20 or other token balances.

**Params:** `[address, blockTag]`
| Index | Param | Type | Description |
|-------|-------|------|-------------|
| 0 | address | string | 0x-prefixed 20-byte address |
| 1 | blockTag | string | `"latest"`, `"earliest"`, or hex block number |

**Response:** `result` — hex balance in wei, e.g. `"0x1bc16d674ec80000"` (1 ETH = 10^18 wei)

**curl:**
```bash
curl -X POST https://api.zan.top/node/v1/eth/mainnet/YOUR_API_KEY \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "method": "eth_getBalance",
    "params": ["0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb", "latest"],
    "id": 1
  }'
```

**TypeScript:**
```typescript
const response = await fetch(`https://api.zan.top/node/v1/eth/mainnet/${API_KEY}`, {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({
    jsonrpc: "2.0",
    method: "eth_getBalance",
    params: ["0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb", "latest"],
    id: 1,
  }),
});
const data = await response.json();
const balanceEth = Number(BigInt(data.result)) / 1e18;
```

---

### eth_getTransactionByHash

**Description:** Returns full transaction details by hash. Returns `null` if not yet mined or hash does not exist.

**Params:** `[transactionHash]`
| Index | Param | Type | Description |
|-------|-------|------|-------------|
| 0 | transactionHash | string | 0x-prefixed 32-byte tx hash |

**Response:** `result` — transaction object or `null`

| Field | Type | Description |
|-------|------|-------------|
| hash | string | Transaction hash |
| nonce | string | Sender nonce (hex) |
| blockHash | string \| null | Block hash, null if pending |
| blockNumber | string \| null | Block number (hex), null if pending |
| from | string | Sender address |
| to | string \| null | Recipient, null for contract creation |
| value | string | Amount in wei (hex) |
| gas | string | Gas limit (hex) |
| gasPrice | string | Gas price in wei (hex) |
| input | string | Calldata (0x-prefixed) |

**curl:**
```bash
curl -X POST https://api.zan.top/node/v1/eth/mainnet/YOUR_API_KEY \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "method": "eth_getTransactionByHash",
    "params": ["0x88df016429689c079f3b2f6ad39fa052532c56795b733da78a91ebe6a713944b"],
    "id": 1
  }'
```

**TypeScript:**
```typescript
const response = await fetch(`https://api.zan.top/node/v1/eth/mainnet/${API_KEY}`, {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({
    jsonrpc: "2.0",
    method: "eth_getTransactionByHash",
    params: [txHash],
    id: 1,
  }),
});
const data = await response.json();
const tx = data.result; // null if not found
```

---

### eth_call

**Description:** Executes a contract read-only call (view/pure) at a given block. No gas consumed, no state change. Returns hex-encoded result; decode with ABI.

**Params:** `[callObject, blockTag]`
| Index | Param | Type | Description |
|-------|-------|------|-------------|
| 0 | callObject | object | Call params |
| 1 | blockTag | string | `"latest"`, `"earliest"`, or hex block number |

**callObject:**
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| to | string | yes | Contract address |
| data | string | no | Calldata (selector + ABI-encoded args) |
| from | string | no | Simulated caller (some contracts need it) |
| value | string | no | Simulated value in wei (hex) |
| gas | string | no | Gas limit (hex) |

**Response:** `result` — hex-encoded return value (e.g. 32-byte for uint256)

**curl (ERC20 balanceOf):**
```bash
curl -X POST https://api.zan.top/node/v1/eth/mainnet/YOUR_API_KEY \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "method": "eth_call",
    "params": [
      {
        "to": "0xdAC17F958D2ee523a2206206994597C13D831ec7",
        "data": "0x70a08231000000000000000000000000742d35Cc6634C0532925a3b844Bc9e7595f0bEb"
      },
      "latest"
    ],
    "id": 1
  }'
```

**TypeScript:**
```typescript
import { ethers } from "ethers";

const iface = new ethers.Interface(["function balanceOf(address) view returns (uint256)"]);
const data = iface.encodeFunctionData("balanceOf", [holderAddress]);

const response = await fetch(`https://api.zan.top/node/v1/eth/mainnet/${API_KEY}`, {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({
    jsonrpc: "2.0",
    method: "eth_call",
    params: [{ to: contractAddress, data }, "latest"],
    id: 1,
  }),
});
const dataRes = await response.json();
const balance = BigInt(dataRes.result);
```

---

### eth_estimateGas

**Description:** Estimates gas required for a transaction. Simulates execution at given block state. May return error if tx would revert.

**Params:** `[transactionObject]`
| Index | Param | Type | Description |
|-------|-------|------|-------------|
| 0 | transactionObject | object | Simulated transaction |

**transactionObject:**
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| from | string | yes | Sender address |
| to | string | no | Recipient (omit for contract creation) |
| value | string | no | Amount in wei (hex) |
| data | string | no | Calldata |
| gas | string | no | Gas limit (hex) |
| gasPrice | string | no | Gas price (hex) |

**Response:** `result` — estimated gas in hex, e.g. `"0x5208"` (21000 for simple transfer)

**curl:**
```bash
curl -X POST https://api.zan.top/node/v1/eth/mainnet/YOUR_API_KEY \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "method": "eth_estimateGas",
    "params": [
      {
        "from": "0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb",
        "to": "0xd46e8dd67c5d32be8058bb8eb970870f07244567",
        "value": "0x9184e72a000"
      }
    ],
    "id": 1
  }'
```

**TypeScript:**
```typescript
const response = await fetch(`https://api.zan.top/node/v1/eth/mainnet/${API_KEY}`, {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({
    jsonrpc: "2.0",
    method: "eth_estimateGas",
    params: [{ from, to, value }],
    id: 1,
  }),
});
const data = await response.json();
const gasEstimate = data.error ? null : parseInt(data.result, 16);
```

---

## Error Codes

| Code | Message | Description |
|------|---------|-------------|
| -32600 | Invalid Request | Malformed request |
| -32601 | Method not found | Invalid method name |
| -32602 | Invalid params | Invalid parameter format |
| -32603 | Internal error | Server internal error |
| -32000 | execution reverted | Contract execution reverted (e.g. eth_estimateGas) |

---

## Supported Chains

| Chain | Endpoint path |
|-------|---------------|
| Ethereum Mainnet | `eth/mainnet` |
| Base Mainnet | `base/mainnet` |
| Arbitrum One | `arbitrum/mainnet` |
| Polygon | `polygon/mainnet` |
| BNB Smart Chain | `bsc/mainnet` |
| Optimism | `optimism/mainnet` |

ZAN supports 20+ chains. Use the `{chain}/{network}` path from your project config in the [ZAN Console](https://zan.top).

---

## Agent Notes

- **Native vs token:** `eth_getBalance` = native only. Use `eth_call` + `balanceOf` for ERC20.
- **Hex values:** Block numbers, balances, gas, and values are hex. Convert with `parseInt(hex, 16)` or `BigInt(hex)`.
- **wei → ETH:** Divide by `1e18`. Token decimals vary (USDT/USDC often 6).
- **eth_call data:** Encode with `ethers.encodeFunctionData` or equivalent.
- **eth_estimateGas:** If it errors, the tx would likely revert. Add 10–20% buffer when setting gas limit.
