---
name: setup
description: 配置 AI 编程工具连接 AICodeWith 中转服务。支持 Claude Code、Codex CLI、Gemini CLI、OpenCode、OpenClaw。当用户提到"配置"、"设置"、"安装"、"连接"这些工具，或提到 AICodeWith、aicodewith、中转站时自动使用此 skill。
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
3. 按下方对应工具的方法执行配置
4. 必须测试验证

---

## Claude Code

> 官方文档: https://code.claude.com/docs/en/env-vars

通过环境变量 `ANTHROPIC_API_KEY` 和 `ANTHROPIC_BASE_URL` 将请求路由到自定义端点。配置写入 `~/.claude/settings.json` 的 `env` 字段。

**步骤**：

1. 读取 `~/.claude/settings.json`（如存在），保留已有配置
2. 在 `env` 中设置：

```json
{
  "env": {
    "ANTHROPIC_API_KEY": "<用户的KEY>",
    "ANTHROPIC_BASE_URL": "<BASE_URL>"
  }
}
```

3. 确保 `~/.claude.json` 存在且包含 `{"hasCompletedOnboarding": true}`

**测试**: `claude -p "say hi"`

---

## Codex CLI

> 官方文档: https://github.com/openai/codex/tree/main/codex-cli
> Codex 支持任何 OpenAI 兼容 API，通过 provider 配置自定义 baseURL 和 envKey。

**步骤**：

1. `mkdir -p ~/.codex`
2. 写入 `~/.codex/config.yaml`：

```yaml
model: <用户选择的模型，默认 gpt-5.3-codex>
provider: aicodewith

providers:
  aicodewith:
    name: "AICodeWith"
    baseURL: "<BASE_URL>/chatgpt/v1"
    envKey: "AICODEWITH_API_KEY"
```

3. 设置环境变量（写入 shell profile 或 `.env`）：

```bash
export AICODEWITH_API_KEY="<用户的KEY>"
```

**测试**: `codex "say hi"`

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
> OpenCode 支持通过 `provider` 配置添加任意 OpenAI 兼容 API，使用 `@ai-sdk/openai-compatible` 包。

**前提**: Node.js 18+

**步骤**：

1. 安装（如未安装）: `npm i -g opencode-ai`
2. 运行 `opencode` 一次初始化配置目录
3. 运行 `/connect`，选择 "Other"，输入 provider ID（如 `aicodewith`），输入 API Key
4. 编辑配置文件（`~/.config/opencode/opencode.json` 或 `~/.opencode.json`），添加 provider：

```json
{
  "$schema": "https://opencode.ai/config.json",
  "provider": {
    "aicodewith": {
      "npm": "@ai-sdk/openai-compatible",
      "name": "AICodeWith",
      "options": {
        "baseURL": "<BASE_URL>/v1",
        "apiKey": "{env:AICODEWITH_API_KEY}"
      },
      "models": {
        "claude-sonnet-4": {
          "name": "Claude Sonnet 4",
          "limit": { "context": 200000, "output": 65536 }
        }
      }
    }
  }
}
```

5. 模型名称和列表根据用户实际需求调整

**测试**: 提示用户运行 `opencode` 并发送测试消息

---

## OpenClaw

> 官方文档: https://docs.openclaw.ai/gateway/configuration-reference
> OpenClaw 通过 `models.providers` 配置自定义 provider，支持 `openai-completions`、`anthropic-messages` 等适配器。

**前提**: Node.js 18+

**步骤**：

1. 安装（如未安装）: `npm install -g openclaw@latest`
2. 运行 `openclaw onboard --install-daemon` 完成初始化
3. 编辑配置文件，在 `models.providers` 中添加自定义 provider：

```json5
{
  "models": {
    "mode": "merge",
    "providers": {
      "aicodewith": {
        "baseUrl": "<BASE_URL>",
        "apiKey": "<用户的KEY>",
        "api": "anthropic-messages",
        "models": [
          {
            "id": "claude-sonnet-4",
            "name": "Claude Sonnet 4",
            "reasoning": false,
            "input": ["text"],
            "contextWindow": 200000,
            "maxTokens": 65536
          }
        ]
      }
    }
  }
}
```

4. 设置默认模型: `openclaw models set aicodewith/claude-sonnet-4`
5. 模型名称、api 适配器和列表根据用户实际需求调整（如用 OpenAI 兼容接口则 api 改为 `openai-completions`，baseUrl 加 `/v1`）

**测试**: 提示用户运行 `openclaw` 并发送测试消息

---

## 执行规则

- 操作配置文件前，先 Read 读取现有内容，合并而非覆盖
- 创建目录用 `mkdir -p`
- 完成后告知配置文件的具体路径
- 交互式命令（auth login 等）提示用户手动执行，不要尝试在 Bash 中运行
- 测试失败时分析错误信息并修复配置
