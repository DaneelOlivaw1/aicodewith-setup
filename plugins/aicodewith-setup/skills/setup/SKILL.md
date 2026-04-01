---
name: setup
description: 配置 AI 编程工具或调用大模型 API 连接 AICodeWith 中转服务。支持 Claude Code、Codex CLI、Gemini CLI、OpenCode、OpenClaw 等编程工具配置，以及通过 Python、JS 等语言直接调用 Claude、GPT、Gemini、DeepSeek 等大模型 API。当用户提到"配置"、"设置"、"连接"这些工具，或想用代码调用 AI、大模型、LLM API 时自动使用此 skill。
allowed-tools: Bash, Read, Write, Edit, Glob, Grep
argument-hint: [工具名称]
---

# AICodeWith 工具配置助手

帮助用户将 AI 编程工具连接到 AICodeWith 中转服务。

## 服务信息

- 主线路: `https://api.aicodewith.com`（推荐，经 Cloudflare → 日本 → 美国）
- 备用线路: `https://api.with7.cn`（国内直连，经阿里云 ESA → 日本 → 美国）
- 两条线路使用相同 API Key，数据互通，可随时切换
- API Key 在 https://api.aicodewith.com 密钥管理中创建

## 配置流程

1. 确认用户要配置哪个工具（可多选或全部）
2. 收集 API Key 和线路偏好（默认主线路）
3. 获取模型列表：**必须用 Bash 执行 `curl -s <BASE_URL>/models`**（不要使用 Fetch/WebFetch 工具，会被安全策略拦截），解析返回的 JSON。响应格式：
   ```json
   {
     "data": [
       {
         "id": "claude-sonnet-4-6",
         "name": "Sonnet 4.6",
         "provider": "anthropic",
         "context_window": 200000,
         "max_output_tokens": 128000,
         "supports_reasoning": true,
         "supports_temperature": true,
         "supports_tool_call": true,
         "input_modalities": ["text", "image"],
         "output_modalities": ["text"],
         "api_format": "anthropic"
       }
     ]
   }
   ```
   按 `api_format` 字段分组配置：
   - `anthropic` → Claude 类型（Anthropic Messages API）
   - `openai-responses` → OpenAI Responses API（GPT 系列）
   - `openai-completions` → OpenAI Chat Completions（DeepSeek、GLM、Kimi、Qwen 等）
   - `gemini` → Gemini 类型（Google Generative AI）
4. 按下方对应工具的方法执行配置，使用返回的 `context_window`、`max_output_tokens`、`supports_reasoning`、`supports_temperature`、`supports_tool_call`、`input_modalities`、`output_modalities` 填充配置，不要硬编码。**特别注意**：如果模型的 `input_modalities` 包含 `"image"`，必须在配置中声明 `modalities` 和 `attachment` 字段；如果模型的 `supports_reasoning` 为 `true`，必须声明 `"reasoning": true`（具体格式见各工具配置示例），否则该模型无法使用视觉和推理能力
5. 必须测试验证

---

## Claude Code

> 官方文档: https://code.claude.com/docs/en/env-vars

通过环境变量将请求路由到自定义端点。配置写入 `~/.claude/settings.json` 的 `env` 字段。需要同时设置 `ANTHROPIC_API_KEY` �� `ANTHROPIC_AUTH_TOKEN`，前者用于非交互模式，后者确保交互模式下跳过登录。

**步骤**：

1. 读取 `~/.claude/settings.json`（如存在），保留已有配置
2. 在 `env` 中设置：

```json
{
  "env": {
    "ANTHROPIC_API_KEY": "<用户的KEY>",
    "ANTHROPIC_AUTH_TOKEN": "<用户的KEY>",
    "ANTHROPIC_BASE_URL": "<BASE_URL>"
  }
}
```

3. 将从模型列表中获取的所有 Claude 模型（`provider` 为 `anthropic`）写入 `availableModels` 字段：

```json
{
  "env": { ... },
  "availableModels": ["claude-sonnet-4-6", "claude-opus-4-6", ...]
}
```

4. 确保 `~/.claude.json` 存在且包含 `{"hasCompletedOnboarding": true}`

