# eth_call

## API Name

`eth_call`

## Category

Chain RPC

## Description

调用合约的**只读方法**（view/pure），在指定区块状态下模拟执行，**不消耗 Gas、不上链**。返回方法的十六进制编码返回值，需按 ABI 解码。

## Supported Auth Methods

| 鉴权方式 | 支持 |
| --- | --- |
| API Key | ✅ |
<!-- x402 支付方式正在开发中，后续将在此处补充 x402 相关内容 -->

## Use Cases

- **查询 ERC20 代币余额**：调用 `balanceOf(address)` 获取持仓
- **读取合约状态**：调用任意 view/pure 方法获取链上数据
- **模拟交易结果**：在提交交易前预览执行返回值

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
| `method` | string | 是 | 固定为 `"eth_call"` |
| `params` | array | 是 | `[callObject, blockTag]` |
| `id` | number \| string | 是 | 请求 ID |

### params 说明

| 索引 | 参数 | 类型 | 说明 |
| --- | --- | --- | --- |
| 0 | `callObject` | object | 调用对象 |
| 1 | `blockTag` | string | 区块标签：`"latest"`、`"earliest"` 或区块号十六进制 |

### callObject 字段

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `to` | string | 是 | 合约地址 |
| `data` | string | 否 | 调用数据（函数选择器 + 参数 ABI 编码），不传则视为转账 |
| `from` | string | 否 | 模拟调用者地址，部分合约需要 |
| `value` | string | 否 | 模拟转账金额（wei，十六进制） |
| `gas` | string | 否 | Gas 上限（十六进制） |

## Response Schema

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `jsonrpc` | string | 固定为 `"2.0"` |
| `id` | number \| string | 与请求中的 `id` 一致 |
| `result` | string | 方法返回值的十六进制编码，如 `"0x0000000000000000000000000000000000000000000000000de0b6b3a7640000"` |

返回值需根据合约 ABI 解码。例如 `balanceOf` 返回 uint256，对应 32 字节十六进制。

## Error Codes

### 标准 JSON-RPC 错误

| code | message | 说明 |
| --- | --- | --- |
| -32600 | Invalid Request | 请求格式错误 |
| -32601 | Method not found | 方法名错误 |
| -32602 | Invalid params | 参数格式错误 |
| -32603 | Internal error | 服务端内部错误 |

### 合约执行错误

若合约执行 revert，`result` 可能包含 `revert` 原因，或返回错误对象。

<!-- x402 支付方式正在开发中，后续将在此处补充 x402 相关内容 -->

## Example Request

### 查询 ERC20 balanceOf

`balanceOf(address)` 的函数选择器为 `0x70a08231`，参数为 32 字节地址（左填充 0）。

### curl（API Key）

```bash
curl -X POST https://api.zan.top/node/v1/eth/mainnet/YOUR_API_KEY \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "method": "eth_call",
    "params": [
      {
        "to": "0xdAC17F958D2ee523a2206206994597C13D831ec7",
        "data": "0x70a08231000000000000000000000000742d35Cc6634C0532925a3b844Bc9e7595f0bEb"
      },
      "latest"
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
  "result": "0x0000000000000000000000000000000000000000000000000de0b6b3a7640000"
}
```

`0x0de0b6b3a7640000` = 1000000000000000000 wei = 1 个代币（18 位小数）。

## Code Samples

### JavaScript/TypeScript（API Key）- 查询 USDT 余额

```typescript
import { ethers } from "ethers";

const API_KEY = process.env.ZAN_API_KEY;
const endpoint = `https://api.zan.top/node/v1/eth/mainnet/${API_KEY}`;

// USDT 合约地址 (Ethereum Mainnet)
const USDT_ADDRESS = "0xdAC17F958D2ee523a2206206994597C13D831ec7";
const holderAddress = "0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb";

// 使用 ethers 编码 balanceOf(address)
const iface = new ethers.Interface(["function balanceOf(address) view returns (uint256)"]);
const data = iface.encodeFunctionData("balanceOf", [holderAddress]);

const response = await fetch(endpoint, {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({
    jsonrpc: "2.0",
    method: "eth_call",
    params: [{ to: USDT_ADDRESS, data }, "latest"],
    id: 1,
  }),
});

const dataRes = await response.json();
const balanceWei = BigInt(dataRes.result);
const balance = Number(balanceWei) / 1e6; // USDT 为 6 位小数
console.log(`USDT 余额: ${balance}`);
```

<!-- x402 支付方式正在开发中，后续将在此处补充 x402 相关内容 -->

## Notes for Agents

- **仅支持只读方法**：`eth_call` 不执行状态变更，适用于 view/pure。写入操作需通过发送交易完成。
- **data 编码**：需按 ABI 编码函数选择器与参数，建议使用 ethers/viem 等库的 `encodeFunctionData`。
- **代币小数位**：ERC20 余额需根据合约的 `decimals()` 换算，USDT/USDC 通常为 6，ETH 为 18。
- **区分原生币与 Token**：原生币用 `eth_getBalance`，Token 用 `eth_call` + `balanceOf`。
- 需根据用户指定的链选择 endpoint 和合约地址。
