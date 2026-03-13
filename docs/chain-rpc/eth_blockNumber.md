# eth_blockNumber

## API Name

`eth_blockNumber`

## Category

Chain RPC

## Description

查询当前最新区块高度。返回节点当前已同步到的最新区块号，以十六进制字符串表示。

## Supported Auth Methods

| 鉴权方式 | 支持 |
| --- | --- |
| API Key | ✅ |
<!-- x402 支付方式正在开发中，后续将在此处补充 x402 相关内容 -->

## Use Cases

- **检查节点同步状态**：确认 RPC 节点是否与链同步
- **监控出块速度**：对比不同时间点的区块高度
- **确认区块确认数**：计算交易所在区块与最新区块的差值

## Request Method

`POST`

## Endpoint

| 鉴权方式 | Endpoint |
| --- | --- |
| API Key | `https://api.zan.top/node/v1/eth/mainnet/{API_KEY}` |
<!-- x402 支付方式正在开发中，后续将在此处补充 x402 相关内容 -->

> 将 `eth/mainnet` 替换为目标链，如 `base/mainnet`、`arbitrum/mainnet` 等。

## Parameters

| 参数 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `jsonrpc` | string | 是 | 固定为 `"2.0"` |
| `method` | string | 是 | 固定为 `"eth_blockNumber"` |
| `params` | array | 是 | 空数组 `[]` |
| `id` | number \| string | 是 | 请求 ID，用于匹配响应 |

## Response Schema

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `jsonrpc` | string | 固定为 `"2.0"` |
| `id` | number \| string | 与请求中的 `id` 一致 |
| `result` | string | 当前最新区块高度的十六进制表示，如 `"0x12a4b3c"` |

## Error Codes

### 标准 JSON-RPC 错误

| code | message | 说明 |
| --- | --- | --- |
| -32600 | Invalid Request | 请求格式错误 |
| -32601 | Method not found | 方法名错误 |
| -32602 | Invalid params | 参数格式错误 |
| -32603 | Internal error | 服务端内部错误 |

<!-- x402 支付方式正在开发中，后续将在此处补充 x402 相关内容 -->

## Example Request

### curl（API Key）

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

<!-- x402 支付方式正在开发中，后续将在此处补充 x402 相关内容 -->

## Example Response

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": "0x12a4b3c"
}
```

`result` 为十六进制区块高度，`0x12a4b3c` 转换为十进制为 `19531196`。

## Code Samples

### JavaScript/TypeScript（API Key）

```typescript
const API_KEY = process.env.ZAN_API_KEY;
const endpoint = `https://api.zan.top/node/v1/eth/mainnet/${API_KEY}`;

const response = await fetch(endpoint, {
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
const blockNumberHex = data.result;
const blockNumber = parseInt(blockNumberHex, 16);
console.log("当前区块高度:", blockNumber);
```

<!-- x402 支付方式正在开发中，后续将在此处补充 x402 相关内容 -->

## Notes for Agents

- 当用户询问「最新区块」「区块高度」「节点是否同步」时，使用 `eth_blockNumber`。
- 返回值为十六进制字符串，需转换为十进制后再展示或用于计算。
- 不同链的 endpoint 不同，需根据用户指定的链（Ethereum、Base、Arbitrum 等）选择对应 URL。
- 若用户未指定链，可追问或默认使用 Ethereum Mainnet。
