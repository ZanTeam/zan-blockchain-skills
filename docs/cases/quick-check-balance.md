# 场景 1：快速查询账户余额

## 场景描述

用户需要快速查询某个地址的 ETH 余额，用于验证转账到账、检查 Gas 费是否充足或简单资产盘点。

## 用户类型

企业开发者

## 触发语句示例

- 「帮我查一下这个地址的 ETH 余额，我有 API Key」
- 「0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb 有多少 ETH？」
- 「检查这个钱包的余额是否足够支付 Gas」

## 推荐方式

**API Key**

## 完整实现步骤

### 1. 准备 API Key

在 [ZAN 控制台](https://zan.top) 注册并创建项目，获取 API Key。将 API Key 配置到环境变量：

```bash
export ZAN_API_KEY="your-api-key"
```

### 2. 构造 eth_getBalance 请求

使用 JSON-RPC 格式，`method` 为 `eth_getBalance`，`params` 为 `[address, blockTag]`：

- `address`：要查询的地址（0x 开头的 20 字节格式）
- `blockTag`：区块标签，通常使用 `"latest"` 表示最新区块

### 3. 发送请求

向 ZAN RPC 端点发送 POST 请求，将 `eth/mainnet` 替换为目标链（如 `base/mainnet`、`arbitrum/mainnet`）。

### 4. 解析结果（wei 转 ETH）

返回的 `result` 为十六进制字符串，单位为 wei。需转换为 ETH：`1 ETH = 10^18 wei`。

## 代码示例

### curl

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

### TypeScript（API Key）

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

### wei 转 ETH 工具函数

```typescript
function weiToEth(weiHex: string): number {
  return Number(BigInt(weiHex)) / 1e18;
}
```

## 预期输出

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": "0x1bc16d674ec80000"
}
```

`0x1bc16d674ec80000` = 2000000000000000000 wei = **2 ETH**。

控制台输出示例：

```
余额: 2 ETH
```

## 扩展场景：查询 ERC20 Token 余额

若需查询 USDT、USDC 等 ERC20 代币余额，需使用 `eth_call` 调用合约的 `balanceOf(address)` 方法。

### 函数选择器

`balanceOf(address)` 的选择器为 `0x70a08231`。

### TypeScript 示例（查询 USDT 余额）

```typescript
import { ethers } from "ethers";

const API_KEY = process.env.ZAN_API_KEY;
const endpoint = `https://api.zan.top/node/v1/eth/mainnet/${API_KEY}`;

// USDT 合约地址 (Ethereum Mainnet)
const USDT_ADDRESS = "0xdAC17F958D2ee523a2206206994597C13D831ec7";
const holderAddress = "0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb";

const iface = new ethers.Interface([
  "function balanceOf(address) view returns (uint256)",
]);
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
// USDT 为 6 位小数
const balance = Number(balanceWei) / 1e6;
console.log(`USDT 余额: ${balance}`);
```

> 不同代币的 `decimals` 不同：USDT/USDC 通常为 6，ETH 为 18，需根据合约调整除数。

## 相关文档

- [eth_getBalance](../chain-rpc/eth_getBalance.md)
- [eth_call](../chain-rpc/eth_call.md)
- [API Key 快速开始](../getting-started/quick-start-api-key.md)
