# Token API

## 概述

ZAN Token API 提供 ERC-20 等标准 Token 的元数据、余额、持有者、授权列表等链上数据查询能力。所有方法均通过 **POST** 请求，使用 **JSON-RPC 2.0** 格式。

**统一端点**：`https://api.zan.top/data/v1/{chain}/{network}/{apiKey}`

**鉴权方式**：API Key（URL 路径）

<!-- x402 支付方式正在开发中，后续将在此处补充 x402 相关内容 -->

---

## 1. zan_getTokenMetadata

查询 Token 元数据（name, symbol, decimal, ecosystem）。

### 参数

| 参数 | 类型 | 说明 |
|------|------|------|
| `contractAddress` | string | Token 合约地址 |

`params`: `[contractAddress]`

### 响应 Schema

```json
{
  "id": 1,
  "jsonrpc": "2.0",
  "result": {
    "address": "string",
    "decimal": 18,
    "ecosystem": "eth",
    "symbol": "string",
    "name": "string"
  }
}
```

| 字段 | 类型 | 说明 |
|------|------|------|
| `address` | string | 合约地址 |
| `decimal` | number | 精度（小数位数） |
| `ecosystem` | string | 生态标识 |
| `symbol` | string | Token 符号 |
| `name` | string | Token 名称 |

### curl 示例

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

### TypeScript 示例

```typescript
const response = await fetch(
  'https://api.zan.top/data/v1/eth/mainnet/YOUR_API_KEY',
  {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      id: 1,
      jsonrpc: '2.0',
      method: 'zan_getTokenMetadata',
      params: ['0xdAC17F958D2ee523a2206206994597C13D831ec7'],
    }),
  }
);
const { result } = await response.json();
console.log('Token 元数据:', result); // { address, decimal, ecosystem, symbol, name }
```

---

## 2. zan_getTokenBalanceByOwner

查询指定地址的 Token 余额。

### 参数

| 参数 | 类型 | 说明 |
|------|------|------|
| `ownerAddress` | string | 持有者地址 |
| `contractAddress` | string | Token 合约地址 |

`params`: `[ownerAddress, contractAddress]`

### 响应 Schema

```json
{
  "id": 1,
  "jsonrpc": "2.0",
  "result": {
    "balance": "string",
    "decimal": 18
  }
}
```

| 字段 | 类型 | 说明 |
|------|------|------|
| `balance` | string | 余额（原始值，需根据 decimal 换算） |
| `decimal` | number | 精度 |

### curl 示例

```bash
curl -X POST "https://api.zan.top/data/v1/eth/mainnet/YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "id": 1,
    "jsonrpc": "2.0",
    "method": "zan_getTokenBalanceByOwner",
    "params": ["0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb", "0xdAC17F958D2ee523a2206206994597C13D831ec7"]
  }'
```

### TypeScript 示例

```typescript
const response = await fetch(
  'https://api.zan.top/data/v1/eth/mainnet/YOUR_API_KEY',
  {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      id: 1,
      jsonrpc: '2.0',
      method: 'zan_getTokenBalanceByOwner',
      params: [
        '0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb',
        '0xdAC17F958D2ee523a2206206994597C13D831ec7',
      ],
    }),
  }
);
const { result } = await response.json();
const humanReadable = BigInt(result.balance) / BigInt(10 ** result.decimal);
console.log('余额:', humanReadable.toString());
```

---

## 3. zan_getTokensByOwner

查询指定地址持有的所有 Token。

### 参数

| 参数 | 类型 | 说明 |
|------|------|------|
| `ownerAddress` | string | 持有者地址 |

`params`: `[ownerAddress]`

### 响应 Schema

```json
{
  "id": 1,
  "jsonrpc": "2.0",
  "result": [
    {
      "address": "string",
      "symbol": "string",
      "name": "string",
      "decimal": 18,
      "balance": "string"
    }
  ]
}
```

### curl 示例

```bash
curl -X POST "https://api.zan.top/data/v1/eth/mainnet/YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "id": 1,
    "jsonrpc": "2.0",
    "method": "zan_getTokensByOwner",
    "params": ["0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb"]
  }'
```

### TypeScript 示例

