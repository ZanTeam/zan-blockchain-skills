# Chain RPC

> 链上基础交互与实时查询能力

## 定位

Chain RPC 提供链上**原始状态查询**与**实时交互**能力，是区块链开发中最基础、最常用的能力分类。适用于需要直接与链上数据交互、获取最新状态的场景。

## 支持的鉴权方式

| 鉴权方式 | 说明 |
| --- | --- |
| **API Key** | 适用于企业客户、高频调用，需在 [ZAN 控制台](https://zan.top) 注册并获取 |
<!-- x402 支付方式正在开发中，后续将在此处补充 x402 相关内容 -->

## 能力范围

| 能力 | 说明 |
| --- | --- |
| 查询账户余额 | 原生币（ETH/BNB 等）余额查询 |
| 查询区块信息 | 区块高度、区块详情 |
| 查询交易详情 | 根据交易哈希获取交易信息 |
| 查询交易回执 | 交易执行结果、状态、Gas 消耗 |
| 合约只读调用 | 调用合约 view/pure 方法，不消耗 Gas、不上链 |
| 模拟合约执行 | 在指定区块状态下模拟执行 |
| Gas 估算 | 预估交易所需 Gas 量 |

## 典型场景

- **快速查询钱包余额**：检查地址原生币余额
- **查询交易是否成功**：根据交易哈希确认执行状态
- **模拟合约方法执行**：在提交交易前预览执行结果
- **调试 Gas 消耗**：估算交易费用，优化 Gas 设置

## API 列表

| API | 说明 | 文档 |
| --- | --- | --- |
| `eth_blockNumber` | 查询当前最新区块高度 | [eth_blockNumber.md](eth_blockNumber.md) |
| `eth_getBalance` | 查询指定地址的原生币余额 | [eth_getBalance.md](eth_getBalance.md) |
| `eth_getTransactionByHash` | 根据交易哈希查询交易详情 | [eth_getTransaction.md](eth_getTransaction.md) |
| `eth_getTransactionReceipt` | 根据交易哈希查询交易回执 | （待补充） |
| `eth_call` | 调用合约只读方法 | [eth_call.md](eth_call.md) |
| `eth_estimateGas` | 估算交易所需 Gas 量 | [eth_estimateGas.md](eth_estimateGas.md) |

## Agent 选择建议

当用户表达以下意图时，**优先使用 Chain RPC**：

- 「链上原始状态查询」：余额、区块、交易详情
- 「实时交互」：最新区块、当前余额、即时查询
- 「合约只读调用」：查询 ERC20 余额、读取合约状态
- 「Gas 估算」：提交交易前预估费用

若用户需求偏向「分析、统计、追踪、聚合」，请参考 [Data](../data/README.md) 分类；若涉及「交易加速、广播优化」，请参考 [Trading](../trading/README.md) 分类。
