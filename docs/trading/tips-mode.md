# Solana Trading Boost — Tips 集成方式

## API Name

`sendTransaction`（Tips 方式）

## Category

Trading

## Description

通过向指定账户支付 SOL Tips 来加速 Solana 交易。交易优先级由 Tips 出价金额决定，出价越高优先级越高。基于 ZAN 的 SWQoS（Stake-Weighted Quality of Service）质押验证节点，通过优先通道显著提升交易成功率与速度。

## Supported Auth Methods

| 鉴权方式 | 支持 |
| --- | --- |
| API Key | ✅ |

> 使用 Tips 集成方式前，需提前 [联系 ZAN](https://docs.zan.top/docs/trading-boost#support-channels) 开通账户权限。

## Use Cases

- **高度自定义加速策略**：根据交易紧迫程度动态调整 Tips 金额
- **竞价场景**：在 NFT 铸造、DEX 交易等竞争场景中通过更高 Tips 获取优先
- **复杂业务逻辑**：需要动态优先级调整的套利、做市等策略
- **按需付费**：仅在需要加速时支付，无需预购积分

## Request Method

`POST`

## Endpoint

| 区域 | Endpoint |
| --- | --- |
| 新加坡 | `http://booster-sg.zan.top/node/v1/solana/mainnet/{API_KEY}/tips` |

### MEV 保护

在 URL 后附加 `?anti-mev=true` 启用 MEV 保护（免费）：

```
http://booster-sg.zan.top/node/v1/solana/mainnet/{API_KEY}/tips?anti-mev=true
```

## Parameters

使用标准的 Solana `sendTransaction` 方法，交易中需包含一笔向指定账户转账 Tips 的指令。

| 参数 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `jsonrpc` | string | 是 | 固定为 `"2.0"` |
| `method` | string | 是 | 固定为 `"sendTransaction"` |
| `params` | array | 是 | `[base64_encoded_tx, options]` |
| `id` | number | 是 | 请求 ID |

### params 说明

| 索引 | 参数 | 类型 | 说明 |
| --- | --- | --- | --- |
| 0 | `transaction` | string | Base64 编码的已签名交易（需包含 Tips 转账指令） |
| 1 | `options` | object | 可选配置，如 `{ "encoding": "base64" }` |

### Tips 要求

| 要求 | 说明 |
| --- | --- |
| **最低金额** | 0.0003 SOL（300,000 lamports） |
| **收款账户** | `CEWud6sUrg85auffmvG55cVhGjDbFCGC2HvhGeagd7St` |
| **支付方式** | 在交易中添加一笔 `SystemProgram.transfer` 指令 |

## Response Schema

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `jsonrpc` | string | 固定为 `"2.0"` |
| `id` | number | 与请求 ID 一致 |
| `result` | string | 交易签名（Transaction Signature） |

## Error Codes

| HTTP Status | 说明 | 处理建议 |
| --- | --- | --- |
| 401 | API Key 无效或过期 | 检查 API Key 是否正确 |
| 403 | 账户未开通 Tips 权限 | 联系 ZAN 开通权限 |
| 429 | TPS 超限 | 降低请求频率，超出部分自动回退标准处理 |
| 500 | 服务端内部错误 | 稍后重试 |

### Solana RPC 错误

| 错误码 | 说明 | 处理建议 |
| --- | --- | --- |
| -32002 | 交易模拟失败 | 检查交易构造与签名 |
| -32003 | 交易预检失败 | 检查账户余额与交易参数 |

## Example Request

### curl

```bash
curl -X POST 'http://booster-sg.zan.top/node/v1/solana/mainnet/{YOUR_API_KEY}/tips' \
  -H 'Content-Type: application/json' \
  -d '{
    "jsonrpc": "2.0",
    "id": 1,
    "method": "sendTransaction",
    "params": [
      "<base64_encoded_tx>",
      { "encoding": "base64" }
    ]
  }'
```

### curl（启用 MEV 保护）

```bash
curl -X POST 'http://booster-sg.zan.top/node/v1/solana/mainnet/{YOUR_API_KEY}/tips?anti-mev=true' \
  -H 'Content-Type: application/json' \
  -d '{
    "jsonrpc": "2.0",
    "id": 1,
    "method": "sendTransaction",
    "params": [
      "<base64_encoded_tx>",
      { "encoding": "base64" }
    ]
  }'
```

## Example Response

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": "5VERv8NMhJzWpLvQhKmRwKbPjKp3yP3RGi7Nk8tCWqNhR7EhXkQvZLBnCsJkzKPwDNFgvJHb4kza2NHe7KwbFgS"
}
```

## Code Samples

### JavaScript/TypeScript（构建含 Tips 的交易）

```typescript
import {
  Connection,
  Keypair,
  SystemProgram,
  Transaction,
  PublicKey,
  sendAndConfirmRawTransaction,
} from "@solana/web3.js";
import bs58 from "bs58";

