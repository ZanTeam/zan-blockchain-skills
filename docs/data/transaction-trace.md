# Debug API & Simulation API

## 概述

ZAN 提供 **Debug API** 与 **Simulation API**，用于追踪交易执行过程、模拟交易并分析资产变化。所有方法均通过 **JSON-RPC 2.0** 格式调用，使用 **POST** 方法。

## 统一端点

```
https://api.zan.top/data/v1/{chain}/{network}/{apiKey}
```

示例：
- Ethereum 主网：`https://api.zan.top/data/v1/eth/mainnet/{apiKey}`
- Base 主网：`https://api.zan.top/data/v1/base/mainnet/{apiKey}`

## 通用请求格式

```json
{
  "id": 1,
  "jsonrpc": "2.0",
  "method": "方法名",
  "params": [...]
}
```

---

# Debug API

Debug API 用于追踪已上链交易的执行过程，或模拟调用并获取执行轨迹。

---

## 1. debug_traceTransaction

追踪交易执行过程，返回内部调用的树形结构。

### 参数

| 索引 | 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|------|
| 0 | `txHash` | string | 是 | 交易哈希 |
| 1 | `options` | object | 否 | 追踪配置 |

### options 字段

| 字段 | 类型 | 说明 |
|------|------|------|
| `tracer` | string | 追踪器类型：`callTracer`、`prestateTracer`、`4byteTracer`、`noopTracer` |
| `tracerConfig` | object | 追踪器配置，如 `{ "onlyTopCall": true, "withLog": true }` |
| `timeout` | string | 超时时间，默认 `"5s"` |

### callTracer 配置

| 字段 | 类型 | 说明 |
|------|------|------|
| `onlyTopCall` | boolean | 为 true 时仅追踪主调用，不包含子调用 |
| `withLog` | boolean | 为 true 时包含日志 |

### 响应

返回树形调用结构，每个节点包含：
- `type`: 调用类型（CALL、DELEGATECALL、STATICCALL、CREATE）
- `from`: 调用方地址
- `to`: 被调用方地址
- `value`: 转账金额（wei）
- `gas`: 分配的 gas
- `gasUsed`: 实际消耗的 gas
- `input`: 调用输入（hex）
- `output`: 调用输出（hex）
- `calls`: 嵌套的内部调用
- `error` / `revertReason`: 执行失败时的错误信息

### 示例

#### curl

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
        "tracerConfig": {
          "onlyTopCall": false,
          "withLog": true
        },
        "timeout": "10s"
      }
    ]
  }'
```

#### TypeScript

```typescript
const txHash = "0xc738e0982fdf47f94357e7b3f1e3f3791c558ae957342b3e980686349f8c3d49";

const response = await fetch(endpoint, {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({
    id: 1,
    jsonrpc: "2.0",
    method: "debug_traceTransaction",
    params: [
      txHash,
      {
        tracer: "callTracer",
        tracerConfig: { onlyTopCall: false, withLog: true },
        timeout: "10s",
      },
    ],
  }),
});

const { result } = await response.json();

