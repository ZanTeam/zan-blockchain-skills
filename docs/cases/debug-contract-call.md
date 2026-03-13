# 场景 2：调试合约调用

## 场景描述

开发者需要模拟合约调用以调试问题，例如调用失败、Gas 异常或返回值不符合预期。通过 `eth_call` 和 `eth_estimateGas` 可在不上链的情况下模拟执行，快速定位问题。

## 触发语句示例

- 「我调用合约总是失败，帮我排查一下」
- 「这笔交易为什么会 revert？」
- 「帮我估算一下调用这个合约需要多少 Gas」
- 「模拟执行一下这个合约调用，看看返回值」

## 推荐方式

**API Key**

## 实现步骤

### 1. 使用 eth_call 模拟调用

`eth_call` 在指定区块状态下模拟执行合约的只读方法，**不消耗 Gas、不上链**。若合约执行 revert，可能返回错误信息。

### 2. 使用 eth_estimateGas 检查 Gas 情况

`eth_estimateGas` 估算执行交易所需的 Gas 量。若交易会 revert，通常返回错误而非 Gas 值，可用于快速判断调用是否会成功。

### 3. 分析错误原因

根据 `eth_call` 的返回值或错误信息、`eth_estimateGas` 的错误，结合常见错误类型进行排查。

## 代码示例

### 模拟合约调用（eth_call）

```typescript
const API_KEY = process.env.ZAN_API_KEY;
const endpoint = `https://api.zan.top/node/v1/eth/mainnet/${API_KEY}`;

// 示例：调用 Uniswap V2 的 getReserves
const ROUTER_ADDRESS = "0x7a250d5630B4cF539739dF2C5dAcb4c659F2488D";
const callData = "0x0902f1ac"; // getReserves() 选择器

const response = await fetch(endpoint, {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({
    jsonrpc: "2.0",
    method: "eth_call",
    params: [
      {
        to: ROUTER_ADDRESS,
        data: callData,
      },
      "latest",
    ],
    id: 1,
  }),
});

const data = await response.json();

if (data.error) {
  console.error("调用失败:", data.error.message);
} else {
  console.log("返回值 (hex):", data.result);
  // 根据 ABI 解码 data.result
}
```

### 估算 Gas（eth_estimateGas）

```typescript
const txObject = {
  from: "0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb",
  to: "0x7a250d5630B4cF539739dF2C5dAcb4c659F2488D",
  data: "0x38ed1739...", // swapExactTokensForTokens 等
};

const response = await fetch(endpoint, {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({
    jsonrpc: "2.0",
    method: "eth_estimateGas",
    params: [txObject],
    id: 1,
  }),
});

const data = await response.json();

if (data.error) {
  console.error("Gas 估算失败，交易可能 revert:", data.error.message);
  // 解析 data.error.data 可能包含 revert 原因
} else {
  const gasEstimate = parseInt(data.result, 16);
  console.log("预估 Gas:", gasEstimate);
}
```

### 完整调试流程示例

```typescript
async function debugContractCall(
  from: string,
  to: string,
  data: string,
  value?: string
) {
  const API_KEY = process.env.ZAN_API_KEY;
  const endpoint = `https://api.zan.top/node/v1/eth/mainnet/${API_KEY}`;

  const callObject: Record<string, string> = { to, data, from };
  if (value) callObject.value = value;

  // Step 1: eth_call 模拟
  const callRes = await fetch(endpoint, {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({
      jsonrpc: "2.0",
      method: "eth_call",
      params: [callObject, "latest"],
      id: 1,
    }),
  });
  const callData = await callRes.json();

  if (callData.error) {
    console.log("eth_call 失败:", callData.error);
    return { success: false, callError: callData.error };
  }

  // Step 2: eth_estimateGas 估算
  const estRes = await fetch(endpoint, {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({
      jsonrpc: "2.0",
      method: "eth_estimateGas",
      params: [callObject],
      id: 2,
    }),
  });
  const estData = await estRes.json();

  if (estData.error) {
    console.log("eth_estimateGas 失败:", estData.error);
    return {
      success: false,
      callResult: callData.result,
      gasError: estData.error,
    };
  }

  const gasEstimate = parseInt(estData.result, 16);
  return {
    success: true,
    callResult: callData.result,
    gasEstimate,
  };
}
```

## 常见错误与解决方案

| 错误类型 | 可能原因 | 解决方案 |
|----------|----------|----------|
| `execution reverted` | 合约逻辑失败、条件不满足 | 检查参数、权限、余额、时间锁等 |
| `insufficient funds` | 调用方余额不足 | 确保 `from` 地址有足够 ETH 和 Token |
| `gas required exceeds allowance` | Gas 估算失败，交易可能 revert | 先排查 `eth_call` 是否成功 |
| `invalid opcode` | 合约地址错误或未部署 | 确认 `to` 地址正确且已部署 |
| `revert: transfer amount exceeds balance` | 转账金额超过余额 | 检查 Token 余额或 ETH 余额 |
| `revert: ERC20: transfer amount exceeds allowance` | 授权额度不足 | 先调用 `approve` 增加授权 |
| `-32602 Invalid params` | 参数格式错误 | 检查 `data` 编码、地址格式 |

## 相关文档

- [eth_call](../chain-rpc/eth_call.md)
- [eth_estimateGas](../chain-rpc/eth_estimateGas.md)
- [API Key 快速开始](../getting-started/quick-start-api-key.md)
