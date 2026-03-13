# ZAN Advanced API（Data 分类）

## 定位

**ZAN Advanced API** 提供链上数据聚合、分析与结构化输出能力。帮助您快速获取 Token 元数据、NFT 持有者、地址行为、交易调用链等结构化数据，无需自行解析原始区块或事件日志。

## 支持的鉴权方式

<!-- x402 支付方式正在开发中，后续将在此处补充 x402 相关内容 -->

| 鉴权方式 | 适用场景 |
|----------|----------|
| **API Key** | 企业客户、高频调用、需要完整功能 |

## 统一端点格式

所有 Advanced API 均使用 **POST** 方法，**JSON-RPC 2.0** 格式：

```
https://api.zan.top/data/v1/{chain}/{network}/{apiKey}
```

示例：

- Ethereum 主网：`https://api.zan.top/data/v1/eth/mainnet/{apiKey}`
- BNB Smart Chain：`https://api.zan.top/data/v1/bsc/mainnet/{apiKey}`
- Polygon：`https://api.zan.top/data/v1/polygon/mainnet/{apiKey}`
- Arbitrum：`https://api.zan.top/data/v1/arbitrum/mainnet/{apiKey}`
- Optimism：`https://api.zan.top/data/v1/optimism/mainnet/{apiKey}`
- Base：`https://api.zan.top/data/v1/base/mainnet/{apiKey}`

## 支持的链

Ethereum、BNB Smart Chain、Polygon、Arbitrum、Optimism、Base 等 EVM 兼容链。部分 API 支持 Bitcoin 生态，详见官方文档。

## 通用请求格式

```json
{
  "id": 1,
  "jsonrpc": "2.0",
  "method": "zan_getNFTHolders",
  "params": ["0xContractAddress", 100]
}
```

| 字段 | 类型 | 说明 |
|------|------|------|
| `id` | number | 请求 ID，用于匹配响应 |
| `jsonrpc` | string | 固定为 `"2.0"` |
| `method` | string | API 方法名 |
| `params` | array | 方法参数，按文档顺序传入 |

## API 分组与方法列表

### 1. NFT API（EVM 兼容）

| 方法名 | 说明 |
|--------|------|
| `zan_getNFTMetadata` | 查询 NFT 元数据 |
| `zan_getNFTsByOwner` | 按持有者查询 NFT 列表 |
| `zan_getNftIDs` | 查询 NFT 合约的 tokenId 列表 |
| `zan_verifyNFTHolder` | 验证地址是否为 NFT 持有者 |
| `zan_getNFTHolders` | 查询 NFT 持有者列表 |
| `zan_getNftIDHolders` | 查询指定 tokenId 的持有者 |
| `zan_getNftCollectionHolders` | 查询 NFT 合集持有者 |
| `zan_getNftTransfers` | 查询 NFT 转账记录 |

**详细文档**：[nft-holders.md](./nft-holders.md)

### 2. Token API

| 方法名 | 说明 |
|--------|------|
| `zan_getTokenMetadata` | 查询 Token 元数据（name, symbol, decimal, ecosystem） |
| `zan_getTokenBalanceByOwner` | 查询指定地址的 Token 余额 |
| `zan_getTokensByOwner` | 查询指定地址持有的所有 Token |
| `zan_getTokenHoldersCount` | 查询 Token 持有者数量 |
| `zan_getTokenHolders` | 查询 Token 持有者列表（按持有比例排序） |
| `zan_getApprovalListByAddress` | 查询指定地址的 Token 授权列表 |
| `zan_getApprovalListByToken` | 查询指定 Token 的授权列表 |

**详细文档**：[token-transfers.md](./token-transfers.md)

### 3. Simulation API

| 方法名 | 说明 |
|--------|------|
| `zan_simulateAssetChanges` | 模拟资产变更 |
| `zan_simulateExecution` | 模拟交易执行 |

**详细文档**：[transaction-trace.md](./transaction-trace.md)

### 4. Debug API

| 方法名 | 说明 |
|--------|------|
| `debug_executionWitness` | 获取执行见证 |
| `debug_traceBlockByHash` | 按区块哈希追踪 |
| `debug_traceBlockByNumber` | 按区块号追踪 |
| `debug_traceCall` | 追踪模拟调用 |
| `debug_traceTransaction` | 追踪交易执行 |

**详细文档**：[transaction-trace.md](./transaction-trace.md)

### 5. 其他 API

- **Bitcoin NFT API**、**Bitcoin Account API**：详见 [ZAN 官方文档](https://docs.zan.top/)

## 文档索引

| 文档 | 覆盖内容 |
|------|----------|
| [token-transfers.md](./token-transfers.md) | Token API 完整方法说明与示例 |
| [nft-holders.md](./nft-holders.md) | NFT API 持有者与元数据查询 |
| [transaction-trace.md](./transaction-trace.md) | Debug / Simulation API 调用链与模拟 |
| [address-activity.md](./address-activity.md) | 综合查询与地址活动分析 |

## Agent 使用建议

当用户目标是「**分析、统计、追踪、聚合数据**」时，优先使用 Advanced API：

- **分析**：用户想了解某地址行为、某合约交易模式、某笔交易的执行路径
- **统计**：用户想统计 Token 流向、持有者分布、交易次数等
- **追踪**：用户想追踪资金流向、某笔交易的调用链、某地址的交易历史
- **聚合**：用户需要获取 NFT 持有者列表、Token 元数据、授权列表等结构化数据

若用户目标是「查询余额、区块状态、单次合约调用」等基础交互，应使用 [Chain RPC](../chain-rpc/README.md) 分类。
