# 场景 3：追踪交易

## 场景描述

需要分析一笔复杂交易的完整调用链，包括内部调用（internal calls）、资金流向、合约交互路径等。适用于调试 DeFi 交互、追踪资金、理解多合约调用逻辑。

## 触发语句示例

- 「帮我分析一下这笔交易的内部调用情况」
- 「这笔交易的钱都转去哪了？」
- 「追踪一下 0x88df016429689c079f3b2f6ad39fa052532c56795b733da78a91ebe6a713944b 的调用链」

## 推荐方式

**API Key**

## 实现步骤

### 1. 使用 eth_getTransactionByHash 获取交易基本信息

获取交易的 `from`、`to`、`value`、`input`、`blockNumber` 等基础字段，确认交易已打包。

### 2. 使用 transaction-trace 获取完整调用链

调用 Data API 的 `transaction-trace`，返回树形结构的内部调用链，包括 `call`、`delegatecall`、`staticcall` 等类型。

### 3. 分析资金流向

遍历 trace 树，筛选 `value` 非零的调用，汇总资金转入转出。

## 代码示例

### 获取交易基本信息

```typescript
const API_KEY = process.env.ZAN_API_KEY;
const endpoint = `https://api.zan.top/node/v1/eth/mainnet/${API_KEY}`;
const txHash =
  "0x88df016429689c079f3b2f6ad39fa052532c56795b733da78a91ebe6a713944b";

const response = await fetch(endpoint, {
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
const tx = data.result;

if (!tx) {
  console.log("交易未找到或尚未打包");
} else {
  console.log("发送方:", tx.from);
  console.log("接收方:", tx.to);
  console.log("金额 (wei):", tx.value);
  console.log("区块号:", parseInt(tx.blockNumber, 16));
}
```

### 获取完整调用链

```typescript
const txHash =
  "0x88df016429689c079f3b2f6ad39fa052532c56795b733da78a91ebe6a713944b";

const params = new URLSearchParams({
  apiKey: process.env.ZAN_API_KEY!,
  txHash,
  chain: "eth",
});

const response = await fetch(
  `https://api.zan.top/data/v1/transaction/trace?${params}`
);
const data = await response.json();

// 递归打印调用树
function printTrace(trace: any, depth = 0) {
  const indent = "  ".repeat(depth);
  const value = trace.value ? BigInt(trace.value).toString() : "0";
  console.log(
    `${indent}[${trace.type}] ${trace.from?.slice(0, 10)}... -> ${trace.to?.slice(0, 10)}... value=${value}`
  );
  trace.calls?.forEach((c: any) => printTrace(c, depth + 1));
}
printTrace(data.trace);
```

### 完整分析示例（含资金流向）

```typescript
interface TraceNode {
  type: string;
  from?: string;
  to?: string;
  value?: string;
  gas?: string;
  gasUsed?: string;
  calls?: TraceNode[];
}

function analyzeTrace(trace: TraceNode, depth = 0): void {
  const indent = "  ".repeat(depth);
  const valueWei = trace.value ? BigInt(trace.value) : 0n;
  const valueEth = Number(valueWei) / 1e18;

  if (valueWei > 0n) {
    console.log(
      `${indent}💰 ${trace.from} -> ${trace.to}: ${valueEth} ETH`
    );
  }

  trace.calls?.forEach((c) => analyzeTrace(c, depth + 1));
}

// 使用
const data = await response.json();
analyzeTrace(data.trace);
```

## 结果解读

### trace 结构说明

| 字段 | 说明 |
|------|------|
| `type` | 调用类型：`call`、`delegatecall`、`staticcall`、`create` |
| `from` | 调用方地址 |
| `to` | 被调用方地址 |
| `value` | 转账金额（wei） |
| `gas` / `gasUsed` | 分配的 Gas / 实际消耗 |
| `input` / `output` | 调用输入/输出（hex） |
| `calls` | 嵌套的内部调用 |

### 调用类型含义

- **call**：普通调用，可转账 ETH
- **delegatecall**：委托调用，在调用方上下文中执行
- **staticcall**：静态调用，不允许状态变更
- **create**：合约创建

### 资金流向分析要点

- `value > 0` 的调用表示有 ETH 转移
- 根节点通常为用户发起的顶层调用
- 嵌套 `calls` 表示合约 A 调用合约 B 再调用合约 C 的链式结构
- 部分链或历史区块可能不支持 trace，会返回 `404`

## 相关文档

- [eth_getTransactionByHash](../chain-rpc/eth_getTransaction.md)
- [transaction-trace](../data/transaction-trace.md)
- [API Key 快速开始](../getting-started/quick-start-api-key.md)
