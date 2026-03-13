# x402 SDK 使用指南

> **注意：x402 支付功能正在开发中，暂未开放。以下内容为预览版本，待功能上线后正式启用。**

## SDK 概述

`x402-axios` 是基于 axios 的 x402 协议封装，在发起 HTTP 请求时自动处理 402 Payment Required 响应，完成链上 USDC 支付并重试请求。

## 安装

```bash
npm install x402-axios axios
```

## 配置

### 私钥配置

通过环境变量 `WALLET_PRIVATE_KEY` 配置钱包私钥：

```bash
export WALLET_PRIVATE_KEY="0x你的私钥"
```

> **安全提示**：切勿在代码中硬编码私钥，请使用环境变量或密钥管理服务。

### 网络选择

| 网络 | 用途 |
|------|------|
| Base Mainnet | 正式环境，消耗真实 USDC |
| Base Sepolia | 测试环境，使用测试网 USDC |

## 基本用法

### 初始化客户端

```typescript
import { createX402Client } from 'x402-axios';

const client = createX402Client({
  baseURL: 'https://api.zan.top/node/v1/eth/mainnet',
  privateKey: process.env.WALLET_PRIVATE_KEY!,
  network: 'base-sepolia', // 或 'base-mainnet'
});
```

### 发起 RPC 调用

```typescript
const response = await client.post('/', {
  jsonrpc: '2.0',
  method: 'eth_blockNumber',
  params: [],
  id: 1,
});
```

### 自动支付流程

当服务端返回 402 时，SDK 会：

1. 解析支付指令（金额、收款地址、签名等）
2. 使用配置的私钥发起链上 USDC 转账
3. 等待交易确认
4. 自动重试原请求并返回结果

## 完整代码示例

### 查询区块高度

```typescript
import { createX402Client } from 'x402-axios';

async function getBlockNumber() {
  const client = createX402Client({
    baseURL: 'https://api.zan.top/node/v1/eth/mainnet',
    privateKey: process.env.WALLET_PRIVATE_KEY!,
    network: 'base-sepolia',
  });

  const { data } = await client.post('/', {
    jsonrpc: '2.0',
    method: 'eth_blockNumber',
    params: [],
    id: 1,
  });

  console.log('当前区块高度:', data.result);
  return data.result;
}
```

### 查询余额

```typescript
async function getBalance(address: string) {
  const client = createX402Client({
    baseURL: 'https://api.zan.top/node/v1/eth/mainnet',
    privateKey: process.env.WALLET_PRIVATE_KEY!,
    network: 'base-sepolia',
  });

  const { data } = await client.post('/', {
    jsonrpc: '2.0',
    method: 'eth_getBalance',
    params: [address, 'latest'],
    id: 1,
  });

  console.log('余额 (wei):', data.result);
  return data.result;
}
```

### 合约调用

```typescript
async function callContract(to: string, data: string) {
  const client = createX402Client({
    baseURL: 'https://api.zan.top/node/v1/eth/mainnet',
    privateKey: process.env.WALLET_PRIVATE_KEY!,
    network: 'base-sepolia',
  });

  const { data: result } = await client.post('/', {
    jsonrpc: '2.0',
    method: 'eth_call',
    params: [{ to, data }, 'latest'],
    id: 1,
  });

  return result.result;
}
```

## 高级配置

### 自定义支付金额

```typescript
const client = createX402Client({
  baseURL: 'https://api.zan.top/node/v1/eth/mainnet',
  privateKey: process.env.WALLET_PRIVATE_KEY!,
  network: 'base-sepolia',
  minPaymentAmount: '0.0015', // 正式网络最小支付 (USDC)
});
```

### 自动重试

```typescript
const client = createX402Client({
  baseURL: 'https://api.zan.top/node/v1/eth/mainnet',
  privateKey: process.env.WALLET_PRIVATE_KEY!,
  network: 'base-sepolia',
  retry: {
    maxRetries: 3,
    retryDelay: 1000,
  },
});
```

### 超时设置

```typescript
const client = createX402Client({
  baseURL: 'https://api.zan.top/node/v1/eth/mainnet',
  privateKey: process.env.WALLET_PRIVATE_KEY!,
  network: 'base-sepolia',
  timeout: 30000, // 30 秒
});
```

## 安全最佳实践

| 实践 | 说明 |
|------|------|
| 不要硬编码私钥 | 使用 `process.env.WALLET_PRIVATE_KEY` 或密钥管理服务 |
| 使用环境变量 | 将私钥放在 `.env` 中，并加入 `.gitignore` |
| 测试网优先验证 | 先在 Base Sepolia 上验证逻辑，再切换到 Base Mainnet |
| 最小权限原则 | 使用专门用于 x402 支付的独立钱包，避免使用主钱包 |

## 相关文档

- [x402 概述](./README.md)
- [x402 错误处理](./x402-error-handling.md)
- [快速开始 - x402](../getting-started/quick-start-x402.md)
