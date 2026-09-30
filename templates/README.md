# n8n 工作流模板

面向 n8n 用户的中文工作流模板，均接入 **Anthropic / OpenAI 兼容的 LLM API**。

> 端点：`https://kuafushe.cc/v1`（OpenAI 兼容）

## 可用模板

| 文件 | 功能 | 每天跑？ |
|------|------|----------|
| `ai-batch-rewrite.json` | 批量文案改写：读入一批文案 → AI 改写 → 输出结果 | 否（按需） |

## 使用步骤（以「AI 批量文案改写」为例）

### 1. 导入

n8n → 右上角 `...` → **Import from File** → 选择 `ai-batch-rewrite.json`

### 2. 创建凭据（只需一次）

因为模板**不包含任何密钥**，需要你自己建一个 HTTP Header 凭据：

1. 打开 `调用 API 改写` 节点
2. Authentication → **Generic Credential Type** → **Header Auth**
3. 新建凭据，填：

| 字段 | 值 |
|------|-----|
| Name | 任意，如 `kuafushe` |
| Header Name | `Authorization` |
| Header Value | `Bearer sk-你的密钥` |

> 密钥在控制台 → 令牌 里创建。OpenAI 协议用 `Authorization: Bearer sk-xxx`。

### 3. 运行

- 点「**示例文案**」节点，把 `texts` 换成你自己的文案列表
- 点 **Execute Workflow**，看「提取结果」节点输出的 `rewritten` 字段

### 4. 接你自己的数据源

`示例文案` 节点是演示用的。实际用时替换为：
- **Google Sheets / 飞书表格** 节点读入文案列
- 或 **Webhook** 接收外部传入
- 或 **Schedule Trigger** 定时跑

## 成本说明

- 模型：`gpt-6-astra`（默认，可在节点里改）
- 一次改写输出约几十 token，**成本很低**
- 汇率 1:1（¥1 = $1 额度），¥1 起充

## 模型怎么选（重要）

**模型取决于你 Key 绑定的分组**，不是所有 key 都能用所有模型。

先查你这个 key 能用哪些：

```bash
curl https://kuafushe.cc/v1/models \
  -H "Authorization: Bearer sk-你的密钥"
```

把返回里的模型名填进「调用 API 改写」节点的 `model` 字段。

> 示例：本模板实测用 `gpt-6-astra` 跑通。若你的分组有 Claude，可换成对应 ID。

## 注意

- Anthropic 原生协议端点不带 `/v1`（本模板用的是 OpenAI 协议 `/v1/chat/completions`）。
- ⚠️ **也不要填 `https://kuafushe.cc/anthropic`**（MiniMax 教程是 `/anthropic` 后缀）。本站没有该路径，会返回网页首页 HTML，导致静默失败。
- 模板已去除所有密钥，提交社区时符合「不包含敏感信息」的审核要求。