**测试**: `claude -p "say hi"`

---

## Codex CLI

> 官方文档: https://github.com/openai/codex/tree/main/codex-cli
> Codex 通过 `config.toml` 配置自定义 API 端点，通过 `codex login --with-api-key` 存储凭证。

**前提**: Node.js 18+，需在 git 仓库内运行

**步骤**：

1. 安装（如未安装）: `npm install -g @openai/codex`
2. `mkdir -p ~/.codex`
3. 写入 `~/.codex/config.toml`：

```toml
model = "<用户选择的模型，默认 gpt-5.3-codex>"
openai_base_url = "<BASE_URL>/v1"
```

4. 登录存储 API Key（必须通过此方式，环境变量 OPENAI_API_KEY 不生效）：

```bash
echo "<用户的KEY>" | codex login --with-api-key
```

**测试**: 在 git 仓库目录下运行 `codex exec "say hi"`

---

## Gemini CLI

> 官方文档: https://github.com/google-gemini/gemini-cli
> 支持通过 `GOOGLE_GEMINI_BASE_URL` 环境变量指定自定义 API 端点（参考 issue #1679）。

**步骤**：

1. `mkdir -p ~/.gemini`
2. 写入 `~/.gemini/.env`：

```
GEMINI_API_KEY=<用户的KEY>
GOOGLE_GEMINI_BASE_URL=<BASE_URL>/gemini_cli
GEMINI_MODEL=<用户选择的模型，默认 gemini-3-pro>
```

3. 写入 `~/.gemini/settings.json`：

```json
{
  "authType": "gemini-api-key"
}
```

**测试**: `gemini -p "say hi"`

---

## OpenCode

> 官方文档: https://opencode.ai/docs/providers/
> OpenCode 通过 `provider` 配置添加自定义 API 提供商。不同模型类型使用不同的 npm 适配器包。
> 配置 OpenCode 同时会安装 oh-my-opencode（强大的多 agent 编排框架），并将所有 agent 模型替换为 AICodeWith 模型。

**前提**: Node.js 18+

### 步骤 1: 安装 OpenCode（如未安装）

```bash
npm i -g opencode-ai
```

### 步骤 2: 清理旧版 opencode-aicodewith-auth 插件（如存在）

读取 `~/.config/opencode/opencode.json`（如存在），检查是否安装了旧版 `opencode-aicodewith-auth` 插件。

**检测方式**：在 `plugin` 数组中查找包含 `opencode-aicodewith-auth` 的条目（可能是以下任意形式）：
- `"opencode-aicodewith-auth"`
- `"opencode-aicodewith-auth@x.y.z"`
- `"file:///...opencode-aicodewith-auth/dist/index.js"`
- 任何包含 `opencode-aicodewith-auth` 字样的字符串

**如果检测到，执行清理**：
1. 从 `plugin` 数组中移除所有包含 `opencode-aicodewith-auth` 的条目
2. 从 `provider` 对象中移除 `"aicodewith"` 键（单 provider 配置，会被新的多 provider 配置替代）
3. 如果 `model` 字段以 `aicodewith/` 开头，先记下模型名（`aicodewith/` 之后的部分），后续会用新的 provider 前缀替换
4. 清理 `~/.local/share/opencode/auth.json` 中的 `"aicodewith"` 条目（如存在）
5. **保留** `oh-my-opencode` 相关条目（plugin 中的 `"oh-my-opencode"` 和 `oh-my-opencode.json` 文件），后续步骤会更新它们

**如果未检测到**，跳过此步。

### 步骤 3: 配置 OpenCode Provider

**适配器选择**（按模型的 `api_format` 字段）：

| api_format | npm 包 | baseURL 格式 |
|----------|--------|-------------|
| `openai-responses` | `@ai-sdk/openai` | `<BASE_URL>/v1` |
| `openai-completions` | `@ai-sdk/openai-compatible` | `<BASE_URL>/v1` |
| `anthropic` | `@ai-sdk/anthropic` | `<BASE_URL>/v1` |
| `gemini` | `@ai-sdk/google` | `<BASE_URL>/gemini_cli/v1beta` |

