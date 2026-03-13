# query-credit-balance

> **注意：本 API 依赖 x402 支付功能，该功能正在开发中，暂未开放。待 x402 上线后启用。**

<!--
## API Name

`query-credit-balance`

## Category

Operational

## Description

查询 x402 支付方式下的信用余额。适用于使用 x402 即用即付的用户，在调用前检查信用余额是否充足、了解累计购买与消耗情况。

## Supported Auth Methods

| 鉴权方式 | 支持 |
| --- | --- |
| API Key | ❌ |
| x402 | ✅ |

> 本 API 仅支持 x402 鉴权，用于查询与钱包地址关联的 x402 信用余额。API Key 用户请使用 `query-quota` 查询配额。

## Use Cases

- **调用前检查余额**：使用 x402 调用 API 前，确认信用余额是否足够
- **监控消费情况**：查看累计购买与使用量，了解 x402 消费概况
- **余额不足预警**：余额较低时提前充值，避免调用失败
- **对账与核对**：结合 `query-payment-history` 核对余额变动

## Request Method

`GET`

## Endpoint

```
https://api.zan.top/x402/operational/v1/credit-balance
```

> x402 请求需通过 x402 SDK 发起，SDK 会自动完成 SIWE 签名与鉴权。

## Parameters

| 参数 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `walletAddress` | string | 是 | 要查询的钱包地址（0x 开头），需与 x402 支付时使用的钱包一致 |

### 查询参数示例

```
?walletAddress=0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb
```

## Response Schema

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `creditBalance` | number | 当前信用余额（可用额度） |
| `totalPurchased` | number | 累计购买的信用总量 |
| `totalUsed` | number | 累计已使用的信用量 |
| `lastPurchaseTime` | string | 最近一次购买时间（ISO 8601 格式），若从未购买则为 null |

### 响应示例结构

```json
{
  "creditBalance": 1250.5,
  "totalPurchased": 5000,
  "totalUsed": 3749.5,
  "lastPurchaseTime": "2025-03-10T14:30:00Z"
}
```

## Error Codes

| HTTP Status | 错误码 | 说明 | 处理建议 |
| --- | --- | --- | --- |
| 400 | `INVALID_ADDRESS` | 钱包地址格式无效 | 检查 walletAddress 是否为 0x 开头的有效地址 |
| 401 | `UNAUTHORIZED` | x402 鉴权失败 | 检查 SIWE 签名、钱包配置是否正确 |
| 402 | `PAYMENT_REQUIRED` | 需要支付（本 API 通常免费，若出现可联系支持） | 确认 x402 配置 |
| 429 | `RATE_LIMITED` | 请求频率超限 | 降低调用频率，稍后重试 |
| 500 | `INTERNAL_ERROR` | 服务端内部错误 | 稍后重试或联系技术支持 |

## Example Request

### curl（x402）

```bash
# x402 需通过 x402 SDK 发起请求，curl 需携带 x402 协议头
curl -X GET "https://api.zan.top/x402/operational/v1/credit-balance?walletAddress=0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb" \
  -H "Authorization: SIWE <signed_message>"
```

> 实际使用中，x402 请求建议通过 x402 SDK 自动处理签名与鉴权。

## Example Response

### 成功响应

```json
{
  "creditBalance": 1250.5,
  "totalPurchased": 5000,
  "totalUsed": 3749.5,
  "lastPurchaseTime": "2025-03-10T14:30:00Z"
}
```

### 失败响应

```json
{
  "error": {
    "code": "INVALID_ADDRESS",
    "message": "钱包地址格式无效"
  }
}
```

## Code Samples

### JavaScript/TypeScript（x402）

```typescript
import { wrapAxios } from "x402-axios";
import axios from "axios";

const client = wrapAxios(axios, {
  privateKey: process.env.WALLET_PRIVATE_KEY!,
});

const walletAddress = "0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb";

const response = await client.get(
  "https://api.zan.top/x402/operational/v1/credit-balance",
  {
    params: { walletAddress },
  }
);

const data = response.data;

console.log("当前信用余额:", data.creditBalance);
console.log("累计购买:", data.totalPurchased);
console.log("累计使用:", data.totalUsed);
console.log("最近购买时间:", data.lastPurchaseTime);

if (data.creditBalance < 100) {
  console.warn("信用余额较低，建议充值");
}
```

## Notes for Agents

- **触发条件**：用户询问「x402 余额」「信用余额」「还有多少额度」（且用户使用 x402）时，优先使用本 API。
- **鉴权限制**：仅支持 x402，API Key 用户需使用 `query-quota` 查询配额。
- **参数补全**：`walletAddress` 必填，通常从 x402 配置中获取当前使用的钱包地址。
- **结果解读**：`creditBalance` 为当前可用额度，低于预期时可提示用户充值；`totalPurchased` 与 `totalUsed` 可用于对账。
- **与 query-quota 区别**：本 API 面向 x402 用户；`query-quota` 面向 API Key 用户，二者鉴权与数据模型不同。
-->
