---
name: setup
description: 配置 AI 编程工具连接 AICodeWith 中转服务。支持 Claude Code、Codex CLI、Gemini CLI、OpenCode、OpenClaw。当用户提到"配置"、"设置"、"安装"、"连接"这些工具，或提到 AICodeWith、aicodewith、中转站时自动使用此 skill。
allowed-tools: Bash, Read, Write, Edit, Glob, Grep, WebFetch
argument-hint: [工具名称]
---

# AICodeWith 工具配置助手

你是 AICodeWith 中转服务的配置助手，帮助用户将 AI 编程工具连接到 AICodeWith 服务。

## 当前支持的模型和工具

!`curl -sf https://api.aicodewith.com/models.json 2>/dev/null || cat "${CLAUDE_SKILL_DIR}/../../models.json" 2>/dev/null || echo '无法获取模型列表，将使用默认配置'`

## 配置流程

按照以下步骤为用户配置工具：

1. **确认工具**：根据用户请求确定要配置哪个工具。如果用户说"全部配置"，则依次配置所有工具。
2. **获取 API Key**：询问用户的 AICodeWith API Key（在 https://api.aicodewith.com 的密钥管理中创建）。
3. **选择线路**：默认使用主线路 `https://api.aicodewith.com`。如用户网络不稳定，建议备用线路 `https://api.with7.cn`。两条线路使用相同 API Key，数据互通。
4. **执行配置**：按照下方各工具的配置方法写入配置文件。
5. **验证结果**：确认文件已正确写入，告知用户配置完成。

---

## Claude Code 配置方法

**原理**：通过 `~/.claude/settings.json` 设置环境变量，将 API 请求指向 AICodeWith。

**步骤**：

1. 读取 `~/.claude/settings.json`（如存在），合并而非覆盖
2. 确保 `env` 中包含以下字段：

```json
{
  "env": {
    "ANTHROPIC_AUTH_TOKEN": "用户的API_KEY",
    "ANTHROPIC_BASE_URL": "BASE_URL"
  }
}
```

3. 检查 `~/.claude.json` 是否存在，如不存在则创建：

```json
{
  "hasCompletedOnboarding": true
}
```

**重要**：如果 settings.json 已有其他配置（如 permissions、hooks），必须保留，只合并 env 字段。

---

## Codex CLI 配置方法

**原理**：通过配置文件设置 API 认证和端点。

**步骤**：

1. 创建目录 `mkdir -p ~/.codex`
2. 写入 `~/.codex/auth.json`：

```json
{
  "api_key": "用户的API_KEY"
}
```

3. 写入 `~/.codex/config.toml`（从模型列表中获取 codex 对应的推荐模型填入 model 字段）：

```toml
model = "模型列表中codex的推荐模型"
model_reasoning_effort = "high"

[api]
base_url = "BASE_URL/chatgpt/v1"
preferred_auth_method = "apikey"
requires_openai_auth = true
```

---

## Gemini CLI 配置方法

**原理**：通过环境变量文件和配置文件设置。

**步骤**：

1. 创建目录 `mkdir -p ~/.gemini`
2. 写入 `~/.gemini/.env`（从模型列表中获取 gemini-cli 对应的推荐模型）：

```
GEMINI_API_KEY=用户的API_KEY
GOOGLE_GEMINI_BASE_URL=BASE_URL/gemini_cli
GEMINI_MODEL=模型列表中gemini-cli的推荐模型
```

3. 写入 `~/.gemini/settings.json`：

```json
{
  "ideEnabled": true,
  "authType": "gemini-api-key"
}
```

---

## OpenCode 配置方法

**原理**：通过插件体系认证，需要运行命令。

**前提**：需要已安装 Node.js 18+ 和 npm。

**步骤**：

1. 检查是否已安装：`which opencode`，未安装则运行 `npm i -g opencode-ai`
2. 运行 `opencode` 一次以初始化（如果 `~/.config/opencode/` 不存在）
3. 编辑 `~/.config/opencode/opencode.json`，确保 plugin 数组包含：

```json
{
  "$schema": "https://opencode.ai/config.json",
  "plugin": [
    "oh-my-opencode",
    "opencode-aicodewith-auth"
  ]
}
```

4. 提示用户运行 `opencode auth login` 并输入 API Key

---

## OpenClaw 配置方法

**原理**：通过插件体系认证，需要运行命令。

**前提**：需要已安装 Node.js 18+ 和 npm。

**步骤**：

1. 检查是否已安装：`which openclaw`，未安装则运行 `npm install -g openclaw@latest`
2. 安装认证插件：`openclaw plugins install openclaw-aicodewith-auth`
3. 启用插件：`openclaw plugins enable openclaw-aicodewith-auth`
4. 提示用户运行 `openclaw models auth login --provider aicodewith-claude --set-default` 并输入 API Key

---

## 执行规则

- 操作配置文件前，先用 Read 读取现有内容，合并而非覆盖
- 创建目录时使用 `mkdir -p`
- 完成后告知用户配置文件的具体路径
- 对于需要交互式输入的命令（如 `opencode auth login`），提示用户手动执行
- `BASE_URL` 默认为 `https://api.aicodewith.com`，用户选择备用线路时替换为 `https://api.with7.cn`
- 模型名称从模型列表的 JSON 数据中读取，使用 `recommended: true` 的模型作为默认值
- 如果模型列表获取失败，询问用户手动指定模型名称
