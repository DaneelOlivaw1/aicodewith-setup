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

通过环境变量将请求路由到自定义端点。配置写入 `~/.claude/settings.json` 的 `env` 字段。`ANTHROPIC_API_KEY` 必须设为空字符串（避免 Claude Code 用它直接鉴权），真正的 key 放在 `ANTHROPIC_AUTH_TOKEN`（确保交互模式下跳过登录）。

**步骤**：

1. 读取 `~/.claude/settings.json`（如存在），保留已有配置
2. 在 `env` 中设置：

```json
{
  "env": {
    "ANTHROPIC_API_KEY": "",
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

> **Reasoning 规则**：检查每个模型的 `supports_reasoning` 字段。如果为 `true`，则在该模型的配置中添加 `"reasoning": true`。如果为 `false`，则**不要**添加 `reasoning` 字段。上面示例中所有模型都带有 `reasoning: true`，因为它们都支持推理。对于不支持推理的模型（如某些轻量 instruct 模型），不要添加此字段。**以 `/models` 接口实际返回的 `supports_reasoning` 为准，不要猜测。**

### 步骤 4: 设置默认模型

在 `~/.config/opencode/opencode.json` 中设置默认模型（建议选 anthropic 类最强模型）：

```json
{
  "model": "aicodewith-anthropic/<最强claude模型id>"
}
```

**测试**: `opencode run -m "aicodewith-anthropic/<模型名>" "say hi"`

---

## OpenClaw

> 官方文档: https://docs.openclaw.ai/gateway/configuration-reference 及 https://docs.openclaw.ai/concepts/model-providers
> OpenClaw 通过 `models.providers` 配置自定义 provider。配置格式为 JSON5（支持注释和尾逗号）。每个厂商配置为独立 provider，便于在 UI 中区分和管理。模型引用格式为 `provider/model`（如 `aicodewith-claude/claude-opus-4-6`）。

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
2. 移除 `openclaw-aicodewith-auth` 插件（如存在）。该插件的 `cleanupStaleModels()` 会删除非内置 provider 的模型配置，导致 DeepSeek、Qwen 等模型消失：
   - 删除插件目录：`rm -rf ~/.openclaw/extensions/openclaw-aicodewith-auth`
   - 读取 `~/.openclaw/openclaw.json`，删除 `plugins.entries` 中 key 为 `openclaw-aicodewith-auth` 的条目（如存在）
3. 编辑配置文件 `~/.openclaw/openclaw.json`（JSON5 格式，支持注释和尾逗号），**按 provider（厂商）分组**，每个厂商一个 provider 条目。使用 `/models` 返回的真实值填充 `contextWindow`、`maxTokens`、`reasoning`、`input`：

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
>
> **模型字段说明**：`reasoning`（boolean）、`input`（数组，如 `["text", "image"]`）、`contextWindow`（number）、`maxTokens`（number）为推荐字段。省略时 OpenClaw 使用默认值：`reasoning: false`、`input: ["text"]`、`contextWindow: 200000`、`maxTokens: 8192`。可选字段 `cost`（`{ input, output, cacheRead, cacheWrite }`）省略时默认为 0。
>
> **注意**：对于使用 `api: "openai-completions"` 且 `baseUrl` 非 `api.openai.com` 的 provider（即所有 AICodeWith 代理），OpenClaw 会自动设置 `compat.supportsDeveloperRole: false`，避免不支持 `developer` 角色的 provider 返回 400 错误，无需手动配置。

4. 在 `agents.defaults.model.primary` 字段设置默认模型，并在 `agents.defaults.models` 中注册所有模型：

```json
{
  "agents": {
    "defaults": {
      "model": {
        "primary": "aicodewith-claude/<最强claude模型>",
        "fallbacks": ["aicodewith-openai/<备选模型>"]
      },
      "models": {
        "aicodewith-claude/<模型名>": {},
        "aicodewith-openai/<模型名>": {},
        "aicodewith-gemini/<模型名>": {},
        "aicodewith-deepseek/<模型名>": {}
      }
    }
  }
}
```

> **重要**：`agents.defaults.models` 是一个**白名单（catalog + allowlist）**——只有列在这里的模型才会出现在 `openclaw models list` 和 `/model` 命令中。必须遍历 `models.providers` 中定义的**所有模型**，以 `"<provider-id>/<model-id>": {}` 格式全部添加到此字段。遗漏任何模型都会导致该模型不出现在模型列表中。每个条目可选包含 `alias`（快捷名）和 `params`（如 `temperature`、`maxTokens`、`cacheRetention`），空对象 `{}` 即可使用默认值。

> **`model` 字段**：`agents.defaults.model` 接受字符串（如 `"aicodewith-claude/claude-opus-4-6"`）或对象（`{ primary, fallbacks }`）。建议使用对象形式配置 fallback 链以实现模型故障转移。

> **注意**：不要用 `openclaw models set` 命令设置默认模型，该命令会重写 `agents.defaults.models`，导致其他模型从列表中消失。始终直接编辑配置文件。

**测试**: `openclaw agent --local --to "+10000000000" --message "say hi" --json`

---

## Hermes Agent

> 官方文档: https://hermes-agent.nousresearch.com/docs/integrations/providers#custom--self-hosted-llm-providers
> Hermes 通过 `config.yaml` 的 `custom_providers` 列表配置自定义 API 端点。每个 provider 指定 `base_url`、`api_key` 和 `api_mode`。

**前提**: Python 3.11+，已安装 Hermes Agent（`pip install hermes-agent` 或 git clone 安装）

**适配器选择**（按模型的 `api_format` 字段）：

| api_format | Hermes `api_mode` | `base_url` 格式 |
|----------|--------|-------------|
| `anthropic` | `anthropic_messages` | `<BASE_URL>`（不带 /v1） |
| `openai-responses` | `chat_completions` | `<BASE_URL>/v1` |
| `openai-completions` | `chat_completions` | `<BASE_URL>/v1` |
| `gemini` | `chat_completions` | `<BASE_URL>/gemini_cli/v1beta` |

> **注意**：
> - `anthropic` 类型使用 `anthropic_messages`，base_url **不带** `/v1`（Anthropic SDK 自动拼接路径）
> - `openai-responses` 和 `openai-completions` 统一使用 `chat_completions`（Hermes 内部统一通过 OpenAI SDK 处理）
> - Gemini 类型也走 `chat_completions`，通过 OpenAI 兼容端点

**步骤**：

1. 确认安装：`hermes --version`（如未安装，参考 https://github.com/NousResearch/hermes-agent 安装）
2. `mkdir -p ~/.hermes`
3. 获取模型列表后，**按供应商（`provider` 字段）分组**，每个供应商建一个独立的 custom_provider 条目。编辑 `~/.hermes/config.yaml`：

```yaml
model:
  default: <默认模型id，如 claude-opus-4-6>
  provider: <默认provider名，如 aicodewith-claude>