// 递归遍历调用树
function printTrace(trace: any, depth = 0) {
  const indent = "  ".repeat(depth);
  console.log(`${indent}${trace.type} ${trace.from} -> ${trace.to} value=${trace.value}`);
  trace.calls?.forEach((c: any) => printTrace(c, depth + 1));
}
printTrace(result);
```

### 响应示例

```json
{
  "result": {
    "type": "CALL",
    "from": "0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb",
    "to": "0x7a250d5630B4cF539739dF2C5dAcb4c659F2488D",
    "value": "0x0de0b6b3a7640000",
    "gas": "0x30d40",
    "gasUsed": "0x24a2c",
    "input": "0x38ed1739...",
    "output": "0x0000...",
    "calls": [
      {
        "type": "CALL",
        "from": "0x7a250d5630B4cF539739dF2C5dAcb4c659F2488D",
        "to": "0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2",
        "value": "0x0",
        "gas": "0xc350",
        "gasUsed": "0xb429",
        "input": "0xa9059cbb...",
        "output": "0x0000...",
        "calls": []
      }
    ]
  },
  "id": 1,
  "jsonrpc": "2.0"
}
```

---

## 2. debug_traceCall

模拟调用并追踪执行过程，在指定区块状态下执行，不上链。

### 参数

| 索引 | 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|------|
| 0 | `callObject` | object | 是 | 调用对象，同 eth_call |
| 1 | `blockNumber` | string | 是 | 区块标签，如 `"latest"`、`"0x123"` |
| 2 | `options` | object | 否 | 追踪配置，同 debug_traceTransaction |

### callObject 字段

| 字段 | 类型 | 说明 |
|------|------|------|
| `from` | string | 调用方地址 |
| `to` | string | 合约地址 |
| `data` | string | 调用数据（hex） |
| `value` | string | 转账金额（hex） |
| `gas` | string | Gas 上限（hex） |

### 示例

#### curl

```bash
curl -X POST "https://api.zan.top/data/v1/eth/mainnet/YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "id": 1,
    "jsonrpc": "2.0",
    "method": "debug_traceCall",
    "params": [
      {
        "from": "0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb",
        "to": "0xdAC17F958D2ee523a2206206994597C13D831ec7",
        "data": "0xa9059cbb0000000000000000000000005964456c99bee03b64d688e690992830d2de84d9000000000000000000000000000000000000000000000000000000058b2fc480",
        "value": "0x0"
      },
      "latest",
      { "tracer": "callTracer" }
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
    method: "debug_traceCall",
    params: [
      {
        from: "0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb",
        to: "0xdAC17F958D2ee523a2206206994597C13D831ec7",
        data: "0xa9059cbb...",
        value: "0x0",
      },
      "latest",
      { tracer: "callTracer" },
    ],
  }),
}).then((r) => r.json());
```

---

## 3. debug_traceBlockByHash

追踪指定区块中所有交易的执行过程。

### 参数

| 索引 | 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|------|
| 0 | `blockHash` | string | 是 | 区块哈希 |
| 1 | `options` | object | 否 | 追踪配置 |

### 示例

#### curl

```bash
curl -X POST "https://api.zan.top/data/v1/eth/mainnet/YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "id": 1,
    "jsonrpc": "2.0",
    "method": "debug_traceBlockByHash",
    "params": [
      "0x7fa45b350f52f9d2935e1b53aa10af6f70f6769039c8ba799719f1234d3e77aa",
      { "tracer": "callTracer" }
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
    method: "debug_traceBlockByHash",
    params: ["0x7fa45b35...", { tracer: "callTracer" }],
  }),
}).then((r) => r.json());
```

---

## 4. debug_traceBlockByNumber

按区块号追踪该区块中所有交易的执行过程。

### 参数

| 索引 | 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|------|
| 0 | `blockNumber` | string | 是 | 区块号或标签，如 `"0x123"`、`"latest"` |
| 1 | `options` | object | 否 | 追踪配置 |

### 示例

#### curl

```bash
curl -X POST "https://api.zan.top/data/v1/eth/mainnet/YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "id": 1,
    "jsonrpc": "2.0",
    "method": "debug_traceBlockByNumber",
    "params": ["0xFD6B5", { "tracer": "callTracer" }]
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
    method: "debug_traceBlockByNumber",
    params: ["0xFD6B5", { tracer: "callTracer" }],
  }),
}).then((r) => r.json());
```

---

## 5. debug_executionWitness

获取执行见证数据，用于证明某区块的执行状态。

### 参数

| 索引 | 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|------|
| 0 | `blockNumber` | string | 是 | 区块号（十六进制） |

### 响应

返回该区块的执行见证数据。

### 示例

#### curl

```bash
curl -X POST "https://api.zan.top/data/v1/eth/mainnet/YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "id": 1,
    "jsonrpc": "2.0",
    "method": "debug_executionWitness",
    "params": ["0xFD6B5"]
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
    method: "debug_executionWitness",
    params: ["0xFD6B5"],
  }),
}).then((r) => r.json());
```

---

# Simulation API

Simulation API 用于在不上链的情况下模拟交易执行，分析资产变化与调用轨迹。

---

## 6. zan_simulateAssetChanges

模拟交易的资产变化，返回人类可读的资产变更列表（ERC20 转账、NFT 转移、授权等）。

### 参数

| 索引 | 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|------|
| 0 | `txObject` | object | 是 | 交易调用对象 |

### txObject 字段

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `from` | string | 否 | 调用方地址 |
| `to` | string | 是 | 目标合约地址 |
| `data` | string | 否 | 调用数据（hex） |
| `value` | string | 否 | 转账金额（hex），默认 `"0x0"` |
| `gas` | string | 否 | Gas 上限（hex） |
| `gasPrice` | string | 否 | Gas 价格（hex） |
| `maxPriorityFeePerGas` | string | 否 | EIP-1559 优先费（hex） |
| `maxFeePerGas` | string | 否 | EIP-1559 最大费（hex） |

### 响应

| 字段 | 类型 | 说明 |
|------|------|------|
| `changes` | array | 资产变更列表 |
| `gasUsed` | string | 消耗的 gas（hex） |
| `error` | string | 执行失败时的错误信息 |
| `revertReason` | string | 合约 revert 原因 |

### changes 每项字段

| 字段 | 类型 | 说明 |
|------|------|------|
| `assetType` | string | 资产类型：`NATIVE`、`ERC20`、`ERC721`、`ERC1155` |
| `contractAddress` | string | 合约地址（非 NATIVE 时） |
| `changeType` | string | 变更类型：`TRANSFER`、`APPROVE`、`APPROVE_ALL`、`REVOKE_APPROVAL_ALL` |
| `from` | string | 转出地址（TRANSFER 时） |
| `to` | string | 转入地址（TRANSFER 时） |
| `amount` | number | 数量（按 decimals 换算后） |
| `rawAmount` | string | 原始数量 |
| `decimals` | number | 精度（可选） |
| `name` | string | 代币名称（可选） |
| `symbol` | string | 代币符号（可选） |
| `tokenId` | string | NFT Token ID（ERC721/1155 时） |
| `owner` | string | 授权方（APPROVE 时） |
| `spender` | string | 被授权方（APPROVE 时） |

### 示例

#### curl

```bash
curl -X POST "https://api.zan.top/data/v1/eth/mainnet/YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "id": 1,
    "jsonrpc": "2.0",
    "method": "zan_simulateAssetChanges",
    "params": [
      {
        "from": "0xf977814e90da44bfa03b6295a0616a897441acec",
        "to": "0xdAC17F958D2ee523a2206206994597C13D831ec7",
        "data": "0xa9059cbb0000000000000000000000005964456c99bee03b64d688e690992830d2de84d9000000000000000000000000000000000000000000000000000000058b2fc480",
        "value": "0x0"
      }
    ]
  }'
