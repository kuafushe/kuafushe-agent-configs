# Claude Code 接第三方 Anthropic 兼容端点

## 环境变量方式

```bash
export ANTHROPIC_BASE_URL="https://你的站点"
export ANTHROPIC_AUTH_TOKEN="sk-你的密钥"
claude
```

> 如果你的端点用的是 `ANTHROPIC_API_KEY` 而不是 `ANTHROPIC_AUTH_TOKEN`，两者都支持，选一个即可。

## 三个必踩的坑

### 1. `base_url` 不带 `/v1`

客户端会自己拼上 `/v1/messages`。

```text
填 https://example.com       → 实际请求 /v1/messages        ✅
填 https://example.com/v1    → 实际请求 /v1/v1/messages      ❌ 404
```

**报错不会告诉你「多写了一层」**，只给一个光秃秃的 Not Found。这是接中转站最高频的翻车点。

### 2. 不要填 `/anthropic` 后缀

有些中转站的教程用 `https://xxx/anthropic`（MiniMax 就是这个写法），
但**不是所有站都有这个路径**。没有的话会返回网站首页 HTML，
SDK 解析失败，报错信息**很含糊、极难排查**。

```text
填 https://example.com/anthropic  → 返回 HTML 首页  ❌ 静默失败
```

**判断方法**：拿 curl 打一下，返回 401（要 key）说明是真端点，返回 HTML 就是路径不对：

```bash
curl -i https://你的站点/v1/messages \
  -H "x-api-key: sk-你的密钥" \
  -H "anthropic-version: 2023-06-01" \
  -H "content-type: application/json" \
  -d '{"model":"claude-sonnet-4-5","max_tokens":32,"messages":[{"role":"user","content":"ping"}]}'
```

- **401** = 端点真实存在，只是缺有效 key ✅
- **HTML** = 路径填错了 ❌

### 3. 模型可用性取决于你的 Key 分组

中转站普遍按分组放模型，**一个 Key 不一定有全部模型**。
接到 404 `model_not_found` 先别急着以为端点坏了，先查你的 key 支持哪些：

```bash
curl https://你的站点/v1/models \
  -H "Authorization: Bearer sk-你的密钥"
```

> ⚠️ 部分站点未开放 `/v1/models`，此时报 404 属正常现象。去控制台的分组说明里看可用模型即可。

## 常见报错对照

| 报错 | 原因 | 处理 |
|------|------|------|
| **401** | Key 无效 / 前缀不对 / 没传 header | 检查 `ANTHROPIC_AUTH_TOKEN` |
| **402** | 余额不足 | 充值 |
| **403** | 分组无权访问该模型 | 换分组或换模型 |
| **404** | 模型名不在你的分组，或 `base_url` 多带了 `/v1` `/anthropic` | 查 `/v1/models`、修 base_url |
| **返回 HTML** | `base_url` 路径不存在 | 去掉 `/anthropic` 之类后缀 |

## cc-switch 用户

用 [cc-switch](https://github.com/farion1231/cc-switch) 管理供应商时，走「自定义配置」，编辑框填：

```json
{
  "env": {
    "ANTHROPIC_BASE_URL": "https://你的站点",
    "ANTHROPIC_AUTH_TOKEN": "sk-你的密钥"
  }
}
```

cc-switch 的「获取模型列表」按钮会去调 `/v1/models`。端点没开这个路径时按钮报 404 属正常，手动填模型名即可。

## 可运行示例

下面用 [夸父社API](https://kuafushe.cc)（OpenAI + Anthropic 双协议）做示例，换其他端点只改地址：

```bash
export ANTHROPIC_BASE_URL="https://kuafushe.cc"
export ANTHROPIC_AUTH_TOKEN="sk-你的密钥"
claude
```

注册入口：https://kuafushe.cc/register?aff=XB5VM25Z6A8C（新用户送 $0.2 体验额度）

常用模型 ID（按分组而定，以控制台为准）：

| 分组 | 模型 | 倍率 |
|---|---|---|
| Claude Kiro Plan | `claude-sonnet-5` / `claude-opus-5` / `claude-fable-5` | 0.3x |
| GPT-福利组 | `gpt-5.6-sol` / `gpt-6-astra` | 0.2x |
| 国模-003 | `deepseek-v4.1-flash` | 0.03x |

## 免责

- 第三方中转服务，**敏感数据别走中转**，重要业务用官方 API
- 用多少充多少，别一次充太多
- 各站分组 / 价格 / 模型随时可能调整，以官网为准