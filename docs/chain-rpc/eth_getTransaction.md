# eth_getTransactionByHash

## API Name

`eth_getTransactionByHash`

## Category

Chain RPC

## Description

根据交易哈希查询交易详情，返回交易的完整字段（发送方、接收方、金额、Gas、input 等）。若交易尚未被打包或哈希不存在，返回 `null`。

## Supported Auth Methods

| 鉴权方式 | 支持 |
| --- | --- |
| API Key | ✅ |
<!-- x402 支付方式正在开发中，后续将在此处补充 x402 相关内容 -->

## Use Cases

- **查询交易详情**：根据 tx hash 获取交易完整信息
- **确认交易是否被打包**：判断交易是否已进入区块
- **分析交易参数**：解析 `from`、`to`、`value`、`input` 等字段

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
| `method` | string | 是 | 固定为 `"eth_getTransactionByHash"` |
| `params` | array | 是 | `[transactionHash]` |
| `id` | number \| string | 是 | 请求 ID |

### params 说明

| 索引 | 参数 | 类型 | 说明 |
| --- | --- | --- | --- |
| 0 | `transactionHash` | string | 交易哈希，0x 开头的 32 字节十六进制 |

## Response Schema

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `jsonrpc` | string | 固定为 `"2.0"` |
| `id` | number \| string | 与请求中的 `id` 一致 |
| `result` | object \| null | 交易对象，未找到时为 `null` |

### result 对象字段

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `hash` | string | 交易哈希 |
| `nonce` | string | 发送方 nonce（十六进制） |
| `blockHash` | string \| null | 所在区块哈希，未打包时为 `null` |
| `blockNumber` | string \| null | 所在区块号（十六进制），未打包时为 `null` |
| `transactionIndex` | string \| null | 交易在区块中的索引 |
| `from` | string | 发送方地址 |
| `to` | string \| null | 接收方地址，合约创建时为 `null` |
| `value` | string | 转账金额（wei，十六进制） |
| `gas` | string | Gas 上限（十六进制） |
| `gasPrice` | string | Gas 单价（wei，十六进制） |
| `input` | string | 调用数据（0x 开头，合约调用时为 calldata） |
| `v` | string | 签名 v |
| `r` | string | 签名 r |
| `s` | string | 签名 s |

## Error Codes

### 标准 JSON-RPC 错误

| code | message | 说明 |
| --- | --- | --- |
| -32600 | Invalid Request | 请求格式错误 |
| -32601 | Method not found | 方法名错误 |
| -32602 | Invalid params | 交易哈希格式错误 |
| -32603 | Internal error | 服务端内部错误 |

<!-- x402 支付方式正在开发中，后续将在此处补充 x402 相关内容 -->

## Example Request

### curl（API Key）

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

<!-- x402 支付方式正在开发中，后续将在此处补充 x402 相关内容 -->

## Example Response

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": {
    "hash": "0x88df016429689c079f3b2f6ad39fa052532c56795b733da78a91ebe6a713944b",
    "nonce": "0x1",
    "blockHash": "0x1d59ff54b1eb26b013ce3cb5fc9dab3705b415a6716a3d3f0e3542efb52f9e50",
    "blockNumber": "0x12a4b3c",
    "transactionIndex": "0x0",
    "from": "0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb",
    "to": "0xd46e8dd67c5d32be8058bb8eb970870f07244567",
    "value": "0x9184e72a000",
    "gas": "0x76c0",
    "gasPrice": "0x9184e72a000",
    "input": "0x",
    "v": "0x25",
    "r": "0x...",
    "s": "0x..."
  }
}
```

## Code Samples

### JavaScript/TypeScript（API Key）

```typescript
const API_KEY = process.env.ZAN_API_KEY;
const endpoint = `https://api.zan.top/node/v1/eth/mainnet/${API_KEY}`;
const txHash = "0x88df016429689c079f3b2f6ad39fa052532c56795b733da78a91ebe6a713944b";

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
if (tx) {
  console.log("交易已打包，区块号:", parseInt(tx.blockNumber, 16));
  console.log("发送方:", tx.from);
  console.log("接收方:", tx.to);
  console.log("金额(wei):", tx.value);
} else {
  console.log("交易未找到或尚未打包");
}
```

<!-- x402 支付方式正在开发中，后续将在此处补充 x402 相关内容 -->

## Notes for Agents

- 当用户提供交易哈希并询问「交易详情」「交易是否成功」「交易状态」时，使用 `eth_getTransactionByHash`。
- 若需确认**执行结果**（成功/失败、Gas 消耗），应使用 `eth_getTransactionReceipt`。
- `result` 为 `null` 表示交易未被打包或哈希不存在，可提示用户稍后重试。
- `value` 为 wei，展示时可转换为 ETH。
- 需根据用户指定的链选择对应 endpoint。