```

#### TypeScript

```typescript
import { ethers } from "ethers";

// 模拟 USDT transfer 调用
const iface = new ethers.Interface([
  "function transfer(address to, uint256 amount) returns (bool)",
]);
const data = iface.encodeFunctionData("transfer", [
  "0x5964456c99bee03b64d688e690992830d2de84d9",
  ethers.parseUnits("23810", 6), // USDT 6 位小数
]);

const response = await fetch(endpoint, {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({
    id: 1,
    jsonrpc: "2.0",
    method: "zan_simulateAssetChanges",
    params: [
      {
        from: "0xf977814e90da44bfa03b6295a0616a897441acec",
        to: "0xdAC17F958D2ee523a2206206994597C13D831ec7",
        data,
        value: "0x0",
      },
    ],
  }),
});

const { result } = await response.json();

console.log("Gas 消耗:", result.gasUsed);
result.changes?.forEach((c: any) => {
  if (c.changeType === "TRANSFER") {
    console.log(`${c.symbol}: ${c.amount} 从 ${c.from} 转至 ${c.to}`);
  }
});
```

### 响应示例

```json
{
  "result": {
    "changes": [
      {
        "assetType": "ERC20",
        "contractAddress": "0xdac17f958d2ee523a2206206994597c13d831ec7",
        "decimals": 6,
        "name": "Tether USD",
        "symbol": "USDT",
        "amount": 23810,
        "changeType": "TRANSFER",
        "from": "0xf977814e90da44bfa03b6295a0616a897441acec",
        "rawAmount": "23810000000",
        "to": "0x5964456c99bee03b64d688e690992830d2de84d9"
      }
    ],
    "gasUsed": "0xb429"
  },
  "id": 1,
  "jsonrpc": "2.0"
}
```

---

## 7. zan_simulateExecution

模拟交易执行，返回调用轨迹及解码后的输入/输出（人类可读格式）。

### 参数

| 索引 | 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|------|
| 0 | `txObject` | object | 是 | 交易调用对象，同 zan_simulateAssetChanges |

### 响应

| 字段 | 类型 | 说明 |
|------|------|------|
| `from` | string | 调用方地址 |
| `to` | string | 被调用合约地址 |
| `type` | string | 调用类型：`CALL`、`CALLCODE`、`DELEGATECALL`、`STATICCALL` |
| `value` | string | 转账金额（wei） |
| `gas` | string | Gas 上限 |
| `gasUsed` | string | 实际消耗的 gas |
| `input` | string | 原始输入（hex） |
| `output` | string | 原始输出（hex） |
| `decoded` | object | 解码后的输入/输出 |
| `calls` | array | 子调用列表（递归结构） |
| `logs` | array | 触发的日志事件 |
| `error` | string | 执行失败时的错误信息 |
| `revertReason` | string | 合约 revert 原因 |

### decoded 字段

| 字段 | 类型 | 说明 |
|------|------|------|
| `authority` | string | ABI 来源，如 `ETHERSCAN` |
| `methodName` | string | 调用的方法名 |
| `inputs` | array | 解码后的输入参数 |
| `outputs` | array | 解码后的输出参数 |

### 示例

#### curl

```bash
curl -X POST "https://api.zan.top/data/v1/eth/mainnet/YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "id": 1,
    "jsonrpc": "2.0",
    "method": "zan_simulateExecution",
    "params": [
      {
        "from": "0xf977814e90da44bfa03b6295a0616a897441acec",
        "to": "0xdAC17F958D2ee523a2206206994597C13D831ec7",
        "data": "0xa9059cbb0000000000000000000000005964456c99bee03b64d688e690992830d2de84d9000000000000000000000000000000000000000000000000000000058b2fc480",
        "value": "0x0"
      }
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
    method: "zan_simulateExecution",
    params: [
      {
        from: "0xf977814e90da44bfa03b6295a0616a897441acec",
        to: "0xdAC17F958D2ee523a2206206994597C13D831ec7",
        data: "0xa9059cbb...",
        value: "0x0",
      },
    ],
  }),
}).then((r) => r.json());

