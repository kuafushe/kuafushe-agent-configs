# Anthropic / OpenAI 兼容端点配置合集

把 Claude Code、claude-agent-sdk、n8n、Cursor、Cline 等工具接到**任意 Anthropic / OpenAI 兼容端点**的配置示例与避坑指南。

写这个仓库的原因：接第三方端点时，`base_url` 和模型名这两个地方踩的坑最多，报错还特别含糊，每次都得重新查一遍。这里把所有坑固定下来。

---

## ⚠️ 接之前先看：`base_url` 三大坑

### 1. Anthropic 协议不带 `/v1`

客户端自己会拼 `/v1/messages`：

```text
https://kuafushe.cc        ✅
https://kuafushe.cc/v1     ❌ 变成 /v1/v1/messages → 404
```

OpenAI 协议才带 `/v1`（`https://kuafushe.cc/v1`）。

### 2. 不要填 `/anthropic` 后缀

MiniMax 那类教程用 `https://xxx/anthropic`，但**不是每个站都有这个路径**。
没有的话返回网站首页 HTML —— SDK 解析失败，报错极其含糊，**静默失败**。

### 3. 模型名取决于你的 Key 分组

一个 Key 不一定有全部模型。撞到 `model_not_found` 先查：

```bash
curl https://kuafushe.cc/v1/models -H "Authorization: Bearer sk-你的密钥"
```

---

## 目录

| 路径 | 内容 |
|------|------|
| [`configs/claude-code.md`](configs/claude-code.md) | Claude Code 接入 + `base_url` 三坑 + 报错对照表 |
| [`configs/providers.json.example`](configs/providers.json.example) | claude-agent-sdk 项目用的 `providers.json` 示例（可直接复制） |
| [`tutorials/ai-pentest-agent.md`](tutorials/ai-pentest-agent.md) | 给 AI 渗透 Agent 配便宜 Claude API（含分组选型、成本对比） |
| [`templates/README.md`](templates/README.md) | n8n 工作流模板（含导入与凭据步骤） |

---

## 快速接入

**Claude Code：**

```bash
export ANTHROPIC_BASE_URL="https://kuafushe.cc"
export ANTHROPIC_AUTH_TOKEN="sk-你的密钥"
claude
```

**OpenAI SDK：**

```python
from openai import OpenAI
client = OpenAI(api_key="sk-你的密钥", base_url="https://kuafushe.cc/v1")
```

**Claude API（curl 验证端点通不通）：**

```bash
curl -i https://kuafushe.cc/v1/messages \
  -H "x-api-key: sk-你的密钥" \
  -H "anthropic-version: 2023-06-01" \
  -H "content-type: application/json" \
  -d '{"model":"claude-sonnet-4-5-20250929","max_tokens":32,"messages":[{"role":"user","content":"ping"}]}'
```

> 返回 `401` = 端点真实存在（只是要 key）；返回 HTML = 路径填错了。

---

## 关于本仓库的端点

示例里用的是 [夸父社API](https://kuafushe.cc)（同时兼容 OpenAI + Anthropic 原生协议，1:1 汇率，¥1 起充）。
换任何其他兼容端点，方法完全一样 —— **只改 `base_url`，代码一行不动**。

注册送 $0.2 体验额度：https://kuafushe.cc/register?aff=XB5VM25Z6A8C

---

## 免责

- 第三方中转**仅供学习与开发测试**，敏感数据请走官方 API
- 各站分组/价格/模型随时可能调整，以官网为准
- 用多少充多少，别一次充太多
- **合规前提**：`tutorials/` 下的安全向内容仅用于 CTF、授权渗透与安全研究，**禁止对未授权目标发起攻击**
