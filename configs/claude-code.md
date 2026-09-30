# Claude Code 接第三方 Anthropic 兼容端点

## 环境变量方式

```bash
export ANTHROPIC_BASE_URL="https://kuafushe.cc"
export ANTHROPIC_AUTH_TOKEN="sk-你的密钥"
claude
```

## 三个必踩的坑

### 1. `base_url` 不带 `/v1`

客户端会自己拼上 `/v1/messages`。

```text
填 https://kuafushe.cc       → 实际请求 /v1/messages        ✅
填 https://kuafushe.cc/v1    → 实际请求 /v1/v1/messages      ❌ 404
```

### 2. 不要填 `/anthropic` 后缀

有些中转站的教程用 `https://xxx/anthropic`（MiniMax 就是这个写法），
但**不是所有站都有这个路径**。没有的话会返回网站首页 HTML，
SDK 解析失败，报错信息**很含糊、极难排查**。

```text
填 https://kuafushe.cc/anthropic  → 返回 HTML 首页  ❌ 静默失败
```

**判断方法**：拿 curl 打一下，返回 401（要 key）说明是真端点，返回 HTML 就是路径不对：

```bash
curl -i https://kuafushe.cc/v1/messages \
  -H "x-api-key: sk-你的密钥" \
  -H "anthropic-version: 2023-06-01" \
  -H "content-type: application/json" \
  -d '{"model":"claude-sonnet-4-5-20250929","max_tokens":32,"messages":[{"role":"user","content":"ping"}]}'
```

### 3. 模型可用性取决于你的 Key 分组

中转站普遍按分组放模型，**一个 Key 不一定有全部模型**。
接到 404 `model_not_found` 先别急着以为端点坏了，先查你的 key 支持哪些：

```bash
curl https://kuafushe.cc/v1/models \
  -H "Authorization: Bearer sk-你的密钥"
```

用返回列表里的模型名。

## 常见报错对照

| 报错 | 原因 | 处理 |
|------|------|------|
| 401 | Key 无效 / 前缀不对 / 没传 header | 检查 `ANTHROPIC_AUTH_TOKEN` |
| 402 | 余额不足 | 充值 |
| 404 | 模型名不在你的分组，或 `base_url` 多带了 `/v1` `/anthropic` | 查 `/v1/models`、修 base_url |
| 返回 HTML 内容 | `base_url` 路径不存在 | 去掉 `/anthropic` 之类后缀 |

## 免责

- 第三方中转服务，**敏感数据别走中转**，重要业务用官方 API
- 用多少充多少，别一次冲太多