> **注意**：
> - `openai-responses`（GPT 系列）使用 OpenAI Responses API，必须用 `@ai-sdk/openai`；`openai-completions`（DeepSeek、GLM、Kimi、Qwen 等）使用 Chat Completions，必须用 `@ai-sdk/openai-compatible`，否则请求会挂起。
> - **Prompt Caching**：使用 `@ai-sdk/openai` 的 provider（即 `aicodewith-openai`）必须在 `options` 中添加 `"setCacheKey": true`。原因：OpenAI 的 prompt caching 是隐式的，要求请求路由到同一后端节点；通过代理时无法保证路由一致，设置 `setCacheKey` 后 OpenCode 会在每次请求中携带 `promptCacheKey`，确保相同上下文的请求命中缓存。Anthropic（`@ai-sdk/anthropic`）使用显式 `cache_control` 标记嵌在请求体中，通过代理天然有效，无需额外配置。

获取模型列表后，**按供应商（`provider` 字段）分组**，每个供应商建一个独立的 provider 条目。**不要按 `api_format` 合并，也不要过滤任何模型——完全以 `/models` 端点返回的数据为准。**

常见供应商与 provider ID 对应关系：

| 供应商 | provider ID | npm 包 |
|--------|-------------|--------|
| anthropic | `aicodewith-anthropic` | `@ai-sdk/anthropic` |
| openai | `aicodewith-openai` | `@ai-sdk/openai` |
| gemini | `aicodewith-gemini` | `@ai-sdk/google` |
| deepseek | `aicodewith-deepseek` | `@ai-sdk/openai-compatible` |
| glm | `aicodewith-glm` | `@ai-sdk/openai-compatible` |
| minimax | `aicodewith-minimax` | `@ai-sdk/openai-compatible` |
| kimi | `aicodewith-kimi` | `@ai-sdk/openai-compatible` |
| qwen | `aicodewith-qwen` | `@ai-sdk/openai-compatible` |
| step | `aicodewith-step` | `@ai-sdk/openai-compatible` |

编辑 `~/.config/opencode/opencode.json`，使用 `/models` 返回的真实值填充模型配置。

**模型字段映射**（`/models` API → OpenCode `opencode.json`）：

| `/models` 返回字段 | OpenCode 模型字段 | 说明 |
|---|---|---|
| `context_window` | `limit.context` | 上下文窗口大小 |
| `max_output_tokens` | `limit.output` | 最大输出 token 数 |
| `input_modalities` | `modalities.input` | 输入模态，如 `["text", "image"]` |
| `output_modalities` | `modalities.output` | 输出模态，如 `["text"]` |
| `supports_reasoning` | `reasoning` | 是否支持推理/思考模式 |
| `supports_temperature` | `temperature` | 是否支持 temperature 参数 |
| `supports_tool_call` | `tool_call` | 是否支持工具调用 |
| （根据 `input_modalities` 是否包含 `"image"`） | `attachment` | 是否启用附件/图片上传 UI |

> **关键**：`attachment` 和 `modalities` 字段决定了 OpenCode UI 是否允许上传图片和文件。如果模型的 `input_modalities` 包含 `"image"`，则必须设置 `"attachment": true` 和 `"modalities": {"input": ["text", "image"], "output": ["text"]}`，否则 OpenCode 不会显示图片上传功能。

