# ZAN Blockchain Skills

> 面向 **AI Agent** 与 **开发者** 的标准化区块链能力封装，提供链上查询、数据分析、交易辅助与运维管理等 Skills。

[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](LICENSE)

## 项目简介

ZAN Blockchain Skills 是由 ZAN Team 构建的区块链能力 Skills 集合，旨在帮助 AI Agent 和开发者更高效地完成链上任务。每个 Skill 均按照 [Agent Skills Specification](https://agentskills.io/specification) 标准组织，提供统一的接口描述、代码示例与调用建议。

### 支持的能力范围

| 分类 | 能力 | 参考文档 |
| --- | --- | --- |
| **Chain RPC** | 余额查询、区块查询、交易查询、合约调用、Gas 估算 | [chain-rpc.md](skills/zan-blockchain-skill/references/chain-rpc.md) |
| **Data** | NFT API、Token API、Simulation API、Debug API（JSON-RPC 2.0） | [data-api.md](skills/zan-blockchain-skill/references/data-api.md) |
| **Trading** | 基于 SWQoS 的 Solana Trading Boost，支持 Tips / Points 两种方式 | [trading-boost.md](skills/zan-blockchain-skill/references/trading-boost.md) |
| **Operational** | 配额查询、服务状态 | [operational.md](skills/zan-blockchain-skill/references/operational.md) |

### 接入方式

当前支持通过 **API Key** 方式接入 ZAN 区块链服务。

| 方式 | 适用场景 | 特点 |
| --- | --- | --- |
| **API Key** | 企业客户、高频调用 | 功能完整、有账户管理、支持订阅 |

<!-- x402 Payment 方式正在开发中，后续将支持无需注册的即用即付接入方式 -->

## 快速开始

1. 注册 [ZAN 账户](https://zan.top) 并获取 API Key
2. 设置环境变量

```bash
export ZAN_API_KEY="your-api-key"
```

3. 调用 RPC 端点

```bash
curl -X POST https://api.zan.top/node/v1/eth/mainnet/${ZAN_API_KEY} \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}'
```

## 在 AI Agent 中使用

### Claude Code / Cursor 集成

ZAN Blockchain Skills 遵循 [Agent Skills Specification](https://agentskills.io/specification)，可直接被 Claude Code、Cursor 等 AI Agent 工具加载使用。

**方式一：通过 GitHub 安装**

```bash
npx skills add zan-team/zan-blockchain-skills
```

**方式二：本地安装**

```bash
# 克隆仓库
git clone https://github.com/zan-team/zan-blockchain-skills.git

# 全局安装
cp -r zan-blockchain-skills/skills/zan-blockchain-skill ~/.claude/skills/zan-blockchain-skill

# 或项目级安装
cp -r zan-blockchain-skills/skills/zan-blockchain-skill .claude/skills/zan-blockchain-skill
```

**方式三：直接使用**

将本仓库克隆到项目目录中，Claude Code 会自动发现 `skills/zan-blockchain-skill/SKILL.md` 并加载。

### 安装验证

安装后，在 Agent 中尝试以下提问来验证：
- "帮我查询这个地址的 ETH 余额"
- "查询 BAYC 的 NFT 持有者列表"
- "加速我的 Solana 交易"

## 项目结构

```
zan-blockchain-skills/
├── README.md                                # 项目说明
├── LICENSE                                  # Apache 2.0
└── skills/
    └── zan-blockchain-skill/
        ├── SKILL.md                         # Skill 入口（Agent 自动加载）
        └── references/                      # 详细参考（Agent 按需加载）
            ├── chain-rpc.md                 # Chain RPC API 参考
            ├── data-api.md                  # Advanced Data API 参考
            ├── trading-boost.md             # Solana Trading Boost 参考
            └── operational.md               # Operational API 参考
```

## Agent 使用说明

### Skill 选择原则

| 用户意图 | 推荐 Skill 分类 |
| --- | --- |
| 余额、交易、区块、Gas、合约调用 | **Chain RPC** |
| 分析、持仓、NFT、调用链、资金流 | **Data** |
| Solana 交易加速、Trading Boost | **Trading** |
| 配额、余量、资源状态 | **Operational** |

### 对模糊问题的处理建议

若用户意图不完整，Agent 应主动追问：
- 目标链是 Ethereum、Base、Arbitrum 还是其他网络？
- 查询的是原生币余额还是 Token 余额？
- 需要返回原始结果还是结构化分析结果？
- 是否需要附带代码示例？

## 定价

请访问 [ZAN 官网](https://zan.top/pricing) 查看最新的 API Key 订阅套餐与定价。

## 风险提示

> **重要提醒：**
> 1. 本 Skill 提供的所有数据仅供参考，不构成任何投资或交易建议
> 2. 区块链数据具有不可篡改性但可能存在延迟，请以链上最终确认为准
> 3. 对于因使用本 Skill 产生的任何损失，ZAN 不承担法律责任
> 4. 请遵守当地法律法规，合法合规使用区块链数据服务

## License

本项目采用 [Apache License 2.0](LICENSE) 开源协议。

Copyright 2024-2026 ZAN Team

## 参考资料

- [Agent Skills Specification](https://agentskills.io/specification)
- [QuickNode Blockchain Skills](https://github.com/quiknode-labs/blockchain-skills)
- [ZAN 官网](https://zan.top)
- [ZAN API 文档](https://docs.zan.top/reference)
