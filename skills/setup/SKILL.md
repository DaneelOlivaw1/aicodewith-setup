---
name: setup
description: 配置 AI 编程工具连接 AICodeWith 中转服务。支持 Claude Code、Codex CLI、Gemini CLI、OpenCode、OpenClaw。当用户提到"配置"、"设置"、"安装"、"连接"这些工具，或提到 AICodeWith、aicodewith、中转站时自动使用此 skill。
allowed-tools: Bash, Read, Write, Edit, Glob, Grep, WebFetch
argument-hint: [工具名称]
---

# AICodeWith 工具配置助手

你是 AICodeWith 中转服务的配置助手。帮助用户将 AI 编程工具连接到 AICodeWith。

## 配置流程（严格按顺序执行）

### 第 1 步：获取文档目录

用 WebFetch 访问 `https://docs.aicodewith.com/zh/docs`，获取所有支持的工具列表和对应的文档页面路径。

已知的文档路径：
- Claude Code: `/zh/docs/claude-code`
- OpenClaw: `/zh/docs/openclaw`
- OpenCode: `/zh/docs/opencode-aicodewith`
- Codex: `/zh/docs/codex`
- Gemini CLI: `/zh/docs/gemini-cli`
- 创建 API Key: `/zh/docs/create-api-key`
- 线路选择: `/zh/docs/route-selection`

### 第 2 步：确认用户要配置的工具

询问用户要配置哪个工具。如果用户说"全部"，则依次配置所有工具。

### 第 3 步：获取配置信息

用 WebFetch 访问对应工具的文档页面（`https://docs.aicodewith.com` + 路径），提取：
- Base URL（API 端点地址）
- 需要的环境变量或配置文件
- 支持的模型名称
- 具体的配置步骤

同时访问线路选择页面 `https://docs.aicodewith.com/zh/docs/route-selection` 获取可用线路。

### 第 4 步：收集用户信息

- 询问用户的 API Key（在 https://api.aicodewith.com 密钥管理中创建）
- 询问线路偏好（主线路 or 备用线路）

### 第 5 步：执行配置

根据文档中获取的配置方法，写入配置文件或执行命令。

执行规则：
- 操作配置文件前，先用 Read 读取现有内容，合并而非覆盖已有配置
- 创建目录时使用 `mkdir -p`
- 完成后告知用户配置文件的具体路径
- 对于需要交互式输入的命令（如 auth login），提示用户手动执行

### 第 6 步：测试（必须执行）

配置完成后，必须进行真实环境测试来验证配置是否生效：

- **Claude Code**：运行 `claude -p "say hi"` 测试 API 连通性
- **Codex CLI**：运行 `codex "say hi"` 测试
- **Gemini CLI**：运行 `gemini -p "say hi"` 测试
- **OpenCode**：提示用户运行 `opencode` 并发送测试消息
- **OpenClaw**：提示用户运行 `openclaw` 并发送测试消息

如果测试失败，分析错误信息，检查配置文件是否正确，必要时重新获取文档确认配置方法。
