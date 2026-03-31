# ZAN Advanced Data API Reference

Technical reference for agents. Aggregates Token metadata, NFT holders, address activity, transaction traces, and simulation results without parsing raw blocks or event logs.

## Overview

- **Endpoint**: `POST https://api.zan.top/data/v1/{chain}/{network}/{apiKey}`
- **Protocol**: JSON-RPC 2.0
- **Auth**: API Key in URL path

**Example URLs:**
- Ethereum: `https://api.zan.top/data/v1/eth/mainnet/{apiKey}`
- BNB Chain: `https://api.zan.top/data/v1/bsc/mainnet/{apiKey}`
- Polygon: `https://api.zan.top/data/v1/polygon/mainnet/{apiKey}`

**Request format:**
```json
{
  "id": 1,
  "jsonrpc": "2.0",
  "method": "methodName",
  "params": []
}
```

---

## NFT API (8 methods)

### zan_getNFTMetadata
Query NFT contract metadata.

| Params | Type | Required | Description |
|--------|------|----------|-------------|
| 0 | string | yes | `contractAddress` |

**Response:** `{ address, decimal, ecosystem, symbol, name }`

---

### zan_getNFTsByOwner
List NFTs held by an address.

| Params | Type | Required | Description |
|--------|------|----------|-------------|
| 0 | string | yes | `ownerAddress` |

**Response:** Array of NFTs (contract, tokenId, metadata, etc.)

---

### zan_getNftIDs
List all token IDs for an NFT contract.

| Params | Type | Required | Description |
|--------|------|----------|-------------|
| 0 | string | yes | `contractAddress` |

**Response:** `string[]` (token IDs)

---

### zan_verifyNFTHolder
Check if address holds the NFT.

| Params | Type | Required | Description |
|--------|------|----------|-------------|
| 0 | string | yes | `contractAddress` |
| 1 | string | yes | `ownerAddress` |

**Response:** `boolean` or holder info

---

### zan_getNFTHolders
List NFT holders, sorted by holding proportion.

| Params | Type | Required | Description |
|--------|------|----------|-------------|
| 0 | string | yes | `contractAddress` |
| 1 | number | no | `topN` – limit results (omit for all) |

**Response:** `string[]` (holder addresses)

**curl:**
```bash
curl -X POST "https://api.zan.top/data/v1/eth/mainnet/YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "id": 1,
    "jsonrpc": "2.0",
    "method": "zan_getNFTHolders",
    "params": ["0xa01d803e2734c542d13a13772deced63cd6453bf", 100]
  }'
```

**TypeScript:**
```typescript
const response = await fetch(
  `https://api.zan.top/data/v1/eth/mainnet/${process.env.ZAN_API_KEY}`,
  {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({
      id: 1,
      jsonrpc: "2.0",
      method: "zan_getNFTHolders",
      params: ["0xa01d803e2734c542d13a13772deced63cd6453bf", 100],
    }),
  }
);
const { result } = await response.json();
console.log("Holders:", result); // ["0xc5293c93...", "0x753c930f...", ...]
```

---

### zan_getNftIDHolders
Get holder(s) of a specific token ID (ERC-1155 may have multiple).

| Params | Type | Required | Description |
|--------|------|----------|-------------|
| 0 | string | yes | `contractAddress` |
| 1 | string | yes | `tokenId` |

**Response:** Address or address array

---

### zan_getNftCollectionHolders
List holders of an NFT collection with stats.

| Params | Type | Required | Description |
|--------|------|----------|-------------|
| 0 | string | yes | `contractAddress` |

**Response:** Holder list and collection stats

---

### zan_getNftTransfers
Query NFT transfer history.

| Params | Type | Required | Description |
|--------|------|----------|-------------|
| 0 | string | yes | `contractAddress` |
| 1 | number | yes | `pageSize` (≤ 100) |
| 2 | number | yes | `pageKey` (1-based) |
| 3 | string | no | `fromAddress` filter |
| 4 | string | no | `toAddress` filter |

**Response:** `{ pageSize, pageKey, txs }` – each tx: `blockHash`, `blockNumber`, `from`, `to`, `tokenAddress`, `tokenId`, `tokenStandard`, `transactionHash`

---

## Token API (7 methods)

### zan_getTokenMetadata
Query Token metadata (name, symbol, decimal, ecosystem).

| Params | Type | Required | Description |
|--------|------|----------|-------------|
| 0 | string | yes | `contractAddress` |

**Response:** `{ address, decimal, ecosystem, symbol, name }`

**curl:**
```bash
curl -X POST "https://api.zan.top/data/v1/eth/mainnet/YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "id": 1,
    "jsonrpc": "2.0",
    "method": "zan_getTokenMetadata",
    "params": ["0xdAC17F958D2ee523a2206206994597C13D831ec7"]
  }'
