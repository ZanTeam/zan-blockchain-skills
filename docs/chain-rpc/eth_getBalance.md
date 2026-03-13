# eth_getBalance

## API Name

`eth_getBalance`

## Category

Chain RPC

## Description

查询指定地址的**原生币**余额（如 ETH、BNB、MATIC 等），以 wei 为单位返回十六进制字符串。不包含 ERC20 等代币余额。

## Supported Auth Methods

| 鉴权方式 | 支持 |
| --- | --- |
| API Key | ✅ |
<!-- x402 支付方式正在开发中，后续将在此处补充 x402 相关内容 -->

## Use Cases

- **查询钱包 ETH 余额**：检查地址持有的原生币数量
- **验证转账是否到账**：转账前后对比余额变化
- **判断账户是否有足够 Gas 费**：提交交易前检查余额是否充足

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
| `method` | string | 是 | 固定为 `"eth_getBalance"` |
| `params` | array | 是 | `[address, blockTag]` |
| `id` | number \| string | 是 | 请求 ID，用于匹配响应 |

### params 说明

| 索引 | 参数 | 类型 | 说明 |
| --- | --- | --- | --- |
| 0 | `address` | string | 要查询的地址，需为 0x 开头的 20 字节地址 |
| 1 | `blockTag` | string | 区块标签：`"latest"`（最新）、`"earliest"`（创世块）、或区块号十六进制 |

## Response Schema

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `jsonrpc` | string | 固定为 `"2.0"` |
| `id` | number \| string | 与请求中的 `id` 一致 |
| `result` | string | 余额的十六进制表示（单位：wei），如 `"0x1bc16d674ec80000"` |

### wei 与 ETH 换算

- 1 ETH = 10^18 wei
- 将十六进制转为 BigInt 后除以 `10^18` 可得 ETH 数量

## Error Codes

### 标准 JSON-RPC 错误

| code | message | 说明 |
| --- | --- | --- |
| -32600 | Invalid Request | 请求格式错误 |
| -32601 | Method not found | 方法名错误 |
| -32602 | Invalid params | 参数格式错误（如地址格式不正确） |
| -32603 | Internal error | 服务端内部错误 |

<!-- x402 支付方式正在开发中，后续将在此处补充 x402 相关内容 -->

## Example Request

### curl（API Key）

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

<!-- x402 支付方式正在开发中，后续将在此处补充 x402 相关内容 -->

## Example Response

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": "0x1bc16d674ec80000"
}
```

`0x1bc16d674ec80000` = 2000000000000000000 wei = 2 ETH。

## Code Samples

### JavaScript/TypeScript（API Key）

```typescript
const API_KEY = process.env.ZAN_API_KEY;
const endpoint = `https://api.zan.top/node/v1/eth/mainnet/${API_KEY}`;
const address = "0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb";

const response = await fetch(endpoint, {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({
    jsonrpc: "2.0",
    method: "eth_getBalance",
    params: [address, "latest"],
    id: 1,
  }),
});

const data = await response.json();
const balanceWei = BigInt(data.result);
const balanceEth = Number(balanceWei) / 1e18;
console.log(`余额: ${balanceEth} ETH`);
```

<!-- x402 支付方式正在开发中，后续将在此处补充 x402 相关内容 -->

### wei 转 ETH 工具函数

```typescript
function weiToEth(weiHex: string): number {
  return Number(BigInt(weiHex)) / 1e18;
}
```

## Notes for Agents

- **区分原生币与 Token**：`eth_getBalance` 仅查询原生币（ETH/BNB 等），ERC20 等代币需使用 `eth_call` 调用合约的 `balanceOf`。
- **确认目标链**：不同链的原生币不同（ETH、BNB、MATIC 等），需根据用户指定的链选择 endpoint 并说明币种。
- **wei 转 ETH**：返回值为 wei，展示时建议转换为 ETH（除以 10^18），避免大数显示不友好。
- **地址格式**：地址需为 0x 开头的 20 字节格式，可主动做 checksum 校验。
- 若用户未明确是「原生币」还是「Token」，应追问或同时说明两种查询方式。
