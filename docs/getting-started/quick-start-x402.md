# x402 快速开始

> **注意：x402 支付功能正在开发中，暂未开放。以下内容为预览版本，待功能上线后正式启用。**

本指南帮助您通过 x402 支付方式快速接入 ZAN 区块链服务，**无需注册 ZAN 账户**，使用钱包中的 USDC 即可按需付费。

## 什么是 x402？

x402 是一种基于 HTTP 402 Payment Required 的支付协议，允许客户端在请求 API 时自动完成链上支付。您只需配置钱包私钥，SDK 会在需要时自动发起 USDC 支付，无需人工干预，非常适合 AI Agent 与自动化场景。

## 前置条件

- **钱包**：支持 EVM 的钱包（如 MetaMask）
- **USDC**：在 Base Mainnet（正式网）或 Base Sepolia（测试网）上有 USDC
- **Node.js**：v18 或更高版本

## 步骤 1：安装 x402 SDK

```bash
npm install x402-axios
```

## 步骤 2：配置钱包私钥

通过环境变量配置私钥（**切勿将私钥提交到代码仓库**）：

```bash
export PRIVATE_KEY=your_wallet_private_key_here
```

或在 `.env` 文件中：

```
PRIVATE_KEY=your_wallet_private_key_here
```

## 步骤 3：发起首次调用（自动支付）

当您发起请求时，若账户信用不足，SDK 会自动完成 USDC 支付并重试请求。

### TypeScript 示例

```typescript
import { createX402Client } from 'x402-axios';

const client = createX402Client({
  privateKey: process.env.PRIVATE_KEY!,
  chainId: 8453, // Base Mainnet
});

const response = await client.post(
  'https://api.zan.top/x402/base-mainnet/rpc',
  {
    jsonrpc: '2.0',
    method: 'eth_blockNumber',
    params: [],
    id: 1,
  },
  {
    headers: { 'Content-Type': 'application/json' },
  }
);

console.log('当前区块高度:', response.data.result);
```

首次调用时，若信用不足，SDK 会自动购买信用包并完成请求。

## 定价说明

| 网络类型 | 价格 | 信用额度 | 说明 |
|----------|------|----------|------|
| 正式网络 | $0.15 USDC | 1M credits | Base Mainnet |
| 测试网络 | $0.10 USDC | 0.1M credits | Base Sepolia |

**默认信用包**：10,000 credits

- 正式网络：单次最小支付约 $0.0015 USDC
- 测试网络：单次最小支付约 $0.01 USDC

## 支持的网络

### Phase 1（当前可用）

- **Base Mainnet**（正式网）
- **Base Sepolia**（测试网）

### Phase 2（规划中）

- Ethereum Mainnet
- Polygon Mainnet
- BNB Smart Chain
- Arbitrum One
- Optimism

## 错误处理基础

常见 x402 相关错误：

| 错误 | 说明 | 处理建议 |
|------|------|----------|
| `402 Payment Required` | 需要支付 | SDK 会自动处理；若自动支付失败，检查钱包 USDC 余额 |
| `Insufficient Credits` | 信用不足 | 确保钱包有足够 USDC，SDK 会尝试自动购买 |
| `Invalid Signature` | 签名无效 | 检查私钥配置是否正确 |
| `Facilitator Unavailable` | 支付通道不可用 | 稍后重试或切换网络 |

## 下一步

- 查看 [x402 Payment](../x402/README.md) 分类，了解完整 x402 能力
- 查看 [x402 SDK 使用指南](../x402/x402-sdk-guide.md)
- 查看 [x402 错误处理](../x402/x402-error-handling.md)
