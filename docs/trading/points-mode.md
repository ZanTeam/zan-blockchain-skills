# Solana Trading Boost — Points 集成方式

## API Name

`sendTransaction`（Points 方式）

## Category

Trading

## Description

通过预购 Points 积分加速 Solana 交易。交易加速成功后异步扣除费用，优先级由预设的服务等级（Prime / Turbo / Standard）决定。基于 ZAN 的 SWQoS（Stake-Weighted Quality of Service）质押验证节点，提供固定透明的定价与简洁的使用体验。

## Supported Auth Methods

| 鉴权方式 | 支持 |
| --- | --- |
| API Key | ✅ |

## Use Cases

- **需求简单直接**：不需要动态竞价，按固定等级加速即可
- **运营简洁**：固定透明定价，系统依赖少，便于成本预估
- **稳定流程**：适合需要稳定、可预测的交易加速服务的业务
- **高频交易**：批量交易场景下的稳定加速支持

## Request Method

`POST`

## Endpoint

| 区域 | Endpoint |
| --- | --- |
| 新加坡 | `https://api.zan.top/node/v1/solana/mainnet/{API_KEY}/prime` |
| 弗吉尼亚 | `https://api-us.zan.top/node/v1/solana/mainnet/{API_KEY}/prime` |

### URL 参数

| 参数 | 说明 | 可选值 |
| --- | --- | --- |
| `level` | 加速等级 | `prime`（$0.5）、`turbo`（$0.3）、`standard`（$0.1） |
| `anti-mev` | MEV 保护 | `true`（启用）、`false` 或省略（禁用） |

> 未指定 `level` 参数时，交易将以标准（非加速）方式处理。

### 端点示例

```
# Prime 等级
https://api.zan.top/node/v1/solana/mainnet/{API_KEY}/prime

# Turbo 等级
https://api.zan.top/node/v1/solana/mainnet/{API_KEY}/turbo

# Standard 等级
https://api.zan.top/node/v1/solana/mainnet/{API_KEY}/standard

# Prime 等级 + MEV 保护
https://api.zan.top/node/v1/solana/mainnet/{API_KEY}/prime?anti-mev=true
```

## 服务等级与定价

| 等级 | 优先级 | Points 消耗 | 费用 |
| --- | --- | --- | --- |
| **Prime** | 专属通道 | 5 Points | $0.5 |
| **Turbo** | 高优先级 | 3 Points | $0.3 |
| **Standard** | 共享通道 | 1 Point | $0.1 |

> - 1 Point = $0.1 USD
> - 仅在交易成功上链确认后扣除 Points
> - 余额不足时自动回退至标准（非加速）处理

## Parameters

使用标准的 Solana `sendTransaction` 方法，无需额外参数。

| 参数 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `jsonrpc` | string | 是 | 固定为 `"2.0"` |
| `method` | string | 是 | 固定为 `"sendTransaction"` |
| `params` | array | 是 | `[base64_encoded_tx, options]` |
| `id` | number | 是 | 请求 ID |

### params 说明

| 索引 | 参数 | 类型 | 说明 |
| --- | --- | --- | --- |
| 0 | `transaction` | string | Base64 编码的已签名交易 |
| 1 | `options` | object | 可选配置，如 `{ "encoding": "base64" }` |

## Response Schema

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `jsonrpc` | string | 固定为 `"2.0"` |
| `id` | number | 与请求 ID 一致 |
| `result` | string | 交易签名（Transaction Signature） |

## Error Codes

| HTTP Status | 说明 | 处理建议 |
| --- | --- | --- |
| 401 | API Key 无效或过期 | 检查 API Key |
| 429 | TPS 超限 | 降低请求频率，超出部分自动回退标准处理 |
| 500 | 服务端内部错误 | 稍后重试 |

### Solana RPC 错误

| 错误码 | 说明 | 处理建议 |
| --- | --- | --- |
| -32002 | 交易模拟失败 | 检查交易构造与签名 |
| -32003 | 交易预检失败 | 检查账户余额与交易参数 |

### Points 相关

| 场景 | 行为 |
| --- | --- |
| Points 余额不足 | 自动回退至标准非加速处理 |
| 交易未成功上链 | 不扣除 Points |

## Example Request

### curl（Prime 等级）

```bash
curl -X POST 'https://api.zan.top/node/v1/solana/mainnet/{YOUR_API_KEY}/prime' \
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

### curl（Turbo 等级 + MEV 保护）

```bash
curl -X POST 'https://api.zan.top/node/v1/solana/mainnet/{YOUR_API_KEY}/turbo?anti-mev=true' \
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
  "result": "3Dg5bKMr9MqfnCH7C4LjVFnQkJoS8KZsJPnRvLVkwDxN1YhtQRYhLCHxH4oYzzGJcUuLgZ5dH3sWqwz2RRDMQY"
}
```

## Code Samples

### Python

```python
import requests