```json
{
  "$schema": "https://opencode.ai/config.json",
  "provider": {
    "aicodewith-anthropic": {
      "npm": "@ai-sdk/anthropic",
      "name": "AICodeWith Anthropic",
      "options": {
        "baseURL": "<BASE_URL>/v1",
        "apiKey": "<用户的KEY>"
      },
      "models": {
        "claude-sonnet-4-6": {
          "name": "Sonnet 4.6",
          "attachment": true,
          "reasoning": true,
          "temperature": true,
          "tool_call": true,
          "modalities": { "input": ["text", "image"], "output": ["text"] },
          "limit": { "context": 200000, "output": 128000 }
        },
        "claude-opus-4-6": {
          "name": "Opus 4.6",
          "attachment": true,
          "reasoning": true,
          "temperature": true,
          "tool_call": true,
          "modalities": { "input": ["text", "image"], "output": ["text"] },
          "limit": { "context": 200000, "output": 128000 }
        }
      }
    },
    "aicodewith-openai": {
      "npm": "@ai-sdk/openai",
      "name": "AICodeWith OpenAI",
      "options": {
        "baseURL": "<BASE_URL>/v1",
        "apiKey": "<用户的KEY>",
        "setCacheKey": true
      },
      "models": {
        "gpt-5.2": {
          "name": "GPT-5.2",
          "attachment": true,
          "reasoning": true,
          "temperature": true,
          "tool_call": true,
          "modalities": { "input": ["text", "image"], "output": ["text"] },
          "limit": { "context": 200000, "output": 100000 }
        }
      }
    },
    "aicodewith-gemini": {
      "npm": "@ai-sdk/google",
      "name": "AICodeWith Gemini",
      "options": {
        "baseURL": "<BASE_URL>/gemini_cli/v1beta",
        "apiKey": "<用户的KEY>"
      },
      "models": {
        "gemini-3-pro": {
          "name": "Gemini 3 Pro",
          "attachment": true,
          "reasoning": true,
          "temperature": true,
          "tool_call": true,
          "modalities": { "input": ["text", "image"], "output": ["text"] },
          "limit": { "context": 1048576, "output": 65536 }
        }
      }
    },
    "aicodewith-deepseek": {
      "npm": "@ai-sdk/openai-compatible",
      "name": "AICodeWith DeepSeek",
      "options": {
        "baseURL": "<BASE_URL>/v1",
        "apiKey": "<用户的KEY>"
      },
      "models": {
        "deepseek-r1": {
          "name": "DeepSeek R1",
          "attachment": false,
          "reasoning": true,
          "temperature": true,
          "tool_call": true,
          "limit": { "context": 131072, "output": 65536 }
        }
      }
    }
  }
}
```

> **Modalities 规则**：遍历 `/models` 返回的每个模型，检查其 `input_modalities` 字段。如果包含 `"image"`，则在该模型的配置中添加 `"modalities": { "input": ["text", "image"], "output": ["text"] }`。如果只有 `["text"]`，则**不要**添加 `modalities` 字段。上面示例中 `claude-opus-4-6`、`claude-sonnet-4-6`、`gpt-5.2`、`gemini-3-pro` 带有 modalities 是因为它们支持视觉输入，而 `deepseek-r1` 不带是因为只支持文本。**以 `/models` 接口实际返回的 `input_modalities` 为准，不要猜测。**

> **Reasoning 规则**：检查每个模型的 `supports_reasoning` 字段。如果为 `true`，则在该模型的配置中添加 `"reasoning": true`。如果为 `false`，则**不要**添加 `reasoning` 字段。上面示例中所有模型都带有 `reasoning: true`，因为它们都支持推理。对于不支持推理的模型（如某些轻量 instruct 模型），不要添加此字段。设置 `reasoning: true` 后，OpenCode 会为该模型启用内置的思考 variant（如 Anthropic 的 `high`/`max`、OpenAI 的 `low`/`medium`/`high`/`xhigh`、Gemini 的 `low`/`high`），OMO 的 agent/category 通过 `variant` 字段选择具体强度。**以 `/models` 接口实际返回的 `supports_reasoning` 为准，不要猜测。**

**记住你配置了哪些 provider ID**（如 `aicodewith-anthropic`、`aicodewith-openai`、`aicodewith-gemini`），以及每个 provider 下有哪些模型 ID。后面配置 oh-my-opencode 时需要用 `<provider-id>/<模型id>` 格式引用。

### 步骤 4: 安装 oh-my-opencode

oh-my-opencode 是一个 OpenCode 增强框架，提供多 agent 协作编排（Sisyphus、Oracle、Librarian 等）。必须先通过安装命令生成配置文件，然后再修改模型配置。

**4.1 运行安装命令**

```bash
bunx oh-my-opencode install --no-tui --claude no --openai no --gemini no --copilot no
```