```

**TypeScript:**
```typescript
const response = await fetch(
  `https://api.zan.top/data/v1/eth/mainnet/${process.env.ZAN_API_KEY}`,
  {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({
      id: 1,
      jsonrpc: "2.0",
      method: "zan_getTokenMetadata",
      params: ["0xdAC17F958D2ee523a2206206994597C13D831ec7"],
    }),
  }
);
const { result } = await response.json();
console.log("Token:", result.name, result.symbol, result.decimal);
```

---

### zan_getTokenBalanceByOwner
Get Token balance for an address.

| Params | Type | Required | Description |
|--------|------|----------|-------------|
| 0 | string | yes | `ownerAddress` |
| 1 | string | yes | `contractAddress` |

**Response:** `{ balance: string, decimal: number }` – balance is raw value

---

### zan_getTokensByOwner
List all Tokens held by an address.

| Params | Type | Required | Description |
|--------|------|----------|-------------|
| 0 | string | yes | `ownerAddress` |

**Response:** `[{ address, symbol, name, decimal, balance }]`

---

### zan_getTokenHoldersCount
Get total Token holder count.

| Params | Type | Required | Description |
|--------|------|----------|-------------|
| 0 | string | yes | `contractAddress` |

**Response:** `number`

---

### zan_getTokenHolders
List Token holders sorted by share.

| Params | Type | Required | Description |
|--------|------|----------|-------------|
| 0 | string | yes | `contractAddress` |
| 1 | number | yes | `topN` |

**Response:** `[{ address, balance?, percentage? }]` or address array (version-dependent)

---

### zan_getApprovalListByAddress
List Token approvals for an address.

| Params | Type | Required | Description |
|--------|------|----------|-------------|
| 0 | string | yes | `address` |

**Response:** `[{ tokenAddress, spender, amount }]`

---

### zan_getApprovalListByToken
List approvals for a Token.

| Params | Type | Required | Description |
|--------|------|----------|-------------|
| 0 | string | yes | `contractAddress` |

**Response:** `[{ owner, spender, amount }]`

---

## Debug API (5 methods)

### debug_traceTransaction
Trace execution of a transaction (call tree).

| Params | Type | Required | Description |
|--------|------|----------|-------------|
| 0 | string | yes | `txHash` |
| 1 | object | no | `options` |

**options:** `{ tracer?: "callTracer"|"prestateTracer"|"4byteTracer"|"noopTracer", tracerConfig?: { onlyTopCall?, withLog? }, timeout?: string }`

**Response:** Tree of calls – `type`, `from`, `to`, `value`, `gas`, `gasUsed`, `input`, `output`, `calls`, `error`/`revertReason`

**curl:**
```bash
curl -X POST "https://api.zan.top/data/v1/eth/mainnet/YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "id": 1,
    "jsonrpc": "2.0",
    "method": "debug_traceTransaction",
    "params": [
      "0xc738e0982fdf47f94357e7b3f1e3f3791c558ae957342b3e980686349f8c3d49",
      {
        "tracer": "callTracer",
        "tracerConfig": { "onlyTopCall": false, "withLog": true },
        "timeout": "10s"
      }
    ]
  }'
