# 场景示例

## 概述

本文档提供 ZAN Blockchain Skills 的**端到端场景示例**，覆盖从快速查询到自动化监控的典型用例。每个示例均包含完整实现步骤、代码示例与鉴权方式建议，帮助开发者和 AI Agent 快速落地链上能力。

## 用例列表

| 序号 | 用例 | 适用场景 | 推荐鉴权 | 文档链接 |
|------|------|----------|----------|----------|
| 1 | 快速查询账户余额 | 用户需要快速查询某地址的 ETH 余额 | API Key | [quick-check-balance.md](./quick-check-balance.md) |
| 2 | 调试合约调用 | 开发者需要模拟合约调用以排查问题 | API Key | [debug-contract-call.md](./debug-contract-call.md) |
| 3 | 追踪交易 | 分析一笔复杂交易的完整调用链与资金流向 | API Key | [trace-transaction.md](./trace-transaction.md) |
| 4 | 生成 NFT 持有者代码 | 获取某 NFT 项目的持有者列表并生成分析代码 | API Key | [generate-nft-holder-code.md](./generate-nft-holder-code.md) |

## 用例简要说明

### 场景 1：快速查询账户余额

- **触发语句**：「帮我查一下这个地址的 ETH 余额，我有 API Key」
- **核心能力**：`eth_getBalance`（Chain RPC）
- **扩展**：可延伸至 ERC20 Token 余额查询（`eth_call` + `balanceOf`）

### 场景 2：调试合约调用

- **触发语句**：「我调用合约总是失败，帮我排查一下」
- **核心能力**：`eth_call`、`eth_estimateGas`（Chain RPC）
- **输出**：模拟调用结果、Gas 估算、常见错误与解决方案

### 场景 3：追踪交易

- **触发语句**：「帮我分析一下这笔交易的内部调用情况」
- **核心能力**：`eth_getTransactionByHash`、`transaction-trace`（Chain RPC + Data）
- **输出**：交易基本信息、完整调用链、资金流向分析

### 场景 4：生成 NFT 持有者代码

- **触发语句**：「帮我查询 BAYC 的持有者列表并生成代码」
- **核心能力**：`nft-holders`（Data）
- **输出**：持有者列表、分页处理、汇总分析代码

## 相关文档

- [快速开始 - API Key](../getting-started/quick-start-api-key.md)
- [Chain RPC](../chain-rpc/README.md)
- [Data](../data/README.md)
- [Trading](../trading/README.md)
- [Operational](../operational/README.md)
