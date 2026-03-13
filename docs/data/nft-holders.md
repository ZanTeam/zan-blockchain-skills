# NFT API（EVM 兼容）

## 概述

ZAN NFT API 提供 EVM 兼容链上的 NFT 数据查询能力，包括元数据、持有者列表、转账记录等。所有方法均通过 **JSON-RPC 2.0** 格式调用，使用 **POST** 方法。

## 统一端点

```
https://api.zan.top/data/v1/{chain}/{network}/{apiKey}
```

示例：
- Ethereum 主网：`https://api.zan.top/data/v1/eth/mainnet/{apiKey}`
- Base 主网：`https://api.zan.top/data/v1/base/mainnet/{apiKey}`
- Arbitrum 主网：`https://api.zan.top/data/v1/arbitrum/mainnet/{apiKey}`

## 通用请求格式

```json
{
  "id": 1,
  "jsonrpc": "2.0",
  "method": "方法名",
  "params": [...]
}
```

## 支持的鉴权方式

<!-- x402 支付方式正在开发中，后续将在此处补充 x402 相关内容 -->

| 鉴权方式 | 说明 |
|----------|------|
| **API Key** | 在 URL 路径中传入 `{apiKey}` |

---

## 1. zan_getNFTMetadata

查询 NFT 合约元数据。

### 参数

| 索引 | 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|------|
| 0 | `contractAddress` | string | 是 | NFT 合约地址 |

### 响应

| 字段 | 类型 | 说明 |
|------|------|------|
| `address` | string | 合约地址 |
| `decimal` | number | 精度（通常为 0 或 18） |
| `ecosystem` | string | 生态标识，如 `eth` |
| `symbol` | string | 代币符号 |
| `name` | string | 代币名称 |

### 示例

#### curl

```bash
curl -X POST "https://api.zan.top/data/v1/eth/mainnet/YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "id": 1,
    "jsonrpc": "2.0",
    "method": "zan_getNFTMetadata",
    "params": ["0xBC4CA0EdA7647A8aB7C2061c2E118A18a936f13D"]
  }'
```

#### TypeScript

```typescript
const endpoint = `https://api.zan.top/data/v1/eth/mainnet/${process.env.ZAN_API_KEY}`;

const response = await fetch(endpoint, {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({
    id: 1,
    jsonrpc: "2.0",
    method: "zan_getNFTMetadata",
    params: ["0xBC4CA0EdA7647A8aB7C2061c2E118A18a936f13D"],
  }),
});

const { result } = await response.json();
console.log("NFT 名称:", result.name);
console.log("NFT 符号:", result.symbol);
```

### 响应示例

```json
{
  "result": {
    "address": "0xb39221e6790ae8360b7e8c1c7221900fad9397f9",
    "decimal": 18,
    "ecosystem": "eth",
    "name": "Bored Ape Yacht Club",
    "symbol": "BAYC"
  },
  "id": 1,
  "jsonrpc": "2.0"
}
```

---

## 2. zan_getNFTsByOwner

查询指定地址持有的所有 NFT。

### 参数

| 索引 | 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|------|
| 0 | `ownerAddress` | string | 是 | 持有者地址 |

### 响应

返回该地址持有的 NFT 列表，包含合约地址、tokenId、元数据等信息。

### 示例

#### curl

```bash
curl -X POST "https://api.zan.top/data/v1/eth/mainnet/YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "id": 1,
    "jsonrpc": "2.0",
    "method": "zan_getNFTsByOwner",
    "params": ["0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb"]
  }'
```

#### TypeScript

```typescript
const response = await fetch(endpoint, {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({
    id: 1,
    jsonrpc: "2.0",
    method: "zan_getNFTsByOwner",
    params: ["0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb"],
  }),
});

const { result } = await response.json();
console.log("持有的 NFT:", result);
```

---

## 3. zan_getNftIDs

查询 NFT 合约下所有 Token ID。

### 参数

| 索引 | 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|------|
| 0 | `contractAddress` | string | 是 | NFT 合约地址 |

### 响应

返回 tokenId 数组。

### 示例

#### curl

```bash
curl -X POST "https://api.zan.top/data/v1/eth/mainnet/YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "id": 1,
    "jsonrpc": "2.0",
    "method": "zan_getNftIDs",
    "params": ["0xBC4CA0EdA7647A8aB7C2061c2E118A18a936f13D"]
  }'
```

#### TypeScript

```typescript
const { result } = await fetch(endpoint, {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({
    id: 1,
    jsonrpc: "2.0",
    method: "zan_getNftIDs",
    params: ["0xBC4CA0EdA7647A8aB7C2061c2E118A18a936f13D"],
  }),
}).then((r) => r.json());

