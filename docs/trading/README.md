# Trading 分类 — Solana Trading Boost

> 基于 SWQoS 技术的 Solana 交易加速服务，显著提升交易成功率与速度

## 定位

Trading 分类提供 **Solana Trading Boost** 能力，利用 SWQoS（Stake-Weighted Quality of Service）技术与 ZAN 的质押验证节点，显著提升 Solana 链上交易的成功率和速度。

## 核心特性

| 特性 | 说明 |
| --- | --- |
| **优先通道** | 基于服务等级的专属带宽分配 |
| **超低延迟** | Prime 等级：95% 交易 ≤1 秒确认 |
| **MEV 保护** | 防止恶意抢跑交易（免费功能） |
| **实时分析** | 成功率、延迟、消耗等实时监控指标 |

## 技术原理

Trading Boost 基于 Solana 的 [SWQoS（Stake-Weighted Quality of Service）](https://solana.com/developers/guides/advanced/stake-weighted-qos) 机制实现。ZAN 通过其质押的验证节点获取优先交易通道，根据质押权重分配带宽资源，从而为用户提供更快、更可靠的交易确认体验。

## 支持的鉴权方式

| 鉴权方式 | 支持 | 说明 |
| --- | --- | --- |
| **API Key** | ✅ | 唯一支持的鉴权方式 |

## 两种集成方式

ZAN 提供 **Tips** 和 **Points** 两种集成方式，可根据业务需求灵活选择：

| 集成方式 | 描述 | 优势 | 适用场景 |
| --- | --- | --- | --- |
| **Tips** | 向指定账户支付 Tips（最低 0.0003 SOL）以加速交易，交易优先级由 Tips 出价决定 | 灵活竞价机制、单次支付门槛低、按实际加速需求付费 | 用户端高度自定义加速策略、复杂业务逻辑需要动态优先级调整 |
| **Points** | 预购 Points 积分，交易加速成功后异步扣除费用，优先级由预设的服务等级决定 | 固定透明定价、使用逻辑简单、系统依赖少 | 需求简单直接、偏好运营简洁与流程稳定的业务 |

## 服务等级（Points 方式）

| 等级 | 优先级 | Points 消耗 |
| --- | --- | --- |
| **Prime** | 专属通道 | 5 Points（$0.5） |
| **Turbo** | 高优先级 | 3 Points（$0.3） |
| **Standard** | 共享通道 | 1 Point（$0.1） |

> 1 Point = $0.1 USD。仅在交易成功上链后扣除 Points。余额不足时自动回退至标准（非加速）处理。

## MEV 保护

两种集成方式均支持免费的 MEV 保护功能。在端点 URL 后附加 `?anti-mev=true` 即可启用。

**注意事项：**
- MEV 保护的加密验证过程可能在网络高峰期引入微小延迟
- 建议通过 [ZAN Dashboard](https://zan.top/service/sol/overview) 监控实时延迟指标
- 建议对高价值交易启用，低价值交易可根据场景权衡

## 吞吐量限制

- Trading Boost 与 Chain RPC 服务共享 API Key 和吞吐量限制
- 超出限制的交易将自动回退至标准非加速处理
- 如需更高吞吐量，可在 [ZAN 控制台](https://zan.top/personal/price-plan?product=NODE_SERVICE) 升级 Chain RPC 订阅等级
- 或 [联系解决方案团队](https://zan.top/home/contact) 获取定制扩展方案

## API 列表

| API 名称 | 集成方式 | 描述 | 文档链接 |
| --- | --- | --- | --- |
| `sendTransaction`（Tips） | Tips | 通过支付 SOL Tips 加速交易 | [tips-mode.md](./tips-mode.md) |
| `sendTransaction`（Points） | Points | 通过预购 Points 积分加速交易 | [points-mode.md](./points-mode.md) |

## 典型场景

- **DEX 交易加速**：在 Solana DEX 交易中获得更快的确认速度，抓住交易窗口
- **NFT 铸造抢购**：在热门 NFT Mint 场景中通过优先通道提高成功率
- **套利交易**：对延迟敏感的套利策略，使用 Prime 等级获取亚秒级确认
- **高频交易**：大量交易需要稳定高效的提交与确认

## 监控与诊断

### Dashboard

- 登录 ZAN 用户门户 → SOL Trading → Overview
- 支持 Points 视图和 Tips 视图切换

### 关键指标

| 指标 | 说明 |
| --- | --- |
| **Success Rate** | 各等级的交易确认率 |
| **On-Chain Time** | 从提交到上链的时间（不含客户端到 ZAN 的传输时间） |
| **Remaining Points** | 实时剩余 Points（仅 Points 集成方式可用） |

### 交易验证

- 通过交易哈希在 SOL Trading Boost 模块的「查看消费详情」中搜索
- 页面仅显示最近 30 天的交易记录
- 未出现的交易表示未经加速处理

## Agent 使用建议

当用户表达以下意图时，**优先使用 Trading 分类**：

- 「加速 Solana 交易」「SOL 交易太慢」「提高 Solana 交易成功率」
- 「防止 MEV 抢跑」「需要优先通道」「交易确认太慢」
- 「DEX 交易需要加速」「NFT 铸造需要优先」

**Agent 应确认的信息：**
- 确认用户的目标链是 **Solana**（Trading Boost 目前仅支持 Solana）
- 确认用户有 ZAN API Key（Trading 仅支持 API Key 鉴权）
- 引导用户选择 Tips 或 Points 方式
- 对于 Tips 方式，需提前联系 ZAN 开通权限

## FAQ

### Q1：交易加速为什么可能失败？

- 超出 TPS 限制
- Points 余额不足

### Q2：如何验证交易是否被加速？

在「消费详情」中通过交易哈希查询，若未出现则表示未被加速处理。

### Q3：是否支持测试网？

目前暂不支持测试网。

## 支持渠道

如有技术问题，请通过以下渠道联系支持团队：
- [Telegram](https://t.me/ZANTeam_official)
- [Discord](https://discord.gg/yZzn9BEtFU)
- [反馈门户](https://zan.top/home/contact)

## 风险提示

> **重要提醒：**
> 1. 本 Skill 提供的所有数据仅供参考，不构成任何投资或交易建议
> 2. 交易加速涉及链上交易与费用（SOL Tips 或 Points），请用户自行评估风险与成本
> 3. MEV 保护可能引入微小延迟，请根据实际场景权衡安全性与速度
> 4. 请遵守当地法律法规，合法合规使用区块链交易服务

## 参考资料

- [ZAN Trading Boost 官方文档](https://docs.zan.top/docs/trading-boost)
- [Solana SWQoS 技术文档](https://solana.com/developers/guides/advanced/stake-weighted-qos)
- [ZAN sendTransaction API 文档](https://docs.zan.top/reference/sendtransaction-solana)