> 所有订阅选项设为 `no`，因为我们使用 AICodeWith 作为统一认证层，不需要单独的 Claude/ChatGPT/Gemini 订阅。

此命令会：
- 将 `"oh-my-opencode"` 添加到 `~/.config/opencode/opencode.json` 的 `plugin` 数组
- 生成 `~/.config/opencode/oh-my-opencode.json` 配置文件

**4.2 验证安装**

确认以下文件存在且有效：
- `~/.config/opencode/opencode.json` 的 `plugin` 数组包含 `"oh-my-opencode"`
- `~/.config/opencode/oh-my-opencode.json` 存在且是有效 JSON

### 步骤 5: 配置 oh-my-opencode 使用 AICodeWith 模型

**5.1 读取生成的配置**

读取 `~/.config/opencode/oh-my-opencode.json`，解析其中的 `agents` 和 `categories` 对象。

**5.2 模型替换**

遍历配置中**所有** agent 和 category（不要硬编码列表，OMO 可能会新增），将每个的 `model` 字段替换为对应的 AICodeWith 模型。

模型 ID 格式为 `<provider-id>/<模型id>`，其中 `<provider-id>` 是步骤 3 中在 opencode.json 配置的 provider 名称，`<模型id>` 是 `/models` 接口返回的模型 ID。例如：`aicodewith-anthropic/claude-opus-4-6`、`aicodewith-openai/gpt-5.2`、`aicodewith-gemini/gemini-3-pro`。

**替换规则**——根据 agent/category 的角色语义，从步骤 3 获取的可用模型中选择最合适的：

**Agent 替换规则**：

| 角色类型 | 匹配的 Agent 名称 | 选择策略 |
|---------|------------------|---------|
| 主编排/深度推理 | sisyphus, prometheus, metis, build, plan, OpenCode-Builder | 选 `anthropic` 类中最强的模型（通常是 opus 级别） |
| 架构/审查/策略 | oracle, momus | 选 `openai-responses` 类中最强的非 codex 模型（通常是 gpt-5.x） |
| 代码生成 | hephaestus | 选 `openai-responses` 类中带 `codex` 的模型 |
| 前端/视觉/多模态 | frontend-ui-ux-engineer, document-writer, multimodal-looker | 选 `gemini` 类模型 |
| 通用/轻量任务 | librarian, explore, atlas, sisyphus-junior, general, 以及任何不认识的新 agent | 选 `anthropic` 类中 sonnet 级别（性价比最优） |

**Category 替换规则**：

| 类型 | 匹配的 Category 名称 | 选择策略 |
|------|---------------------|---------|
| 视觉/前端/创意/写作 | visual-engineering, visual, artistry, writing | 选 `gemini` 类模型 |
| 重度推理 | ultrabrain, deep | 选 `openai-responses` 类中最强的模型（codex 或 gpt-5.x） |
| 业务逻辑/高级通用 | business-logic, unspecified-high | 选 `openai-responses` 类中最强的非 codex 模型 |
| 快速/轻量/通用 | quick, unspecified-low, data-analysis, 以及任何不认识的新 category | 选 `anthropic` 类中 sonnet 级别 |

**遇到不认识的 agent/category 名称时**，按名称关键词推断：
- 包含 `visual`、`frontend`、`ui`、`ux`、`design`、`image` → gemini 类
- 包含 `oracle`、`review`、`architect`、`strategy`、`logic` → openai-responses 类最强
- 包含 `build`、`plan`、`orchestrat`、`primary` → anthropic 类最强
- 包含 `code`、`codex`、`generate` → openai-responses 类中 codex 模型
- 其他/无法判断 → anthropic 类 sonnet 级别（安全默认值）

**5.3 保留 variant 配置**

如果原配置中某个 agent/category 已有 `variant` 字段（如 `"variant": "high"`），保留不动。`variant` 控制推理强度，是 OMO 根据角色设定的，不需要我们改。

**5.4 设置其他字段**

