# query-quota

## API Name

`query-quota`

## Category

Operational

## Description

查询当前账户的 API 调用配额与使用情况，包括总配额、已使用量、剩余配额、重置时间及当前套餐信息。适用于在调用前检查额度、监控使用量、避免超限等场景。

## Supported Auth Methods

| 鉴权方式 | 支持 |
| --- | --- |
| API Key | ✅ |

## Use Cases

- **调用前检查额度**：在批量调用或自动化任务前，确认配额是否充足
- **监控使用量**：定期查询已使用量，了解 API 消耗情况
- **避免超限**：接近配额上限时提前预警，避免请求被限流
- **套餐管理**：查看当前套餐与重置时间，便于升级或续费决策

## Request Method

`GET`

## Endpoint

```
https://api.zan.top/operational/v1/quota?apiKey={API_KEY}
```

## Parameters

| 参数 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `apiKey` | string | 是 | 您的 ZAN API Key，用于鉴权并查询对应账户的配额 |

> `apiKey` 可通过 URL 查询参数传递，也可通过请求头 `Authorization: Bearer {API_KEY}` 传递。

## Response Schema

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `totalQuota` | number | 总配额（调用次数/月或/天，取决于套餐） |
| `usedQuota` | number | 已使用配额 |
| `remainingQuota` | number | 剩余配额 |
| `resetTime` | string | 配额重置时间（ISO 8601 格式） |
| `plan` | string | 当前套餐名称，如 `free`、`starter`、`pro` |

### 响应示例结构

```json
{
  "totalQuota": 100000,
  "usedQuota": 45230,
  "remainingQuota": 54770,
  "resetTime": "2025-04-01T00:00:00Z",
  "plan": "pro"
}
```

## Error Codes

| HTTP Status | 错误码 | 说明 | 处理建议 |
| --- | --- | --- | --- |
| 401 | `UNAUTHORIZED` | API Key 无效或过期 | 检查 API Key 是否正确、是否已过期 |
| 429 | `RATE_LIMITED` | 请求频率超限 | 降低调用频率，稍后重试 |
| 500 | `INTERNAL_ERROR` | 服务端内部错误 | 稍后重试或联系技术支持 |

## Example Request

### curl（API Key）

```bash
curl -X GET "https://api.zan.top/operational/v1/quota?apiKey=YOUR_API_KEY"
```

### curl（Bearer Token）

```bash
curl -X GET "https://api.zan.top/operational/v1/quota" \
  -H "Authorization: Bearer YOUR_API_KEY"
```

## Example Response

### 成功响应

```json
{
  "totalQuota": 100000,
  "usedQuota": 45230,
  "remainingQuota": 54770,
  "resetTime": "2025-04-01T00:00:00Z",
  "plan": "pro"
}
```

### 失败响应

```json
{
  "error": {
    "code": "UNAUTHORIZED",
    "message": "API Key 无效或已过期"
  }
}
```

## Code Samples

### JavaScript/TypeScript（API Key）

```typescript
const API_KEY = process.env.ZAN_API_KEY;
const endpoint = `https://api.zan.top/operational/v1/quota?apiKey=${API_KEY}`;

const response = await fetch(endpoint);
const data = await response.json();

if (data.totalQuota !== undefined) {
  console.log("总配额:", data.totalQuota);
  console.log("已使用:", data.usedQuota);
  console.log("剩余配额:", data.remainingQuota);
  console.log("重置时间:", data.resetTime);
  console.log("当前套餐:", data.plan);

  if (data.remainingQuota < 1000) {
    console.warn("配额即将用尽，请考虑升级套餐");
  }
} else {
  console.error("查询失败:", data.error);
}
```

## Notes for Agents

- **触发条件**：用户询问「还有多少额度」「配额用完了吗」「API 调用次数」「套餐信息」时，优先使用本 API。
- **鉴权限制**：仅支持 API Key。
- **参数传递**：`apiKey` 可通过 URL 参数或 `Authorization: Bearer` 传递，二选一即可。
- **结果解读**：`remainingQuota` 接近 0 时，可提示用户升级套餐或等待重置；`resetTime` 为配额周期结束时间。
- **调用时机**：在 Agent 执行批量任务前，可主动查询配额，避免任务中途因超限失败。