```

**TypeScript:**
```typescript
const response = await fetch(
  `https://api.zan.top/data/v1/eth/mainnet/${process.env.ZAN_API_KEY}`,
  {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({
      id: 1,
      jsonrpc: "2.0",
      method: "debug_traceTransaction",
      params: [
        "0xc738e0982fdf47f94357e7b3f1e3f3791c558ae957342b3e980686349f8c3d49",
        {
          tracer: "callTracer",
          tracerConfig: { onlyTopCall: false, withLog: true },
          timeout: "10s",
        },
      ],
    }),
  }
);
const { result } = await response.json();
// result: { type, from, to, value, gas, gasUsed, input, output, calls: [...] }
```

---

### debug_traceCall
Simulate and trace a call at a given block (no on-chain execution).

| Params | Type | Required | Description |
|--------|------|----------|-------------|
| 0 | object | yes | `callObject` (from, to, data, value, gas) |
| 1 | string | yes | `blockNumber` (e.g. `"latest"`, `"0x123"`) |
| 2 | object | no | `options` (same as debug_traceTransaction) |

**Response:** Same trace structure as debug_traceTransaction

---

### debug_traceBlockByHash
Trace all transactions in a block by hash.

| Params | Type | Required | Description |
|--------|------|----------|-------------|
| 0 | string | yes | `blockHash` |
| 1 | object | no | `options` |

**Response:** Trace array for block transactions

---

### debug_traceBlockByNumber
Trace all transactions in a block by number.

| Params | Type | Required | Description |
|--------|------|----------|-------------|
| 0 | string | yes | `blockNumber` (e.g. `"0xFD6B5"`, `"latest"`) |
| 1 | object | no | `options` |

**Response:** Trace array for block transactions

---

### debug_executionWitness
Get execution witness for a block.

| Params | Type | Required | Description |
|--------|------|----------|-------------|
| 0 | string | yes | `blockNumber` (hex) |

**Response:** Execution witness data

---

## Simulation API (2 methods)

### zan_simulateAssetChanges
Simulate asset changes (ERC20/NFT transfers, approvals).

| Params | Type | Required | Description |
|--------|------|----------|-------------|
| 0 | object | yes | `txObject` |

**txObject:** `{ from?, to, data?, value?, gas?, gasPrice?, maxPriorityFeePerGas?, maxFeePerGas? }`

**Response:** `{ changes: [...], gasUsed, error?, revertReason? }` – each change: `assetType`, `contractAddress`, `changeType`, `from`, `to`, `amount`, `rawAmount`, `decimals`, `name`, `symbol`, `tokenId`, `owner`, `spender`

---

### zan_simulateExecution
Simulate execution with decoded input/output.

| Params | Type | Required | Description |
|--------|------|----------|-------------|
| 0 | object | yes | `txObject` (same as above) |

**Response:** `{ from, to, type, value, gas, gasUsed, input, output, decoded: { methodName, inputs, outputs }, calls, logs, error?, revertReason? }`

---

## Error Codes

| Code | Description |
|------|-------------|
| `-32600` | Invalid Request – malformed request |
| `-32601` | Method not found – invalid method name |
| `-32602` | Invalid params – parameter format/validation error |
| `-32603` | Internal error – server error |
| `401` | Unauthorized – invalid API Key |
| `429` | Rate limit exceeded |
| `500` | Server error |

**JSON-RPC error response:**
```json
{
  "id": 1,
  "jsonrpc": "2.0",
  "error": { "code": -32602, "message": "Invalid params" }
}
```

---

## Agent Usage

- **NFT**: holders, metadata, transfers, verify holder → NFT API
- **Token**: metadata, balance, holders, approvals → Token API
- **Trace**: call tree, execution path, internal calls → Debug API
- **Simulate**: asset changes, decoded calls before execution → Simulation API
- **Address profile**: combine `zan_getTokensByOwner`, `zan_getNFTsByOwner`, `zan_getApprovalListByAddress`
- Addresses and hashes must be valid 0x format. `balance` is raw; use `decimal` to convert.
