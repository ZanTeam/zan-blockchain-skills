# query-payment-history

> **注意：本 API 依赖 x402 支付功能，该功能正在开发中，暂未开放。待 x402 上线后启用。**

<!--
## API Name

`query-payment-history`

## Category

Operational

## Description

查询 x402 支付历史记录，返回指定钱包地址的 USDC 支付与信用购买记录。适用于对账、核对消费明细、审计支付记录等场景。

## Supported Auth Methods

| 鉴权方式 | 支持 |
| --- | --- |
| API Key | ❌ |
| x402 | ✅ |

> 本 API 仅支持 x402 鉴权，用于查询与钱包地址关联的支付历史。API Key 用户的账单信息请通过 ZAN 控制台查看。

## Use Cases

- **对账与核对**：查看历史支付记录，核对消费明细
- **审计与追溯**：追溯某笔支付的链上交易、金额、时间
- **消费分析**：分析支付频率、金额分布，了解使用习惯
- **问题排查**：支付异常时，通过历史记录定位问题

## Request Method

`GET`

## Endpoint

```
https://api.zan.top/x402/operational/v1/payment-history
```

> x402 请求需通过 x402 SDK 发起，SDK 会自动完成 SIWE 签名与鉴权。

## Parameters

| 参数 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `walletAddress` | string | 是 | 要查询的钱包地址（0x 开头） |
| `startTime` | string | 否 | 起始时间（ISO 8601 格式），不传则不限 |
| `endTime` | string | 否 | 结束时间（ISO 8601 格式），不传则不限 |
| `page` | number | 否 | 页码，默认 1 |
| `pageSize` | number | 否 | 每页条数，默认 20，最大 100 |

### 查询参数示例

```
?walletAddress=0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb&startTime=2025-03-01T00:00:00Z&endTime=2025-03-13T23:59:59Z&page=1&pageSize=20
```

## Response Schema

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `payments` | array | 支付记录数组 |
| `total` | number | 总记录数 |
| `page` | number | 当前页码 |
| `pageSize` | number | 每页条数 |

### payments 数组元素

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `txHash` | string | 链上交易哈希 |
| `amount` | string | 支付金额（USDC，可读格式） |
| `credits` | number | 购买的信用数量 |
| `timestamp` | string | 支付时间（ISO 8601 格式） |
| `status` | string | 状态：`confirmed`、`pending`、`failed` |
| `network` | string | 支付所在网络，如 `base`、`base-sepolia` |

### 响应示例结构

```json
{
  "payments": [
    {
      "txHash": "0xabc123...",
      "amount": "10.00",
      "credits": 1000,
      "timestamp": "2025-03-10T14:30:00Z",
      "status": "confirmed",
      "network": "base"
    }
  ],
  "total": 25,
  "page": 1,
  "pageSize": 20
}
```

## Error Codes

| HTTP Status | 错误码 | 说明 | 处理建议 |
| --- | --- | --- | --- |
| 400 | `INVALID_ADDRESS` | 钱包地址格式无效 | 检查 walletAddress 是否为 0x 开头的有效地址 |
| 400 | `INVALID_TIME_RANGE` | 时间范围无效 | 确保 startTime < endTime，格式为 ISO 8601 |
| 401 | `UNAUTHORIZED` | x402 鉴权失败 | 检查 SIWE 签名、钱包配置是否正确 |
| 429 | `RATE_LIMITED` | 请求频率超限 | 降低调用频率，稍后重试 |
| 500 | `INTERNAL_ERROR` | 服务端内部错误 | 稍后重试或联系技术支持 |

## Example Request

### curl（x402）

```bash
# x402 需通过 x402 SDK 发起请求
curl -X GET "https://api.zan.top/x402/operational/v1/payment-history?walletAddress=0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb&page=1&pageSize=10" \
  -H "Authorization: SIWE <signed_message>"
```

> 实际使用中，x402 请求建议通过 x402 SDK 自动处理签名与鉴权。

## Example Response

### 成功响应

```json
{
  "payments": [
    {
      "txHash": "0xabc123def456789012345678901234567890123456789012345678901234",
      "amount": "10.00",
      "credits": 1000,
      "timestamp": "2025-03-10T14:30:00Z",
      "status": "confirmed",
      "network": "base"
    },
    {
      "txHash": "0xdef456abc123789012345678901234567890123456789012345678901234",
      "amount": "5.00",
      "credits": 500,
      "timestamp": "2025-03-05T09:15:00Z",
      "status": "confirmed",
      "network": "base"
    }
  ],
  "total": 25,
  "page": 1,
  "pageSize": 20
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
  "https://api.zan.top/x402/operational/v1/payment-history",
  {
    params: {
      walletAddress,
      startTime: "2025-03-01T00:00:00Z",
      endTime: "2025-03-13T23:59:59Z",
      page: 1,
      pageSize: 20,
    },
  }
);

const data = response.data;

console.log("总记录数:", data.total);
data.payments.forEach((p: any) => {
  console.log(`${p.timestamp}: ${p.amount} USDC -> ${p.credits} credits (${p.status})`);
  console.log("  交易哈希:", p.txHash);
});
```

## Notes for Agents

- **触发条件**：用户询问「支付历史」「消费记录」「x402 账单」「某笔支付详情」时，优先使用本 API。
- **鉴权限制**：仅支持 x402，API Key 用户的账单请引导至 ZAN 控制台。
- **参数补全**：`walletAddress` 必填；`startTime`、`endTime` 可用于筛选时间范围；分页参数用于大量记录场景。
- **结果解读**：`amount` 为 USDC 金额，`credits` 为购买的信用数；`txHash` 可链接至区块浏览器查看链上详情；`status` 为 `confirmed` 表示已确认。
- **与 query-credit-balance 配合**：余额查询 + 支付历史可完整还原 x402 账户的收支情况。
-->