const API_KEY = process.env.ZAN_API_KEY!;
const TIPS_ENDPOINT = `http://booster-sg.zan.top/node/v1/solana/mainnet/${API_KEY}/tips`;
const TIPS_ACCOUNT = new PublicKey("CEWud6sUrg85auffmvG55cVhGjDbFCGC2HvhGeagd7St");
const TIPS_AMOUNT = 300_000; // 0.0003 SOL = 300,000 lamports

const connection = new Connection(TIPS_ENDPOINT, "confirmed");
const payer = Keypair.fromSecretKey(bs58.decode(process.env.SOLANA_PRIVATE_KEY!));

async function sendWithTips(transaction: Transaction) {
  // 添加 Tips 转账指令
  transaction.add(
    SystemProgram.transfer({
      fromPubkey: payer.publicKey,
      toPubkey: TIPS_ACCOUNT,
      lamports: TIPS_AMOUNT,
    })
  );

  // 获取最新 blockhash
  const { blockhash } = await connection.getLatestBlockhash();
  transaction.recentBlockhash = blockhash;
  transaction.feePayer = payer.publicKey;

  // 签名并发送
  transaction.sign(payer);
  const rawTx = transaction.serialize();

  const signature = await connection.sendRawTransaction(rawTx, {
    skipPreflight: false,
    preflightCommitment: "confirmed",
  });

  console.log("交易已提交，签名:", signature);

  // 等待确认
  const confirmation = await connection.confirmTransaction(signature, "confirmed");
  console.log("交易已确认:", confirmation);

  return signature;
}

// 示例：构建一笔带 Tips 的 SOL 转账
const tx = new Transaction().add(
  SystemProgram.transfer({
    fromPubkey: payer.publicKey,
    toPubkey: new PublicKey("目标地址"),
    lamports: 1_000_000, // 0.001 SOL
  })
);

await sendWithTips(tx);
```

### 启用 MEV 保护

```typescript
const TIPS_ENDPOINT_MEV = `http://booster-sg.zan.top/node/v1/solana/mainnet/${API_KEY}/tips?anti-mev=true`;
const connectionMev = new Connection(TIPS_ENDPOINT_MEV, "confirmed");
```

## Notes for Agents

- **前置条件**：Tips 集成方式需提前联系 ZAN 开通权限，Agent 应在用户首次使用时提醒此要求
- **目标链**：Trading Boost 目前**仅支持 Solana**，如用户询问 EVM 链交易加速，应说明暂不支持
- **Tips 金额**：最低 0.0003 SOL，用户可根据紧迫程度提高出价获取更高优先级
- **MEV 保护**：建议高价值交易启用 `?anti-mev=true`，会引入微小延迟
- **鉴权**：仅支持 API Key
- **测试网**：目前不支持测试网

## 风险提示

> **重要提醒：** 本 API 提供的所有数据仅供参考，不构成任何投资或交易建议。Tips 金额为实际链上 SOL 支出，无论交易最终是否成功上链均不予退还，请谨慎评估成本。

## 参考资料

- [ZAN Trading Boost 官方文档](https://docs.zan.top/docs/trading-boost)
- [Solana sendTransaction API](https://docs.zan.top/reference/sendtransaction-solana)
- [Solana SWQoS 文档](https://solana.com/developers/guides/advanced/stake-weighted-qos)
