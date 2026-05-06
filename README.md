# AICodeWith Setup Plugin

一键配置 AI 编程工具连接 AICodeWith 中转服务。

## 支持的工具

| 工具 | 配置方式 | 官方文档 |
|------|----------|----------|
| Claude Code | 环境变量 + settings.json | https://code.claude.com/docs/en/env-vars |
| Codex CLI | config.toml + codex login | https://github.com/openai/codex |
| Gemini CLI | ~/.gemini/.env | https://github.com/google-gemini/gemini-cli |
| OpenCode | opencode.json provider | https://opencode.ai/docs/providers/ |
| OpenClaw | openclaw.json models.providers | https://docs.openclaw.ai |
| Hermes Agent | config.yaml custom_providers | https://hermes-agent.nousresearch.com/docs |

## 使用方法

安装此插件后，在 AI 编程工具中触发 `setup` skill，按提示输入 API Key 即可完成配置。

## 服务信息

- 主线路: `https://api.aicodewith.com`
- 备用线路: `https://api.with7.cn`
- API Key 管理: https://api.aicodewith.com

## 安装

```bash
# Claude Code
claude mcp add --transport http https://github.com/DaneelOlivaw1/aicodewith-setup

# 或手动安装
git clone https://github.com/DaneelOlivaw1/aicodewith-setup
```