- 设置 `"google_auth": false`（禁用 OMO 内置的 Google OAuth 认证，因为使用 AICodeWith）
- **保留**配置中的所有其他字段不动（`disabled_hooks`、`disabled_agents`、`disabled_skills`、`disabled_mcps`、`disabled_commands` 等）

**5.5 写回配置**

将修改后的配置写回 `~/.config/opencode/oh-my-opencode.json`。

**示例**——假设 `/models` 返回了 `claude-opus-4-6`（anthropic）、`claude-sonnet-4-5`（anthropic）、`gpt-5.2`（openai）、`gpt-5.3-codex`（openai）、`gemini-3-pro`（gemini），修改后的配置大致如下：

```json
{
  "$schema": "https://raw.githubusercontent.com/code-yeongyu/oh-my-opencode/master/assets/oh-my-opencode.schema.json",
  "google_auth": false,
  "agents": {
    "sisyphus": {
      "model": "aicodewith-anthropic/claude-opus-4-6",
      "variant": "max"
    },
    "oracle": {
      "model": "aicodewith-openai/gpt-5.2",
      "variant": "high"
    },
    "hephaestus": {
      "model": "aicodewith-openai/gpt-5.3-codex",
      "variant": "medium"
    },
    "librarian": {
      "model": "aicodewith-anthropic/claude-sonnet-4-5"
    },
    "explore": {
      "model": "aicodewith-anthropic/claude-sonnet-4-5"
    },
    "multimodal-looker": {
      "model": "aicodewith-gemini/gemini-3-pro"
    },
    "frontend-ui-ux-engineer": {
      "model": "aicodewith-gemini/gemini-3-pro"
    }
  },
  "categories": {
    "visual-engineering": {
      "model": "aicodewith-gemini/gemini-3-pro"
    },
    "ultrabrain": {
      "model": "aicodewith-openai/gpt-5.3-codex",
      "variant": "high"
    },
    "quick": {
      "model": "aicodewith-anthropic/claude-sonnet-4-5"
    },
    "business-logic": {
      "model": "aicodewith-openai/gpt-5.2"
    }
  }
}
```

> 上面只是示例，实际配置应包含 OMO 安装时生成的**所有** agent 和 category。模型 ID 以 `/models` 接口实际返回值为准。

### 步骤 6: 设置默认模型

在 `~/.config/opencode/opencode.json` 中设置默认模型（建议选 anthropic 类最强模型）：

```json
{
  "model": "aicodewith-anthropic/<最强claude模型id>"
}
```

**测试**: `opencode run -m "aicodewith-anthropic/<模型名>" "say hi"`

---

## OpenClaw

> 官方文档: https://docs.openclaw.ai/gateway/configuration-reference
> OpenClaw 通过 `models.providers` 配置自定义 provider。每个厂商配置为��立 provider，便于在 UI 中区分和管理。

**前提**: Node.js 18+，需要 git

**适配器选择**（按模型的 `api_format` 字段）：

| api_format | OpenClaw api 值 | baseUrl 格式 |
|----------|--------|-------------|
| `openai-responses` | `openai-responses` | `<BASE_URL>/v1` |
| `openai-completions` | `openai-completions` | `<BASE_URL>/v1` |
| `anthropic` | `anthropic-messages` | `<BASE_URL>` (不带 /v1) |
| `gemini` | `google-generative-ai` | `<BASE_URL>/gemini_cli/v1beta` |

**步骤**：

1. 安装（如未安装）: `npm install -g openclaw@latest`
2. 编辑配置文件 `~/.openclaw/openclaw.json`，**按 provider（厂商）分组**，每个厂商一个 provider 条目。使用 `/models` 返回的真实值填充 `contextWindow`、`maxTokens`、`reasoning`、`input`：

