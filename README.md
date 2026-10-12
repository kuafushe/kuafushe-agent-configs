# 中转站接入速查表

Claude Code、Codex、OpenAI SDK、n8n、Cursor、Cline 接入**任意 Anthropic / OpenAI 兼容端点**时的 `base_url`、报错对照与模型配置，一表查完。

写这个仓库的原因：接第三方端点时，`base_url` 和模型名这两处踩坑最多，报错还特别含糊，每次都要重查一遍。这里把所有坑固定成速查表。

---

## 速查表：各客户端该填哪个地址

**核心规则：Anthropic 协议不带 `/v1`，OpenAI 协议带 `/v1`。**

| 客户端 | 协议 | Base URL | 说明 |
|---|---|---|---|
| **Claude Code** | Anthropic Messages | `https://你的站点` | 客户端自己拼 `/v1/messages` |
| **claude-agent-sdk** | Anthropic Messages | `https://你的站点` | 同上 |
| **Codex** | OpenAI Responses | `https://你的站点/v1` | 带 `/v1` |
| **OpenAI SDK / Cline** | OpenAI Chat | `https://你的站点/v1` | 带 `/v1` |
| **n8n / Dify** | OpenAI Chat | `https://你的站点/v1` | 带 `/v1` |
| **Cursor** | OpenAI Chat | `https://你的站点/v1` | 带 `/v1` |

> 如果某个客户端的文档说「Base URL 填 `https://api.xxx.com`」，别顺手加 `/v1`。它自己会拼。多写一层就是 404。

---

## ⚠️ 三大坑

### 1. Anthropic 协议不带 `/v1`

客户端自己会拼 `/v1/messages`：

```text
https://example.com        ✅  →  https://example.com/v1/messages
https://example.com/v1     ❌  →  https://example.com/v1/v1/messages → 404
```

**报错不会提示你「多写了一层」**，只给你一个光秃秃的 Not Found。这是接中转站最常见的翻车点。

### 2. 不要填 `/anthropic` 后缀

MiniMax 那类教程用 `https://xxx/anthropic`，但**不是每个站都有这个路径**。

没有的话返回网站首页 HTML —— SDK 解析失败，报错极其含糊，属于**静默失败**。看到「200 但返回一堆 HTML」就是这个原因。

### 3. 模型名取决于你的 Key 分组

一个 Key 不一定有全部模型。撞到 `model_not_found` 或 404 先查：

```bash
curl https://你的站点/v1/models -H "Authorization: Bearer sk-你的密钥"
```

> 注意：部分站点未开放 `/v1/models`，此时报 404 属正常，去控制台看分组说明里有哪些模型。

---

## 报错对照表

| 现象 | 原因 | 处理 |
|---|---|---|
| **404 Not Found** | `base_url` 多写了 `/v1`，拼成 `/v1/v1/...` | 去掉 `/v1` |
| **401 Unauthorized** | 密钥错误 / 过期 / 分组不支持该模型 | 重新生成密钥，核对模型名 |
| **402 / 余额不足** | 额度用完 | 充值 |
| **200 但返回 HTML** | 请求打到了网站首页而非 API | 路径填错，或缺 `/anthropic` 后缀 |
| **403 Forbidden** | 分组无权访问该模型 | 换分组或换模型 |
| **`model_not_found`** | 模型名拼写错，或你的分组没开放 | 查控制台分组说明 |

**一个快速判别法：**

```bash
curl -i https://你的站点/v1/messages \
  -H "x-api-key: sk-你的密钥" \
  -H "anthropic-version: 2023-06-01" \
  -H "content-type: application/json" \
  -d '{"model":"claude-sonnet-4-5","max_tokens":32,"messages":[{"role":"user","content":"ping"}]}'
```

- 返回 **401** = 端点真实存在，只是缺有效 key ✅
- 返回 **HTML** = 路径填错了 ❌

---

## 目录

| 路径 | 内容 |
|------|------|
| [`configs/claude-code.md`](configs/claude-code.md) | Claude Code 接入 + `base_url` 三坑 + 报错对照表 |
| [`configs/providers.json.example`](configs/providers.json.example) | claude-agent-sdk 项目用的 `providers.json` 示例（可直接复制） |
| [`tutorials/ai-pentest-agent.md`](tutorials/ai-pentest-agent.md) | 给 AI 渗透 Agent 配 Claude API（含分组选型、成本对比） |
| [`templates/README.md`](templates/README.md) | n8n 工作流模板（含导入与凭据步骤） |

---

## 快速接入

**Claude Code：**

```bash
export ANTHROPIC_BASE_URL="https://你的站点"
export ANTHROPIC_AUTH_TOKEN="sk-你的密钥"
claude
```

**OpenAI SDK：**

```python
from openai import OpenAI
client = OpenAI(api_key="sk-你的密钥", base_url="https://你的站点/v1")
```

---

## 关于示例端点

文中的可运行示例用的是 [夸父社API](https://kuafushe.cc)（OpenAI + Anthropic 双协议，1:1 汇率，¥1 起充，新用户送 $0.2 体验额度，注册入口见[此处](https://kuafushe.cc/register?aff=XB5VM25Z6A8C)）。

换任何其他兼容端点，方法完全一样 —— **只改 `base_url`，代码一行不动**。

---

## 免责

- 第三方中转**仅供学习与开发测试**，敏感数据请走官方 API
- 各站分组 / 价格 / 模型随时可能调整，以官网为准
- 用多少充多少，别一次充太多
- **合规前提**：`tutorials/` 下的安全向内容仅用于 CTF、授权渗透与安全研究，**禁止对未授权目标发起攻击**