API_KEY = "YOUR_API_KEY"
url = f"https://api.zan.top/node/v1/solana/mainnet/{API_KEY}/prime"

payload = {
    "jsonrpc": "2.0",
    "id": 1,
    "method": "sendTransaction",
    "params": [
        "<base64_encoded_tx>",
        {"encoding": "base64"}
    ]
}

headers = {
    "accept": "application/json",
    "content-type": "application/json"
}

response = requests.post(url, json=payload, headers=headers)
print(response.json())
```

### JavaScript/TypeScript

```typescript
import { Connection, Keypair, SystemProgram, Transaction, PublicKey } from "@solana/web3.js";
import bs58 from "bs58";

const API_KEY = process.env.ZAN_API_KEY!;

// 选择加速等级：prime / turbo / standard
const LEVEL = "prime";
const ENDPOINT = `https://api.zan.top/node/v1/solana/mainnet/${API_KEY}/${LEVEL}`;

// 启用 MEV 保护（可选）
// const ENDPOINT = `https://api.zan.top/node/v1/solana/mainnet/${API_KEY}/${LEVEL}?anti-mev=true`;

const connection = new Connection(ENDPOINT, "confirmed");
const payer = Keypair.fromSecretKey(bs58.decode(process.env.SOLANA_PRIVATE_KEY!));

async function sendWithBoost(transaction: Transaction) {
  const { blockhash } = await connection.getLatestBlockhash();
  transaction.recentBlockhash = blockhash;
  transaction.feePayer = payer.publicKey;

  transaction.sign(payer);
  const rawTx = transaction.serialize();

  const signature = await connection.sendRawTransaction(rawTx, {
    skipPreflight: false,
    preflightCommitment: "confirmed",
  });

  console.log("交易已提交，签名:", signature);

  const confirmation = await connection.confirmTransaction(signature, "confirmed");
  console.log("交易已确认:", confirmation);

  return signature;
}

// 构建交易
const tx = new Transaction().add(
  SystemProgram.transfer({
    fromPubkey: payer.publicKey,
    toPubkey: new PublicKey("目标地址"),
    lamports: 1_000_000,
  })
);

await sendWithBoost(tx);
```

### 动态选择加速等级

```typescript
function getBoostEndpoint(apiKey: string, level: "prime" | "turbo" | "standard", antiMev = false): string {
  const base = `https://api.zan.top/node/v1/solana/mainnet/${apiKey}/${level}`;
  return antiMev ? `${base}?anti-mev=true` : base;
}

// 高价值交易使用 Prime + MEV 保护
const highValueEndpoint = getBoostEndpoint(API_KEY, "prime", true);

// 普通交易使用 Standard
const normalEndpoint = getBoostEndpoint(API_KEY, "standard");
```

## 购买 Points

- 在 [ZAN 控制台](https://zan.top) 购买 Points 积分
- 1 Point = $0.1 USD
- Points 仅在交易成功上链后扣除
- 余额不足时自动回退至标准处理，不会导致交易失败

## 监控与诊断

### Dashboard

登录 ZAN 用户门户 → SOL Trading → Overview → Points 视图，可查看：

| 指标 | 说明 |
| --- | --- |
| **Success Rate** | 各等级的交易确认率 |
| **On-Chain Time** | 提交到上链的耗时 |
| **Remaining Points** | 实时剩余 Points |

### 交易验证

1. 在 SOL Trading Boost 模块进入「查看消费详情」
2. 通过交易哈希搜索
3. 页面显示最近 30 天的记录，未出现表示未经加速处理

### 导出交易记录

- 每日最多导出 5 次
- 支持导出最近 6 个月的 Tips 消费详情

## Notes for Agents

- **使用门槛低**：Points 方式无需提前开通权限（不同于 Tips），购买 Points 后直接使用
- **等级选择**：
  - 对延迟敏感的交易（套利、抢购）→ 推荐 **Prime**
  - 一般加速需求 → 推荐 **Turbo**
  - 基础加速或成本敏感 → 推荐 **Standard**
- **目标链**：仅支持 **Solana Mainnet**，不支持测试网和其他链
- **鉴权**：仅支持 API Key
- **计费说明**：Points 仅在交易成功上链后扣除，交易失败不收费
- **MEV 保护**：建议高价值交易启用 `?anti-mev=true`
- **吞吐量**：与 Chain RPC 共享 TPS 限制，超出部分自动回退标准处理

## 风险提示

> **重要提醒：** 本 API 提供的所有数据仅供参考，不构成任何投资或交易建议。Points 消耗为实际费用支出，请谨慎评估交易加速的成本与必要性。

## 参考资料

- [ZAN Trading Boost 官方文档](https://docs.zan.top/docs/trading-boost)
- [Solana sendTransaction API](https://docs.zan.top/reference/sendtransaction-solana)
- [Solana SWQoS 文档](https://solana.com/developers/guides/advanced/stake-weighted-qos)