```typescript
const response = await fetch(
  'https://api.zan.top/data/v1/eth/mainnet/YOUR_API_KEY',
  {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      id: 1,
      jsonrpc: '2.0',
      method: 'zan_getTokensByOwner',
      params: ['0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb'],
    }),
  }
);
const { result } = await response.json();
console.log('持有 Token 列表:', result);
```

---

## 4. zan_getTokenHoldersCount

查询 Token 持有者数量。

### 参数

| 参数 | 类型 | 说明 |
|------|------|------|
| `contractAddress` | string | Token 合约地址 |

`params`: `[contractAddress]`

### 响应 Schema

```json
{
  "id": 1,
  "jsonrpc": "2.0",
  "result": 12345
}
```

`result` 为 number，表示持有者总数。

### curl 示例

```bash
curl -X POST "https://api.zan.top/data/v1/eth/mainnet/YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "id": 1,
    "jsonrpc": "2.0",
    "method": "zan_getTokenHoldersCount",
    "params": ["0xdAC17F958D2ee523a2206206994597C13D831ec7"]
  }'
```

### TypeScript 示例

```typescript
const response = await fetch(
  'https://api.zan.top/data/v1/eth/mainnet/YOUR_API_KEY',
  {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      id: 1,
      jsonrpc: '2.0',
      method: 'zan_getTokenHoldersCount',
      params: ['0xdAC17F958D2ee523a2206206994597C13D831ec7'],
    }),
  }
);
const { result } = await response.json();
console.log('持有者数量:', result);
```

---

## 5. zan_getTokenHolders

查询 Token 持有者列表（按持有比例排序）。

### 参数

| 参数 | 类型 | 说明 |
|------|------|------|
| `contractAddress` | string | Token 合约地址 |
| `topN` | number | 返回前 N 名持有者 |

`params`: `[contractAddress, topN]`

### 响应 Schema

```json
{
  "id": 1,
  "jsonrpc": "2.0",
  "result": [
    {
      "address": "string",
      "balance": "string",
      "percentage": "string"
    }
  ]
}
```

| 字段 | 类型 | 说明 |
|------|------|------|
| `address` | string | 持有者地址 |
| `balance` | string | 持有数量（原始值） |
| `percentage` | string | 持有占比（可选） |

> 实际返回结构可能因 API 版本略有差异，以官方文档为准。部分版本可能直接返回地址数组。

### curl 示例

```bash
curl -X POST "https://api.zan.top/data/v1/eth/mainnet/YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "id": 1,
    "jsonrpc": "2.0",
    "method": "zan_getTokenHolders",
    "params": ["0xdAC17F958D2ee523a2206206994597C13D831ec7", 100]
  }'
```

### TypeScript 示例

```typescript
const response = await fetch(
  'https://api.zan.top/data/v1/eth/mainnet/YOUR_API_KEY',
  {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      id: 1,
      jsonrpc: '2.0',
      method: 'zan_getTokenHolders',
      params: ['0xdAC17F958D2ee523a2206206994597C13D831ec7', 100],
    }),
  }
);
const { result } = await response.json();
console.log('Top 100 持有者:', result);
```

---

## 6. zan_getApprovalListByAddress

查询指定地址的 Token 授权列表。

### 参数

| 参数 | 类型 | 说明 |
|------|------|------|
| `address` | string | 待查询地址 |

`params`: `[address]`

### 响应 Schema

```json
{
  "id": 1,
  "jsonrpc": "2.0",
  "result": [
    {
      "tokenAddress": "string",
      "spender": "string",
      "amount": "string"
    }
  ]
}
```

### curl 示例

```bash
curl -X POST "https://api.zan.top/data/v1/eth/mainnet/YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "id": 1,
    "jsonrpc": "2.0",
    "method": "zan_getApprovalListByAddress",
    "params": ["0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb"]
  }'
```

### TypeScript 示例

```typescript
const response = await fetch(
  'https://api.zan.top/data/v1/eth/mainnet/YOUR_API_KEY',
  {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      id: 1,
      jsonrpc: '2.0',
      method: 'zan_getApprovalListByAddress',
      params: ['0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb'],
    }),
  }
);
const { result } = await response.json();
console.log('授权列表:', result);
```

---

## 7. zan_getApprovalListByToken

查询指定 Token 的授权列表。

### 参数

| 参数 | 类型 | 说明 |
|------|------|------|
| `contractAddress` | string | Token 合约地址 |

