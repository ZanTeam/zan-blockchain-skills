# ZAN Blockchain Skills

> 面向 **AI Agent** 与 **开发者** 的标准化区块链能力封装，提供链上查询、数据分析、交易辅助与运维管理等 Skills。

[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](LICENSE)

## 项目简介

ZAN Blockchain Skills 是由 ZAN Team 构建的区块链能力 Skills 集合，旨在帮助 AI Agent 和开发者更高效地完成链上任务。每个 Skill 均按照 [Agent Skills Specification](https://agentskills.io/specification) 标准组织，提供统一的接口描述、代码示例与调用建议。

### 支持的能力范围

| 分类 | 能力 | 说明 |
| --- | --- | --- |
| **Chain RPC** | 链上基础交互 | 余额查询、区块查询、交易查询、合约调用、Gas 估算 |
| **Data** | Advanced API 数据分析 | NFT API、Token API、Simulation API、Debug API（JSON-RPC 2.0） |
| **Trading** | Solana 交易加速 | 基于 SWQoS 的 Trading Boost，支持 Tips / Points 两种方式 |
| **Operational** | 运维管理 | 配额查询、服务状态 |

### 接入方式

当前支持通过 **API Key** 方式接入 ZAN 区块链服务。

| 方式 | 适用场景 | 特点 |
| --- | --- | --- |
| **API Key** | 企业客户、高频调用 | 功能完整、有账户管理、支持订阅 |

<!-- x402 Payment 方式正在开发中，后续将支持无需注册的即用即付接入方式 -->

## 为什么选择 ZAN Blockchain Skills

- **20+ 链覆盖** — 支持 Ethereum、Base、Polygon、Arbitrum、Optimism 等主流网络
- **标准化文档** — 遵循 Agent Skills Specification，便于 Agent 理解与调用
- **中文文档支持** — 完整的中文文档与代码示例
- **完整能力覆盖** — 涵盖 Chain RPC、数据分析、交易加速、运维管理

## 快速开始

1. 注册 [ZAN 账户](https://zan.top) 并获取 API Key
2. 使用 API Key 调用 RPC 端点

```bash
curl -X POST https://api.zan.top/node/v1/eth/mainnet/{YOUR_API_KEY} \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}'
```

> 详细步骤参见 [API Key 快速开始指南](docs/getting-started/quick-start-api-key.md)

## Skills 分类导航

| 分类 | 文档 | 说明 |
| --- | --- | --- |
| Chain RPC | [docs/chain-rpc/](docs/chain-rpc/README.md) | 链上基础交互与实时查询 |
| Data | [docs/data/](docs/data/README.md) | 链上数据聚合、分析与结构化输出 |
| Trading | [docs/trading/](docs/trading/README.md) | Solana Trading Boost 交易加速 |
| Operational | [docs/operational/](docs/operational/README.md) | 运维、资源与服务状态管理 |
| 场景示例 | [docs/cases/](docs/cases/README.md) | 高频场景端到端用例 |

<!-- x402 Payment 文档暂未开放，待功能上线后启用 -->

## 定价

请访问 [ZAN 官网](https://zan.top/pricing) 查看最新的 API Key 订阅套餐与定价。

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
- 是否需要换算 USD 估值？

## 开发者接入说明

### 环境要求

- Node.js >= 18
- npm 或 yarn

### 环境变量配置

```bash
export ZAN_API_KEY="your-api-key"
```

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
- [MCP Specification](https://modelcontextprotocol.io)
- [QuickNode Blockchain Skills](https://github.com/quiknode-labs/blockchain-skills)
- [ZAN 官网](https://zan.top)
