# eth_estimateGas

## API Name

`eth_estimateGas`

## Category

Chain RPC

## Description

估算执行一笔交易所需的 **Gas 量**。在指定区块状态下模拟交易执行，返回预估的 Gas 消耗（十六进制）。若交易会 revert，可能返回错误。

## Supported Auth Methods

| 鉴权方式 | 支持 |
| --- | --- |
| API Key | ✅ |
<!-- x402 支付方式正在开发中，后续将在此处补充 x402 相关内容 -->

## Use Cases

- **提交交易前预估费用**：计算 Gas 费用 = 估算 Gas × Gas Price
- **优化 Gas 设置**：避免设置过高或过低的 Gas 上限
- **检查交易是否会失败**：若估算失败，通常表示交易会 revert

## Request Method

`POST`

## Endpoint

| 鉴权方式 | Endpoint |
| --- | --- |
| API Key | `https://api.zan.top/node/v1/eth/mainnet/{API_KEY}` |
<!-- x402 支付方式正在开发中，后续将在此处补充 x402 相关内容 -->

> 将 `eth/mainnet` 替换为目标链。

## Parameters

| 参数 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `jsonrpc` | string | 是 | 固定为 `"2.0"` |
| `method` | string | 是 | 固定为 `"eth_estimateGas"` |
| `params` | array | 是 | `[transactionObject]` |
| `id` | number \| string | 是 | 请求 ID |

### params 说明

| 索引 | 参数 | 类型 | 说明 |
| --- | --- | --- | --- |
| 0 | `transactionObject` | object | 模拟的交易对象 |

### transactionObject 字段

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `from` | string | 是 | 发送方地址 |
| `to` | string | 否 | 接收方地址，合约创建时不传 |
| `value` | string | 否 | 转账金额（wei，十六进制） |
| `data` | string | 否 | 调用数据（calldata） |
| `gas` | string | 否 | Gas 上限（十六进制），可选 |
| `gasPrice` | string | 否 | Gas 单价（十六进制），可选 |

## Response Schema

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `jsonrpc` | string | 固定为 `"2.0"` |
| `id` | number \| string | 与请求中的 `id` 一致 |
| `result` | string | 估算的 Gas 量（十六进制），如 `"0x5208"` |

`0x5208` = 21000（简单 ETH 转账的基础 Gas）。

## Error Codes

### 标准 JSON-RPC 错误

| code | message | 说明 |
| --- | --- | --- |
| -32600 | Invalid Request | 请求格式错误 |
| -32601 | Method not found | 方法名错误 |
| -32602 | Invalid params | 参数格式错误 |
| -32603 | Internal error | 服务端内部错误 |

### 交易执行失败

若交易会 revert，可能返回：

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "error": {
    "code": -32000,
    "message": "execution reverted: ..."
  }
}
```

<!-- x402 支付方式正在开发中，后续将在此处补充 x402 相关内容 -->

## Example Request

### 估算简单 ETH 转账 Gas

### curl（API Key）

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

<!-- x402 支付方式正在开发中，后续将在此处补充 x402 相关内容 -->

## Example Response

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": "0x5208"
}
```

`0x5208` = 21000（简单转账的典型 Gas）。

## Code Samples

### JavaScript/TypeScript（API Key）

```typescript
const API_KEY = process.env.ZAN_API_KEY;
const endpoint = `https://api.zan.top/node/v1/eth/mainnet/${API_KEY}`;

const txObject = {
  from: "0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb",
  to: "0xd46e8dd67c5d32be8058bb8eb970870f07244567",
  value: "0x9184e72a000", // 0.0001 ETH
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
  console.error("估算失败，交易可能 revert:", data.error.message);
} else {
  const gasEstimate = parseInt(data.result, 16);
  console.log("预估 Gas:", gasEstimate);
  // 结合 gasPrice 可计算费用：fee = gasEstimate * gasPrice
}
```

<!-- x402 支付方式正在开发中，后续将在此处补充 x402 相关内容 -->

### 计算预估费用示例

```typescript
// 获取 gasPrice 后计算费用
const gasEstimate = 21000;
const gasPriceWei = 20n * 10n ** 9n; // 20 Gwei
const feeWei = BigInt(gasEstimate) * gasPriceWei;
const feeEth = Number(feeWei) / 1e18;
console.log(`预估费用: ${feeEth} ETH`);
```

## Notes for Agents

- 当用户询问「这笔交易要多少 Gas」「预估费用」「Gas 估算」时，使用 `eth_estimateGas`。
- **估算值仅供参考**：实际消耗可能因状态变化而不同，建议设置 Gas 上限时留 10–20% 余量。
- **估算失败**：若返回错误，通常表示交易会 revert（余额不足、合约逻辑失败等），应提示用户检查参数。
- **简单转账**：通常为 21000 Gas；合约调用会更高。
- 需根据用户指定的链选择 endpoint。
