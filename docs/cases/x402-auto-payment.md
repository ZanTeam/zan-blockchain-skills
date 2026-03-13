# 场景 5：x402 自动支付示例

> **注意：x402 支付功能正在开发中，暂未开放。以下内容为预览版本，待功能上线后正式启用。**

## 场景描述

AI Agent 需要自动完成链上数据查询并支付，无需人工干预。典型场景包括：定时监控地址余额变化、自动拉取区块数据、Agent 驱动的链上分析等。x402 协议支持在收到 402 响应时自动完成链上 USDC 支付并重试请求。

## 用户类型

Agent 开发者

## 触发语句示例

- 「我需要 Agent 自动监控地址余额变化」
- 「实现一个定时查询余额的脚本，用 x402 支付」
- 「Agent 要自动调用链上 API，不要人工付钱」

## 推荐方式

**x402 Payment**

## 关键点

- **自动支付**：SDK 在信用不足时自动发起 USDC 支付并重试
- **无需人工干预**：配置好钱包私钥后，Agent 可 7×24 运行
- **即用即付**：按实际调用量付费，无需预充值或订阅

## 完整实现步骤

### 1. 安装 x402 SDK

```bash
npm install x402-axios axios
```

### 2. 配置钱包

- 使用支持 EVM 的钱包（如 MetaMask 导出私钥）
- 在 Base Mainnet 或 Base Sepolia 上持有 USDC
- 通过环境变量配置私钥：`WALLET_PRIVATE_KEY` 或 `PRIVATE_KEY`

### 3. 实现自动监控逻辑（定时查询余额）

使用 `setInterval` 或 `node-cron` 等实现定时任务，每次调用时若信用不足，SDK 会自动完成支付。

### 4. 处理支付与错误

- 402 / Insufficient Credits：SDK 自动处理
- 钱包 USDC 不足：需捕获并告警
- 网络异常：实现重试与退避

## 完整 TypeScript 代码示例（带定时监控功能）

```typescript
import { createX402Client } from "x402-axios";

// 配置
const MONITOR_INTERVAL_MS = 60 * 1000; // 每 60 秒查询一次
const TARGET_ADDRESS = "0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb";
const RPC_URL = "https://api.zan.top/x402/eth-mainnet/rpc";

// 初始化 x402 客户端（自动处理 402 支付）
const client = createX402Client({
  privateKey: process.env.WALLET_PRIVATE_KEY!,
  chainId: 8453, // Base Mainnet，用于支付
  network: "base-mainnet",
  retry: {
    maxRetries: 3,
    retryDelay: 2000,
  },
});

let lastBalance: bigint | null = null;

async function fetchBalance(): Promise<bigint> {
  const { data } = await client.post(RPC_URL, {
    jsonrpc: "2.0",
    method: "eth_getBalance",
    params: [TARGET_ADDRESS, "latest"],
    id: 1,
  });

  if (data.error) {
    throw new Error(data.error.message || "RPC 调用失败");
  }

  return BigInt(data.result);
}

async function monitorBalance() {
  try {
    const balanceWei = await fetchBalance();
    const balanceEth = Number(balanceWei) / 1e18;

    console.log(`[${new Date().toISOString()}] 余额: ${balanceEth} ETH`);

    if (lastBalance !== null) {
      const diff = balanceWei - lastBalance;
      if (diff !== 0n) {
        const diffEth = Number(diff) / 1e18;
        console.log(
          `  ⚠️ 余额变化: ${diff > 0n ? "+" : ""}${diffEth} ETH`
        );
        // 可在此触发告警、Webhook、数据库记录等
      }
    }

    lastBalance = balanceWei;
  } catch (err) {
    console.error("监控失败:", err);
    // 钱包 USDC 不足时，可能抛出错误，需告警
    if (
      err instanceof Error &&
      (err.message.includes("insufficient") || err.message.includes("balance"))
    ) {
      console.error("请检查钱包 USDC 余额是否充足");
    }
  }
}

// 启动定时监控
console.log(`开始监控地址 ${TARGET_ADDRESS}，间隔 ${MONITOR_INTERVAL_MS / 1000}s`);
monitorBalance(); // 立即执行一次
setInterval(monitorBalance, MONITOR_INTERVAL_MS);
```

### 使用 wrapAxios 的替代写法

若项目使用 `wrapAxios` 封装 axios：

```typescript
import { wrapAxios } from "x402-axios";
import axios from "axios";

const client = wrapAxios(axios, {
  privateKey: process.env.WALLET_PRIVATE_KEY!,
});

const response = await client.post(
  "https://api.zan.top/x402/eth-mainnet/rpc",
  {
    jsonrpc: "2.0",
    method: "eth_getBalance",
    params: [TARGET_ADDRESS, "latest"],
    id: 1,
  }
);

const balanceWei = BigInt(response.data.result);
console.log("余额:", Number(balanceWei) / 1e18, "ETH");
```

## 费用估算

| 网络类型 | 信用包价格 | 信用额度 | 单次最小支付 |
|----------|------------|----------|--------------|
| Base Mainnet（正式） | $0.15 USDC | 1M credits | 约 $0.0015 |
| Base Sepolia（测试） | $0.10 USDC | 0.1M credits | 约 $0.01 |

- 单次 `eth_getBalance` 调用消耗 credits 较少
- 每分钟 1 次、每天 1440 次，日费用通常 < $1（取决于实际计费）
- 建议先在 Base Sepolia 测试网验证逻辑

## 安全注意事项

| 事项 | 说明 |
|------|------|
| **私钥保护** | 切勿硬编码私钥，使用 `process.env` 或密钥管理服务（如 AWS Secrets Manager） |
| **独立钱包** | 使用专门用于 x402 支付的钱包，避免使用存有大量资产的主钱包 |
| **USDC 额度** | 仅存入预计消耗的 USDC，降低被盗风险 |
| **测试网优先** | 先在 Base Sepolia 验证逻辑，再切换 Base Mainnet |
| **监控告警** | 对支付失败、余额不足等异常设置告警，避免静默失败 |

## 相关文档

- [x402 快速开始](../getting-started/quick-start-x402.md)
- [x402 SDK 使用指南](../x402/x402-sdk-guide.md)
- [x402 错误处理](../x402/x402-error-handling.md)
- [eth_getBalance](../chain-rpc/eth_getBalance.md)
