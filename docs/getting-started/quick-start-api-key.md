# API Key 快速开始

本指南帮助您通过 API Key 方式快速接入 ZAN 区块链服务。

## 步骤 1：注册 ZAN 账户

1. 访问 [https://zan.top](https://zan.top)
2. 点击「注册」并完成账户创建
3. 登录 ZAN 控制台

## 步骤 2：创建项目并获取 API Key

1. 在控制台中创建新项目
2. 选择目标链（如 Ethereum、Base、Arbitrum 等）
3. 创建完成后，在项目详情页获取 **API Key**
4. 妥善保管 API Key，切勿泄露

## 步骤 3：发起首次 RPC 调用

以查询当前区块高度（`eth_blockNumber`）为例：

### curl 示例

```bash
curl -X POST https://api.zan.top/node/v1/<chain_id>/<project_id> \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -d '{
    "jsonrpc": "2.0",
    "method": "eth_blockNumber",
    "params": [],
    "id": 1
  }'
```

将 `YOUR_API_KEY`、`<chain_id>`、`<project_id>` 替换为您的实际值。

### JavaScript/TypeScript 示例

```typescript
const response = await fetch('https://api.zan.top/node/v1/<chain_id>/<project_id>', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'Authorization': 'Bearer YOUR_API_KEY',
  },
  body: JSON.stringify({
    jsonrpc: '2.0',
    method: 'eth_blockNumber',
    params: [],
    id: 1,
  }),
});

const data = await response.json();
console.log('当前区块高度:', data.result);
```

### 预期响应示例

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": "0x12a4b3c"
}
```

`result` 为十六进制格式的区块高度，可转换为十进制使用。

## 支持的链

ZAN 支持 20+ 条主流链，包括但不限于：

| 链名称 | chain_id 示例 |
|--------|---------------|
| Ethereum Mainnet | eth |
| Base Mainnet | base |
| Arbitrum One | arbitrum |
| Polygon | polygon |
| BNB Smart Chain | bsc |
| Optimism | optimism |

具体 chain_id 与 endpoint 格式请参考控制台中的项目配置。

## 下一步

- 查看 [Chain RPC](../chain-rpc/README.md) 分类，了解更多 RPC 方法（如 `eth_getBalance`、`eth_call` 等）
- 查看 [Data](../data/README.md) 分类，了解 Token 转账、NFT 持有者等数据能力