console.log("Token ID 列表:", result);
```

---

## 4. zan_verifyNFTHolder

验证某地址是否持有指定 NFT。

### 参数

| 索引 | 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|------|
| 0 | `contractAddress` | string | 是 | NFT 合约地址 |
| 1 | `ownerAddress` | string | 是 | 待验证的持有者地址 |

### 响应

返回布尔值或持有信息，表示该地址是否持有该 NFT。

### 示例

#### curl

```bash
curl -X POST "https://api.zan.top/data/v1/eth/mainnet/YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "id": 1,
    "jsonrpc": "2.0",
    "method": "zan_verifyNFTHolder",
    "params": [
      "0xBC4CA0EdA7647A8aB7C2061c2E118A18a936f13D",
      "0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb"
    ]
  }'
```

#### TypeScript

```typescript
const { result } = await fetch(endpoint, {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({
    id: 1,
    jsonrpc: "2.0",
    method: "zan_verifyNFTHolder",
    params: [
      "0xBC4CA0EdA7647A8aB7C2061c2E118A18a936f13D",
      "0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb",
    ],
  }),
}).then((r) => r.json());

console.log("是否持有:", result);
```

---

## 5. zan_getNFTHolders

查询 NFT 持有者列表，按持有比例排序。

### 参数

| 索引 | 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|------|
| 0 | `contractAddress` | string | 是 | NFT 合约地址 |
| 1 | `topN` | number | 否 | 返回前 N 个持有者，不传则返回全部 |

### 响应

返回持有者地址数组，按持有比例从高到低排序。

### 示例

#### curl

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

#### TypeScript

```typescript
const contractAddress = "0xa01d803e2734c542d13a13772deced63cd6453bf";
const topN = 100; // 可选，限制返回数量

const response = await fetch(endpoint, {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({
    id: 1,
    jsonrpc: "2.0",
    method: "zan_getNFTHolders",
    params: topN ? [contractAddress, topN] : [contractAddress],
  }),
});

const { result } = await response.json();
console.log("持有者列表（按持有比例排序）:", result);
// result 为地址数组，如 ["0xc5293c93...", "0x753c930f...", ...]
```

### 响应示例

```json
{
  "result": [
    "0xc5293c9328e59bbe80c4d92ebbc3253e48d21397",
    "0x753c930f1a70cb6332d7c0ad6b45b33195f92110",
    "0xaf50456e3f4233ab99d88b86a91c4646c7fbdc2f",
    "0x197aea36dcecaaa7c4c94b44428c0d33e40cceab",
    "0xb72242db4ffc6f19517b810087c3b520ac322c09"
  ],
  "id": 1,
  "jsonrpc": "2.0"
}
```

---

## 6. zan_getNftIDHolders

查询特定 NFT Token ID 的持有者。

### 参数

| 索引 | 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|------|
| 0 | `contractAddress` | string | 是 | NFT 合约地址 |
| 1 | `tokenId` | string | 是 | Token ID |

### 响应

返回持有该 tokenId 的地址或地址列表（ERC-1155 可能有多持有者）。

### 示例

#### curl

```bash
curl -X POST "https://api.zan.top/data/v1/eth/mainnet/YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "id": 1,
    "jsonrpc": "2.0",
    "method": "zan_getNftIDHolders",
    "params": ["0xBC4CA0EdA7647A8aB7C2061c2E118A18a936f13D", "1234"]
  }'
```

#### TypeScript

```typescript
const { result } = await fetch(endpoint, {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({
    id: 1,
    jsonrpc: "2.0",
    method: "zan_getNftIDHolders",
    params: ["0xBC4CA0EdA7647A8aB7C2061c2E118A18a936f13D", "1234"],
  }),
}).then((r) => r.json());

console.log("Token #1234 持有者:", result);
```

---

## 7. zan_getNftCollectionHolders

查询 NFT Collection 持有者。

### 参数

| 索引 | 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|------|
| 0 | `contractAddress` | string | 是 | NFT 合约地址 |

### 响应

返回该 Collection 的持有者列表及相关统计信息。

### 示例

#### curl

```bash
curl -X POST "https://api.zan.top/data/v1/eth/mainnet/YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "id": 1,
    "jsonrpc": "2.0",
    "method": "zan_getNftCollectionHolders",
    "params": ["0xBC4CA0EdA7647A8aB7C2061c2E118A18a936f13D"]
  }'
```

#### TypeScript

```typescript
const { result } = await fetch(endpoint, {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({
    id: 1,
    jsonrpc: "2.0",
    method: "zan_getNftCollectionHolders",
    params: ["0xBC4CA0EdA7647A8aB7C2061c2E118A18a936f13D"],
  }),
}).then((r) => r.json());