```json
{
  "models": {
    "mode": "merge",
    "providers": {
      "aicodewith-claude": {
        "baseUrl": "<BASE_URL>",
        "apiKey": "<用户的KEY>",
        "api": "anthropic-messages",
        "models": [
          {
            "id": "<模型id>",
            "name": "<模型name>",
            "reasoning": "<模型supports_reasoning>",
            "input": "<模型input_modalities>",
            "contextWindow": "<模型context_window>",
            "maxTokens": "<模型max_output_tokens>"
          }
        ]
      },
      "aicodewith-openai": {
        "baseUrl": "<BASE_URL>/v1",
        "apiKey": "<用户的KEY>",
        "api": "openai-responses",
        "models": [
          {
            "id": "<模型id>",
            "name": "<模型name>",
            "reasoning": "<模型supports_reasoning>",
            "input": "<模型input_modalities>",
            "contextWindow": "<模型context_window>",
            "maxTokens": "<模型max_output_tokens>"
          }
        ]
      },
      "aicodewith-gemini": {
        "baseUrl": "<BASE_URL>/gemini_cli/v1beta",
        "apiKey": "<用户的KEY>",
        "api": "google-generative-ai",
        "models": [
          {
            "id": "<模型id>",
            "name": "<模型name>",
            "reasoning": "<模型supports_reasoning>",
            "input": "<模型input_modalities>",
            "contextWindow": "<模型context_window>",
            "maxTokens": "<模型max_output_tokens>"
          }
        ]
      },
      "aicodewith-deepseek": {
        "baseUrl": "<BASE_URL>/v1",
        "apiKey": "<用户的KEY>",
        "api": "openai-completions",
        "models": [
          {
            "id": "<模型id>",
            "name": "<模型name>",
            "reasoning": "<模型supports_reasoning>",
            "input": "<模型input_modalities>",
            "contextWindow": "<模型context_window>",
            "maxTokens": "<模型max_output_tokens>"
          }
        ]
      },
      "aicodewith-qwen": {
        "baseUrl": "<BASE_URL>/v1",
        "apiKey": "<用户的KEY>",
        "api": "openai-completions",
        "models": [...]
      },
      "aicodewith-kimi": {
        "baseUrl": "<BASE_URL>/v1",
        "apiKey": "<用户的KEY>",
        "api": "openai-completions",
        "models": [...]
      },
      "aicodewith-glm": {
        "baseUrl": "<BASE_URL>/v1",
        "apiKey": "<用户的KEY>",
        "api": "openai-completions",
        "models": [...]
      },
      "aicodewith-minimax": {
        "baseUrl": "<BASE_URL>/v1",
        "apiKey": "<用户的KEY>",
        "api": "openai-completions",
        "models": [...]
      }
    }
  }
}
```

> 按 `/models` 接口返回的 `provider` 字段命名各 provider（格式：`aicodewith-<provider值>`）。所有 `openai-completions` 类型的厂商（DeepSeek、Qwen、Kimi、GLM、MiniMax 等）均使用 `<BASE_URL>/v1` + `openai-completions`，只是模型列表不同。

3. 在 `agents.defaults.model.primary` 字段设置默认模型：

```json
{
  "agents": {
    "defaults": {
      "model": {
        "primary": "aicodewith-claude/<模型名>"
      },
      "models": {
        "aicodewith-claude/<模型名>": {},
        "aicodewith-openai/<模型名>": {}
      }
    }
  }
}
```

> **重要**：`agents.defaults.models` 是一个**白名单**——只有列在这里的模型才会出现在 `openclaw models list` 中。必须遍历 `models.providers` 中定义的**所有模型**，以 `"<provider-id>/<model-id>": {}` 格式全部添加到此字段。遗漏任何模型都会导致该模型不出现在模型列表中。

> **注意**：不要用 `openclaw models set` 命令设置默认模型，该命令会重写 `agents.defaults.models`，导致其他模型从列表中消失。始终直接编辑配置文件。

**测试**: `openclaw agent --local --to "+10000000000" --message "say hi" --json`

---

## 执行规则

- 获取模型列表必须用 Bash 执行 `curl` 命令，不要使用 WebFetch 或 Fetch 工具（会被域名安全策略拦截）
- 操作配置文件前，先 Read 读取现有内容，合并而非覆盖
- 创建目录用 `mkdir -p`
- 完成后告知配置文件的具体路径
- 交互式命令（auth login 等）提示用户手动执行，不要尝试在 Bash 中运行
- 测试失败时分析错误信息并修复配置