console.log("方法名:", result.decoded?.methodName);
console.log("输入参数:", result.decoded?.inputs);
console.log("Gas 消耗:", result.gasUsed);
```

### 响应示例

```json
{
  "result": {
    "decoded": {
      "authority": "ETHERSCAN",
      "methodName": "transfer",
      "inputs": [
        { "name": "_to", "type": "address", "value": "0x5964456c99bee03b64d688e690992830d2de84d9" },
        { "name": "_value", "type": "uint256", "value": "23810000000" }
      ],
      "outputs": []
    },
    "from": "0xf977814e90da44bfa03b6295a0616a897441acec",
    "to": "0xdac17f958d2ee523a2206206994597c13d831ec7",
    "type": "CALL",
    "value": "0x0",
    "gas": "0x2fa9c0c",
    "gasUsed": "0xb429",
    "input": "0xa9059cbb...",
    "logs": [...]
  },
  "id": 1,
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

- **触发条件**：用户询问交易执行路径、内部调用、资金流向、模拟交易结果、资产变化预览时，使用本 API。
- **方法选择**：
  - 追踪已上链交易 → `debug_traceTransaction`
  - 模拟调用并追踪 → `debug_traceCall`
  - 预览资产变化（ERC20/NFT 转账、授权）→ `zan_simulateAssetChanges`
  - 获取解码后的调用详情 → `zan_simulateExecution`
- **参数补全**：`txHash` 需为 0x 开头的有效交易哈希。`txObject` 中 `to` 必填，`data` 需按 ABI 编码。
- **结果说明**：`debug_traceTransaction` 返回原始 hex，`zan_simulateExecution` 返回解码后的人类可读格式。部分链或历史区块可能不支持 trace，会返回错误。