custom_providers:
- name: aicodewith-claude
  base_url: <BASE_URL>
  api_key: <用户的KEY>
  api_mode: anthropic_messages
- name: aicodewith-openai
  base_url: <BASE_URL>/v1
  api_key: <用户的KEY>
  api_mode: chat_completions
- name: aicodewith-gemini
  base_url: <BASE_URL>/gemini_cli/v1beta
  api_key: <用户的KEY>
  api_mode: chat_completions
- name: aicodewith-deepseek
  base_url: <BASE_URL>/v1
  api_key: <用户的KEY>
  api_mode: chat_completions
- name: aicodewith-qwen
  base_url: <BASE_URL>/v1
  api_key: <用户的KEY>
  api_mode: chat_completions
- name: aicodewith-step
  base_url: <BASE_URL>/v1
  api_key: <用户的KEY>
  api_mode: chat_completions
- name: aicodewith-bytedance
  base_url: <BASE_URL>/v1
  api_key: <用户的KEY>
  api_mode: chat_completions

compression:
  summary_model: <anthropic类中最便宜的模型，如 claude-haiku-4-5-20251001>
```

> **关键注意事项**：
> - `model.default` 只写模型名（如 `claude-opus-4-6`），**不要**带 provider 前缀（如 ~~`aicodewith-claude/claude-opus-4-6`~~），否则会 400 报错"模型不存在"
> - `model.provider` 必须与 `custom_providers` 中某个条目的 `name` 一致
> - `compression.summary_model` 用于上下文压缩/摘要，建议选 anthropic 类中最便宜快速的模型
> - 按 `/models` 接口返回的 `provider` 字段命名各 custom_provider（格式：`aicodewith-<provider值>`）
> - 如果 `.env` 中有 `OPENAI_API_KEY` 或 `ANTHROPIC_API_KEY`，**建议删除**，避免 Hermes auto-detect 误走其他路径

4. 清理 `~/.hermes/.env` 中可能冲突的变量（如存在）：
   - 删除 `OPENAI_API_KEY`、`ANTHROPIC_API_KEY`、`OPENROUTER_API_KEY` 行（如存在）
   - 这些变量会导致 Hermes 的辅助客户端（auxiliary client）绕过 custom_provider 直接走其他路径

**测试**: `hermes chat -m "say hi"` 或通过已配置的 Telegram/飞书平台发送消息

**切换模型**：在 Hermes 会话中使用 `/model custom:<provider名>:<模型名>` 切换，如：
- `/model custom:aicodewith-claude:claude-sonnet-4-6`
- `/model custom:aicodewith-openai:gpt-5.2`
- `/model custom:aicodewith-deepseek:deepseek-r1`

---

## 执行规则

- 获取模型列表必须用 Bash 执行 `curl` 命令，不要使用 WebFetch 或 Fetch 工具（会被域名安全策略拦截）
- 操作配置文件前，先 Read 读取现有内容，合并而非覆盖
- 创建目录用 `mkdir -p`
- 完成后告知配置文件的具体路径
- 交互式命令（auth login 等）提示用户手动执行，不要尝试在 Bash 中运行
- 测试失败时分析错误信息并修复配置
