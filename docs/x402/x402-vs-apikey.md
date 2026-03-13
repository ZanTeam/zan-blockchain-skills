# x402 vs API Key 对比

> **注意：x402 支付功能正在开发中，暂未开放。以下内容为预览版本，待功能上线后正式启用。**

## 全面对比

| 维度 | API Key | x402 Payment |
|------|---------|-------------|
| 注册要求 | 需要 ZAN 账户 | 无需注册 |
| 支付方式 | 订阅 / 按量计费 | 链上 USDC 即时支付 |
| 适合人群 | 企业、高频开发者 | AI Agent、低频 / 隐私用户 |
| 功能覆盖 | 全部功能 | Chain RPC / Data / Operational |
| 技术支持 | 有 | 社区支持 |
| 接入速度 | 注册后可用 | 5 分钟接入 |
| 隐私 | 需个人信息 | 无需个人信息 |
| 自动化 | 需管理 Key | 自动支付 |

## 选择建议

### 按用户类型推荐

| 用户类型 | 推荐方式 | 理由 |
|---------|---------|------|
| AI Agent 开发者 | x402 | Agent 可自主支付，无需人工注册 |
| 企业 / 高频用户 | API Key | 订阅更划算，有技术支持 |
| 低频 / 偶尔查询 | x402 | 即用即付，无需订阅 |
| 隐私敏感用户 | x402 | 无需提供邮箱、手机等 |
| 快速原型 / POC | x402 | 5 分钟接入，快速验证 |
| 生产环境长期使用 | API Key | 稳定计费，功能更全 |

### 按场景推荐

| 场景 | 推荐方式 |
|------|---------|
| 自动化脚本 / Agent 批量调用 | x402 |
| 需要发票 / 对公结算 | API Key |
| 临时测试、演示 | x402 |
| 需要 SLA、专属支持 | API Key |

## 迁移指南

### 从 API Key 迁移到 x402

1. 安装 x402 SDK：`npm install x402-axios axios`
2. 准备钱包：创建或使用现有钱包，确保有 USDC（测试网用 Base Sepolia 测试币）
3. 替换客户端：将 `axios` 或带 `Authorization` 的请求改为 `createX402Client`
4. 移除 API Key：删除请求头中的 `Authorization: Bearer <api_key>`
5. 在测试网验证：先在 Base Sepolia 上跑通流程，再切到 Base Mainnet

```typescript
// 之前 (API Key)
const response = await axios.post(url, payload, {
  headers: { Authorization: `Bearer ${API_KEY}` },
});

// 之后 (x402)
const client = createX402Client({
  baseURL: url,
  privateKey: process.env.WALLET_PRIVATE_KEY!,
  network: 'base-sepolia',
});
const response = await client.post('/', payload);
```

### 从 x402 迁移到 API Key

1. 注册 ZAN 账户：访问 ZAN 控制台完成注册
2. 创建 API Key：在控制台创建并保存 Key
3. 替换客户端：将 `createX402Client` 改回 `axios`，并添加 `Authorization` 头
4. 配置计费：选择订阅或按量计费
5. 逐步切换：可先保留 x402 作为备用，验证 API Key 稳定后再完全切换

```typescript
// 之前 (x402)
const client = createX402Client({ ... });
const response = await client.post('/', payload);

// 之后 (API Key)
const response = await axios.post(url, payload, {
  headers: { Authorization: `Bearer ${process.env.ZAN_API_KEY}` },
});
```

## FAQ

### x402 支持哪些链？

Phase 1 支持 Base Mainnet 和 Base Sepolia；Phase 2 计划支持 Ethereum、Polygon、BNB Smart Chain、Arbitrum One、Optimism。

### x402 和 API Key 可以同时使用吗？

可以。同一项目可同时配置 API Key 和 x402，按场景选择使用。

### x402 支付失败会扣款吗？

只有链上交易确认成功后才会扣款。若支付失败或超时，不会扣款。

### 测试网 USDC 从哪里获取？

Base Sepolia 测试网 USDC 可通过 Base Sepolia 水龙头或测试网 Faucet 获取。

### x402 有速率限制吗？

有。具体限制与 API Key 类似，详见 ZAN 文档。若返回 429，请降低请求频率并配合重试策略。

### 私钥丢失怎么办？

x402 使用您自己的钱包，私钥由您保管。私钥丢失无法恢复，请妥善备份。

## 相关文档

- [x402 概述](./README.md)
- [x402 SDK 使用指南](./x402-sdk-guide.md)
- [x402 错误处理](./x402-error-handling.md)
- [快速开始 - API Key](../getting-started/quick-start-api-key.md)
- [快速开始 - x402](../getting-started/quick-start-x402.md)
