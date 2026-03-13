# 场景 4：生成 NFT 持有者代码

## 场景描述

需要获取某个 NFT 项目的持有者列表，用于空投快照、持有者分析、大户识别等。通过 `nft-holders` API 可获取结构化持有者数据，支持分页与多链。

## 触发语句示例

- 「帮我查询 BAYC 的持有者列表并生成代码」
- 「获取这个 NFT 合约的所有持有者，做空投快照」
- 「分析某 NFT 项目的持有者分布」

## 推荐方式

**API Key**

## 实现步骤

### 1. 确定 NFT 合约地址

若用户提供项目名称（如 BAYC），需先解析合约地址。常见 NFT 合约地址示例：

| 项目 | 合约地址 (Ethereum) |
|------|---------------------|
| BAYC (Bored Ape Yacht Club) | `0xBC4CA0EdA7647A8aB7C2061c2E118A18a936f13D` |
| CryptoPunks | `0xb47e3cd837dDF8e4c57F05d70Ab865de6e193BBB` |

### 2. 调用 nft-holders API

使用 Data API 的 `nft-holders`，传入 `contractAddress`、`chain`、`page`、`pageSize` 等参数。

### 3. 处理分页数据

API 返回 `total`、`page`、`pageSize`，需循环请求直至获取全部持有者。

### 4. 生成汇总分析

对持有者列表进行统计：总人数、大户（持有多枚）、活跃度（按 `lastTransferTime`）等。

## 完整代码示例（TypeScript）

### API Key 方式

```typescript
const API_KEY = process.env.ZAN_API_KEY;
const BAYC_CONTRACT = "0xBC4CA0EdA7647A8aB7C2061c2E118A18a936f13D";
const CHAIN = "eth";
const PAGE_SIZE = 100;

interface Holder {
  holderAddress: string;
  tokenId: string;
  balance: string;
  lastTransferTime: number;
}

async function fetchAllHolders(
  contractAddress: string,
  chain: string = "eth"
): Promise<Holder[]> {
  const allHolders: Holder[] = [];
  let page = 1;
  let total = Infinity;

  while (allHolders.length < total) {
    const params = new URLSearchParams({
      apiKey: API_KEY!,
      contractAddress,
      chain,
      page: String(page),
      pageSize: String(PAGE_SIZE),
    });

    const response = await fetch(
      `https://api.zan.top/data/v1/nft/holders?${params}`
    );
    const data = await response.json();

    if (data.holders) {
      allHolders.push(...data.holders);
    }
    total = data.total ?? 0;
    page++;

    if (allHolders.length >= total || !data.holders?.length) break;

    // 避免请求过快
    await new Promise((r) => setTimeout(r, 200));
  }

  return allHolders;
}

async function main() {
  const holders = await fetchAllHolders(BAYC_CONTRACT, CHAIN);

  console.log(`总持有者数: ${holders.length}`);

  // 大户统计（持有数量 > 1，适用于 ERC-1155 或可批量持有的场景）
  const balanceMap = new Map<string, number>();
  holders.forEach((h) => {
    const count = parseInt(h.balance, 10) || 1;
    balanceMap.set(
      h.holderAddress,
      (balanceMap.get(h.holderAddress) ?? 0) + count
    );
  });

  const whales = [...balanceMap.entries()]
    .filter(([, count]) => count > 1)
    .sort((a, b) => b[1] - a[1])
    .slice(0, 10);

  console.log("\n大户 Top 10:");
  whales.forEach(([addr, count], i) => {
    console.log(`  ${i + 1}. ${addr}: ${count} 枚`);
  });

  // 导出为 CSV
  const csv = [
    "holderAddress,tokenId,balance,lastTransferTime",
    ...holders.map(
      (h) =>
        `${h.holderAddress},${h.tokenId},${h.balance},${h.lastTransferTime}`
    ),
  ].join("\n");

  // 写入文件或返回
  return { holders, whales, csv };
}

main();
```

## 数据分析示例

### 按活跃度筛选

```typescript
// 最近 30 天有转账的持有者（活跃持有者）
const thirtyDaysAgo = Math.floor(Date.now() / 1000) - 30 * 24 * 3600;
const activeHolders = holders.filter(
  (h) => h.lastTransferTime >= thirtyDaysAgo
);
console.log(`活跃持有者 (30 天内): ${activeHolders.length}`);
```

### 生成空投快照

```typescript
// 按地址去重，每个地址计 1 票（适用于 1:1 空投）
const uniqueAddresses = [...new Set(holders.map((h) => h.holderAddress))];
console.log(`空投快照地址数: ${uniqueAddresses.length}`);

// 导出为 JSON
const snapshot = {
  contract: BAYC_CONTRACT,
  chain: CHAIN,
  timestamp: Date.now(),
  addresses: uniqueAddresses,
};
```

## 相关文档

- [nft-holders](../data/nft-holders.md)
- [Data 分类](../data/README.md)
- [API Key 快速开始](../getting-started/quick-start-api-key.md)
