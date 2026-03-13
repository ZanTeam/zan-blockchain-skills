# 地址综合查询

## API Name

Address Activity（综合查询）

## Category

Data — Advanced API

## Description

通过组合多个 ZAN Advanced API 方法，实现对指定地址的链上活动综合分析。包括查询地址持有的 Token 列表、NFT 列表、Token 授权情况等，帮助构建完整的地址画像。

本文档介绍如何组合以下 API 实现地址综合查询：

| 方法 | 用途 |
| --- | --- |
| `zan_getTokensByOwner` | 查询地址持有的所有 Token |
| `zan_getTokenBalanceByOwner` | 查询地址的指定 Token 余额 |
| `zan_getNFTsByOwner` | 查询地址持有的所有 NFT |
| `zan_getApprovalListByAddress` | 查询地址的 Token 授权列表 |

## Supported Auth Methods

| 鉴权方式 | 支持 |
| --- | --- |
| API Key | ✅ |

## Endpoint

```
POST https://api.zan.top/data/v1/{chain}/{network}/{apiKey}
```

所有方法使用统一端点，通过 `method` 字段区分调用的 API。

## Use Cases

- **地址画像**：全面了解某地址的 Token 和 NFT 持有情况
- **风险评估**：通过授权列表检查潜在的安全风险
- **资产统计**：汇总地址的所有链上资产
- **钱包分析**：分析巨鲸或关键地址的持仓分布

---

## 方法详解

### 1. zan_getTokensByOwner

查询指定地址持有的所有 Token 列表。

**Parameters:**

| 索引 | 参数 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- | --- |
| 0 | `ownerAddress` | string | 是 | 要查询的钱包地址 |

**curl 示例：**

```bash
curl -X POST 'https://api.zan.top/data/v1/eth/mainnet/{YOUR_API_KEY}' \
  -H 'Content-Type: application/json' \
  -d '{
    "id": 1,
    "jsonrpc": "2.0",
    "method": "zan_getTokensByOwner",
    "params": ["0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb"]
  }'
```

---

### 2. zan_getTokenBalanceByOwner

查询指定地址在特定 Token 合约中的余额。

**Parameters:**

| 索引 | 参数 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- | --- |
| 0 | `ownerAddress` | string | 是 | 钱包地址 |
| 1 | `contractAddress` | string | 是 | Token 合约地址 |

**curl 示例：**

```bash
curl -X POST 'https://api.zan.top/data/v1/eth/mainnet/{YOUR_API_KEY}' \
  -H 'Content-Type: application/json' \
  -d '{
    "id": 1,
    "jsonrpc": "2.0",
    "method": "zan_getTokenBalanceByOwner",
    "params": [
      "0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb",
      "0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48"
    ]
  }'
```

---

### 3. zan_getNFTsByOwner

查询指定地址持有的所有 NFT。

**Parameters:**

| 索引 | 参数 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- | --- |
| 0 | `ownerAddress` | string | 是 | 钱包地址 |

**curl 示例：**

```bash
curl -X POST 'https://api.zan.top/data/v1/eth/mainnet/{YOUR_API_KEY}' \
  -H 'Content-Type: application/json' \
  -d '{
    "id": 1,
    "jsonrpc": "2.0",
    "method": "zan_getNFTsByOwner",
    "params": ["0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb"]
  }'
```

---

### 4. zan_getApprovalListByAddress

查询指定地址的 Token 授权（Approval）列表，帮助用户了解哪些合约被授权操作其资产。

**Parameters:**

| 索引 | 参数 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- | --- |
| 0 | `address` | string | 是 | 钱包地址 |

**curl 示例：**

```bash
curl -X POST 'https://api.zan.top/data/v1/eth/mainnet/{YOUR_API_KEY}' \
  -H 'Content-Type: application/json' \
  -d '{
    "id": 1,
    "jsonrpc": "2.0",
    "method": "zan_getApprovalListByAddress",
    "params": ["0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb"]
  }'
```

---

## 综合代码示例

以下示例展示如何组合多个 API 构建完整的地址活动分析。

### TypeScript

```typescript
const API_KEY = process.env.ZAN_API_KEY!;
const ENDPOINT = `https://api.zan.top/data/v1/eth/mainnet/${API_KEY}`;
const TARGET_ADDRESS = "0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb";

async function rpcCall(method: string, params: unknown[]) {
  const response = await fetch(ENDPOINT, {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({
      id: 1,
      jsonrpc: "2.0",
      method,
      params,
    }),
  });
  const data = await response.json();
  if (data.error) throw new Error(data.error.message);
  return data.result;
}

async function analyzeAddress(address: string) {
  console.log(`=== 地址分析: ${address} ===\n`);

  // 1. 查询持有的 Token 列表
  const tokens = await rpcCall("zan_getTokensByOwner", [address]);
  console.log(`持有 Token 数量: ${Array.isArray(tokens) ? tokens.length : "N/A"}`);
  console.log("Token 列表:", tokens);

  // 2. 查询持有的 NFT 列表
  const nfts = await rpcCall("zan_getNFTsByOwner", [address]);
  console.log(`\n持有 NFT 数量: ${Array.isArray(nfts) ? nfts.length : "N/A"}`);
  console.log("NFT 列表:", nfts);

  // 3. 查询授权列表（安全审计）
  const approvals = await rpcCall("zan_getApprovalListByAddress", [address]);
  console.log(`\n授权记录数: ${Array.isArray(approvals) ? approvals.length : "N/A"}`);
  console.log("授权列表:", approvals);

  // 4. 对特定 Token 查询余额（以 USDC 为例）
  const USDC_ADDRESS = "0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48";
  try {
    const balance = await rpcCall("zan_getTokenBalanceByOwner", [address, USDC_ADDRESS]);
    console.log(`\nUSDC 余额:`, balance);
  } catch (e) {
    console.log("\nUSDC 余额查询失败:", e);
  }
}

analyzeAddress(TARGET_ADDRESS);
```

## Error Codes

| HTTP Status | 说明 | 处理建议 |
| --- | --- | --- |
| 200 + error 字段 | JSON-RPC 错误 | 检查 method 和 params 格式 |
| 401 | API Key 无效 | 检查 API Key 是否正确 |
| 429 | 请求频率超限 | 降低调用频率 |
| 500 | 服务端错误 | 稍后重试 |

## Notes for Agents

- **组合查询**：地址综合分析需组合多个 API 调用，建议并发请求以提高效率
- **链的选择**：端点中的 `{chain}/{network}` 需根据用户指定的链替换，如 `eth/mainnet`、`polygon/mainnet`
- **授权风险**：`zan_getApprovalListByAddress` 可用于安全审计，帮助用户识别可疑授权
- **适用场景**：当用户提到「查看地址资产」「分析钱包」「授权检查」「持仓分布」时使用
- **与 Chain RPC 区别**：Chain RPC 的 `eth_getBalance` 仅查原生币余额，本组 API 可查 Token / NFT 等全量资产

## 参考资料

- [ZAN Advanced API 文档](https://docs.zan.top/reference/zan_gettokenmetadata-advanced)
- [Token API 详细文档](./token-transfers.md)
- [NFT API 详细文档](./nft-holders.md)