`params`: `[contractAddress]`

### 响应 Schema

```json
{
  "id": 1,
  "jsonrpc": "2.0",
  "result": [
    {
      "owner": "string",
      "spender": "string",
      "amount": "string"
    }
  ]
}
```

### curl 示例

```bash
curl -X POST "https://api.zan.top/data/v1/eth/mainnet/YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "id": 1,
    "jsonrpc": "2.0",
    "method": "zan_getApprovalListByToken",
    "params": ["0xdAC17F958D2ee523a2206206994597C13D831ec7"]
  }'
```

### TypeScript 示例

```typescript
const response = await fetch(
  'https://api.zan.top/data/v1/eth/mainnet/YOUR_API_KEY',
  {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      id: 1,
      jsonrpc: '2.0',
      method: 'zan_getApprovalListByToken',
      params: ['0xdAC17F958D2ee523a2206206994597C13D831ec7'],
    }),
  }
);
const { result } = await response.json();
console.log('Token 授权列表:', result);
```

---

## 综合示例：批量查询 Token 元数据与持有者

以下示例展示如何组合 `zan_getTokenMetadata` 与 `zan_getTokenHolders` 完成常见分析任务。

### curl 批量调用

```bash
# 1. 获取 Token 元数据
curl -X POST "https://api.zan.top/data/v1/eth/mainnet/YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "id": 1,
    "jsonrpc": "2.0",
    "method": "zan_getTokenMetadata",
    "params": ["0xdAC17F958D2ee523a2206206994597C13D831ec7"]
  }'

# 2. 获取 Top 10 持有者
curl -X POST "https://api.zan.top/data/v1/eth/mainnet/YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "id": 2,
    "jsonrpc": "2.0",
    "method": "zan_getTokenHolders",
    "params": ["0xdAC17F958D2ee523a2206206994597C13D831ec7", 10]
  }'
```

### TypeScript 综合示例

```typescript
const API_BASE = 'https://api.zan.top/data/v1/eth/mainnet/YOUR_API_KEY';

async function zanRequest<T>(method: string, params: unknown[]): Promise<T> {
  const res = await fetch(API_BASE, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      id: 1,
      jsonrpc: '2.0',
      method,
      params,
    }),
  });
  const json = await res.json();
  if (json.error) throw new Error(json.error.message);
  return json.result;
}

async function analyzeToken(contractAddress: string) {
  // 1. 获取元数据
  const metadata = await zanRequest<{
    address: string;
    decimal: number;
    ecosystem: string;
    symbol: string;
    name: string;
  }>('zan_getTokenMetadata', [contractAddress]);

  console.log(`Token: ${metadata.name} (${metadata.symbol})`);
  console.log(`精度: ${metadata.decimal}`);

  // 2. 获取 Top 10 持有者
  const holders = await zanRequest<Array<{ address: string; balance?: string }>>(
    'zan_getTokenHolders',
    [contractAddress, 10]
  );

  console.log('Top 10 持有者:');
  holders.forEach((h, i) => {
    const balance = h.balance
      ? (Number(h.balance) / 10 ** metadata.decimal).toLocaleString()
      : 'N/A';
    console.log(`  ${i + 1}. ${h.address} - ${balance}`);
  });

  return { metadata, holders };
}

analyzeToken('0xdAC17F958D2ee523a2206206994597C13D831ec7');
```

---

## 错误处理

| 错误码 | 说明 |
|--------|------|
| `400` | 参数错误（如地址格式无效） |
| `401` | 鉴权失败（API Key 无效） |
| `429` | 请求频率超限 |
| `500` | 服务端错误 |

JSON-RPC 错误响应示例：

```json
{
  "id": 1,
  "jsonrpc": "2.0",
  "error": {
    "code": -32602,
    "message": "Invalid params"
  }
}
```

---

## Notes for Agents

- **触发条件**：用户询问 Token 元数据、某地址的 Token 余额、持有者列表、授权列表时，使用 Token API。
- **推荐鉴权**：使用 API Key 进行鉴权。
- **参数补全**：`contractAddress`、`ownerAddress` 等需为有效的 0x 格式地址。`balance` 为原始值，需结合 `decimal` 换算为可读数量。
- **错误处理**：若地址格式无效返回错误，可提示用户检查地址格式。
