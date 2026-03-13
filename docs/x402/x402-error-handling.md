# x402 错误处理指南

> **注意：x402 支付功能正在开发中，暂未开放。以下内容为预览版本，待功能上线后正式启用。**

## 常见错误列表

| 错误码 | 说明 | 处理建议 |
|--------|------|---------|
| 402 Payment Required | 需要支付 | SDK 自动处理；手动模式需提交支付 |
| 402 Insufficient Credits | 信用额度不足 | 购买更多信用额度 |
| 402 Invalid Signature | 签名无效 | 检查私钥配置 |
| 402 Facilitator Unavailable | 支付服务不可用 | 稍后重试 |
| 400 Bad Request | 请求参数错误 | 检查请求格式 |
| 429 Too Many Requests | 速率限制 | 降低请求频率 |
| 500 Internal Server Error | 服务端错误 | 稍后重试 |

## 错误处理最佳实践

### 基础错误捕获

```typescript
import { createX402Client } from 'x402-axios';

async function safeRpcCall() {
  const client = createX402Client({
    baseURL: 'https://api.zan.top/node/v1/eth/mainnet',
    privateKey: process.env.WALLET_PRIVATE_KEY!,
    network: 'base-sepolia',
  });

  try {
    const { data } = await client.post('/', {
      jsonrpc: '2.0',
      method: 'eth_blockNumber',
      params: [],
      id: 1,
    });
    return data.result;
  } catch (error: any) {
    if (error.response) {
      const status = error.response.status;
      const body = error.response.data;

      switch (status) {
        case 402:
          if (body?.code === 'InsufficientCredits') {
            console.error('信用额度不足，请购买更多 credits');
          } else if (body?.code === 'InvalidSignature') {
            console.error('签名无效，请检查 WALLET_PRIVATE_KEY 配置');
          } else if (body?.code === 'FacilitatorUnavailable') {
            console.error('支付服务暂时不可用，请稍后重试');
          }
          break;
        case 400:
          console.error('请求参数错误:', body?.message);
          break;
        case 429:
          console.error('请求过于频繁，请降低调用频率');
          break;
        case 500:
          console.error('服务端错误，请稍后重试');
          break;
      }
    }
    throw error;
  }
}
```

### 按错误类型分类处理

```typescript
function handleX402Error(error: any): void {
  const status = error.response?.status;
  const code = error.response?.data?.code;

  // 可重试错误
  const retryable = [429, 500, 502, 503];
  if (status && retryable.includes(status)) {
    console.warn(`可重试错误 (${status})，建议稍后重试`);
    return;
  }

  // 配置错误，需人工介入
  if (status === 402 && code === 'InvalidSignature') {
    throw new Error('请检查 WALLET_PRIVATE_KEY 是否正确配置');
  }

  // 额度不足
  if (status === 402 && code === 'InsufficientCredits') {
    throw new Error('信用额度不足，请购买更多 credits');
  }
}
```

## 重试策略

### 指数退避重试

```typescript
async function rpcWithRetry(
  client: ReturnType<typeof createX402Client>,
  payload: object,
  maxRetries = 3
) {
  let lastError: any;
  for (let i = 0; i < maxRetries; i++) {
    try {
      const { data } = await client.post('/', payload);
      return data;
    } catch (error: any) {
      lastError = error;
      const status = error.response?.status;

      // 仅对可重试错误进行重试
      if (![429, 500, 502, 503].includes(status)) {
        throw error;
      }

      const delay = Math.pow(2, i) * 1000; // 1s, 2s, 4s
      console.warn(`第 ${i + 1} 次失败，${delay}ms 后重试...`);
      await new Promise((r) => setTimeout(r, delay));
    }
  }
  throw lastError;
}
```

### 使用 SDK 内置重试

若 SDK 支持，可启用内置重试：

```typescript
const client = createX402Client({
  baseURL: 'https://api.zan.top/node/v1/eth/mainnet',
  privateKey: process.env.WALLET_PRIVATE_KEY!,
  network: 'base-sepolia',
  retry: {
    maxRetries: 3,
    retryDelay: 1000,
    retryCondition: (error) => {
      const status = error.response?.status;
      return [429, 500, 502, 503].includes(status);
    },
  },
});
```

## 调试技巧

| 技巧 | 说明 |
|------|------|
| 开启详细日志 | 设置 `DEBUG=x402:*` 或 SDK 的 `debug: true` 查看请求与支付流程 |
| 检查网络 | 确认 RPC 请求能到达 `api.zan.top`，排除网络问题 |
| 验证私钥 | 使用测试网小额支付验证私钥和网络配置 |
| 检查余额 | 确认钱包有足够 USDC 支付（Base Mainnet 需真实 USDC，Sepolia 需测试 USDC） |
| 查看响应头 | 402 响应中的 `X-Payment-*` 头包含支付指令，可用于排查支付逻辑 |

### 调试示例

```typescript
// 开启调试模式
const client = createX402Client({
  baseURL: 'https://api.zan.top/node/v1/eth/mainnet',
  privateKey: process.env.WALLET_PRIVATE_KEY!,
  network: 'base-sepolia',
  debug: true, // 打印请求、支付、重试等日志
});
```

## 相关文档

- [x402 概述](./README.md)
- [x402 SDK 使用指南](./x402-sdk-guide.md)
- [x402 vs API Key 对比](./x402-vs-apikey.md)
