# ZAN Blockchain Skills 规范说明

## 1. 规范遵循

ZAN Blockchain Skills 遵循 [Agent Skills Specification](https://agentskills.io/specification) 规范，确保与主流 AI Agent 生态兼容，便于 Agent 正确理解、发现和调用 ZAN 提供的区块链能力。

## 2. 文档结构

采用**三层结构**组织文档，确保内容清晰、便于扩展、方便 Agent 理解：

### 2.1 第一层：根目录 README

- 位置：`README.md`
- 用途：ZAN Blockchain Skills 整体能力与使用方式
- 内容：项目简介、支持的接入方式、能力范围、分类导航、快速开始、风险提示等

### 2.2 第二层：分类 README

- 位置：`docs/<category>/README.md`
- 用途：按能力类型拆分的分类说明
- 内容：该类 Skills 的定位与边界、支持的鉴权方式、典型场景、API 列表、Agent 选择建议

### 2.3 第三层：单 API 文档

- 位置：`docs/<category>/<api-name>.md`
- 用途：每个 API 独立成文，结构统一
- 内容：见下方 API 文档模板

## 3. API 文档模板

每个 API 文档必须包含以下固定字段：

| 字段 | 说明 |
|------|------|
| **API Name** | API 名称（如 `eth_blockNumber`） |
| **Category** | 所属分类（Chain RPC / Data / Trading / Operational） |
| **Description** | API 功能描述 |
| **Supported Auth Methods** | 支持的鉴权方式：API Key |
| **Use Cases** | 典型使用场景 |
| **Request Method** | HTTP 方法（如 POST） |
| **Endpoint** | 请求端点 URL |
| **Parameters** | 请求参数说明 |
| **Response Schema** | 响应数据结构 |
| **Error Codes** | 错误码及说明 |
| **Example Request** | 请求示例 |
| **Example Response** | 响应示例 |
| **Code Samples** | 代码示例（含 curl、JavaScript/TypeScript） |
| **Notes for Agents** | Agent 调用建议、常见错误处理 |

## 4. 鉴权方式

当前 ZAN Blockchain Skills 支持 **API Key** 鉴权：

### 4.1 API Key

- **适用场景**：企业客户、高频调用、需要完整功能（含 Trading）、需要账单和发票
- **特点**：功能完整、有账户管理、支持订阅套餐、有技术支持
- **使用方式**：通过 URL 路径参数传递 API Key，如 `https://api.zan.top/node/v1/eth/mainnet/{API_KEY}`

<!-- x402 Payment 鉴权方式正在开发中，后续将支持无需注册的即用即付接入 -->

## 5. 文档编写指南

### 5.1 语言规范

- 描述与注释：使用**简体中文**
- 代码与 API 术语：使用**英文**（如 `eth_blockNumber`、`Authorization`、`POST`）

### 5.2 代码示例要求

- 每个 API 必须提供 **curl** 示例
- 每个 API 必须提供 **JavaScript/TypeScript** 示例
- 代码示例应可直接运行，避免占位符

### 5.3 渐进式披露

- 根 README 保持简洁，提供导航与快速开始
- 分类 README 聚焦该分类的边界与典型场景
- 单 API 文档包含完整技术细节，便于 Agent 按需加载

### 5.4 Agent 友好

- 在 **Notes for Agents** 中明确：适合什么问题触发、返回结果如何组织、遇到歧义时如何补问
- 错误处理建议需覆盖常见错误码

## 6. 分类定义

### 6.1 Chain RPC

- **定位**：链上基础交互与实时查询
- **能力**：查询余额、区块、交易、合约调用、Gas 估算等
- **典型场景**：快速查询钱包余额、查询交易状态、模拟合约执行

### 6.2 Data

- **定位**：ZAN Advanced API — 链上数据聚合、分析与结构化输出（JSON-RPC 2.0）
- **能力**：NFT API（8 个方法）、Token API（7 个方法）、Simulation API（2 个方法）、Debug API（5 个方法）
- **典型场景**：查询 NFT/Token 持有者、地址资产分析、交易模拟、调用链追踪

### 6.3 Trading

- **定位**：Solana Trading Boost — 基于 SWQoS 技术的交易加速服务
- **能力**：Tips 竞价加速、Points 等级加速（Prime/Turbo/Standard）、MEV 保护
- **典型场景**：DEX 交易加速、NFT 铸造抢购、套利交易、高频交易

### 6.4 Operational

- **定位**：运维、资源与服务状态
- **能力**：查询配额、查询余量、服务状态
- **典型场景**：检查额度、监控资源可用性

## 7. 参考资料

- [Agent Skills Specification](https://agentskills.io/specification)
- [ZAN API Reference](https://docs.zan.top/reference)
