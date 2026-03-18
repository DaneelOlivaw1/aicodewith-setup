# AICodeWith Setup Plugin

一键配置 AI 编程工具连接 AICodeWith 中转服务。

## 支持的工具

| 工具 | 配置方式 | 官方文档 |
|------|----------|----------|
| Claude Code | 环境变量 → settings.json | [env-vars](https://code.claude.com/docs/en/env-vars) |
| Codex CLI | 自定义 provider → config.yaml | [codex-cli](https://github.com/openai/codex/tree/main/codex-cli) |
| Gemini CLI | 环境变量 → .env | [gemini-cli](https://github.com/google-gemini/gemini-cli) |
| OpenCode | 自定义 provider → opencode.json | [providers](https://opencode.ai/docs/providers/) |
| OpenClaw | 自定义 provider → models.providers | [configuration-reference](https://docs.openclaw.ai/gateway/configuration-reference) |

## 安装

在 Claude Code 中执行：

```
/plugin marketplace add DaneelOlivaw1/aicodewith-setup
/plugin install aicodewith-setup@DaneelOlivaw1-aicodewith-setup
```

## 使用

安装后，直接说：

- "帮我配置 Claude Code"
- "配置 Gemini CLI"
- "把所有工具都配置好"

或直接调用 `/aicodewith-setup:setup`

## API Key

登录 [AICodeWith](https://api.aicodewith.com) → 密钥管理 → 创建密钥

## 线路

| 线路 | 地址 | 说明 |
|------|------|------|
| 主线路 | `https://api.aicodewith.com` | 推荐，经 Cloudflare |
| 备用线路 | `https://api.with7.cn` | 国内直连 |

两条线路共享 API Key 和数据，可随时切换。

## 工作原理

Skill 内置了各工具的配置方法（基于各工具官方文档），不依赖外部服务。配置方法变更时更新 SKILL.md 即可。