console.log("Collection 持有者:", result);
```

---

## 8. zan_getNftTransfers

查询 NFT 转账记录。

### 参数

| 索引 | 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|------|
| 0 | `contractAddress` | string | 是 | NFT 合约地址 |
| 1 | `pageSize` | number | 是 | 每页条数，需 ≤ 100 |
| 2 | `pageKey` | number | 是 | 页码，从 1 开始 |
| 3 | `fromAddress` | string | 否 | 筛选转出地址 |
| 4 | `toAddress` | string | 否 | 筛选转入地址 |

### 响应

| 字段 | 类型 | 说明 |
|------|------|------|
| `pageSize` | number | 每页条数 |
| `pageKey` | number | 当前页码 |
| `txs` | array | 转账记录列表 |

每条 `txs` 记录包含：

| 字段 | 类型 | 说明 |
|------|------|------|
| `blockHash` | string | 区块哈希 |
| `blockNumber` | number | 区块号 |
| `from` | string | 转出地址 |
| `to` | string | 转入地址 |
| `tokenAddress` | string | NFT 合约地址 |
| `tokenId` | string | Token ID |
| `tokenStandard` | string | 标准类型，如 `ERC721`、`ERC1155` |
| `transactionHash` | string | 交易哈希 |

### 示例

#### curl

```bash
curl -X POST "https://api.zan.top/data/v1/eth/mainnet/YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "id": 1,
    "jsonrpc": "2.0",
    "method": "zan_getNftTransfers",
    "params": [
      "0x55b9a11c2e8351b4ffc7b11561148bfac9977855",
      10,
      1,
      "0x38150290c18d9b28c6d13b12bebf779c36f76cb1",
      "0x3d2068aeb969cccdc3b172028a120023e2fff59e"
    ]
  }'
```

#### TypeScript

```typescript
const contractAddress = "0x55b9a11c2e8351b4ffc7b11561148bfac9977855";
const pageSize = 10; // 必须 ≤ 100
const pageKey = 1;
const fromAddress = "0x38150290c18d9b28c6d13b12bebf779c36f76cb1"; // 可选
const toAddress = "0x3d2068aeb969cccdc3b172028a120023e2fff59e";   // 可选

const response = await fetch(endpoint, {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({
    id: 1,
    jsonrpc: "2.0",
    method: "zan_getNftTransfers",
    params: [contractAddress, pageSize, pageKey, fromAddress, toAddress],
  }),
});

const { result } = await response.json();
console.log("转账记录:", result.txs);
console.log("当前页:", result.pageKey, "每页:", result.pageSize);
```

### 响应示例

```json
{
  "result": {
    "pageSize": 10,
    "pageKey": 1,
    "txs": [
      {
        "blockHash": "0x7fa45b350f52f9d2935e1b53aa10af6f70f6769039c8ba799719f1234d3e77aa",
        "blockNumber": 1038693,
        "from": "0x38150290c18d9b28c6d13b12bebf779c36f76cb1",
        "to": "0x3d2068aeb969cccdc3b172028a120023e2fff59e",
        "tokenAddress": "0x55b9a11c2e8351b4ffc7b11561148bfac9977855",
        "tokenId": "1000000000",
        "tokenStandard": "ERC721",
        "transactionHash": "0xc738e0982fdf47f94357e7b3f1e3f3791c558ae957342b3e980686349f8c3d49"
      }
    ]
  },
  "id": 83,
  "jsonrpc": "2.0"
}
```

---

## 错误处理

<!-- x402 支付方式正在开发中，后续将在此处补充 x402 相关内容 -->

| 错误码 | 说明 |
|--------|------|
| `-32600` | Invalid Request - 请求格式错误 |
| `-32601` | Method not found - 方法名错误 |
| `-32602` | Invalid params - 参数格式错误 |
| `-32603` | Internal error - 服务端内部错误 |

---

## Agent 使用建议

- **触发条件**：用户询问 NFT 持有者、元数据、转账记录、空投快照、持有验证时，使用本 API。
- **方法选择**：
  - 获取合约基础信息 → `zan_getNFTMetadata`
  - 获取持有者列表（按比例排序）→ `zan_getNFTHolders`
  - 获取某地址持有的 NFT → `zan_getNFTsByOwner`
  - 获取转账历史 → `zan_getNftTransfers`
  - 验证是否持有 → `zan_verifyNFTHolder`
- **参数补全**：`contractAddress` 必填，需为 0x 开头的有效地址。若用户仅提供项目名称，需提示提供合约地址或通过其他方式解析。
