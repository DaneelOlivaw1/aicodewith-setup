# AICodeWith Setup Plugin

一键配置 AI 编程工具连接 AICodeWith 中转服务。

## 支持的工具

| 工具 | 配置方式 |
|------|----------|
| Claude Code | 自动写入配置文件 |
| Codex CLI | 自动写入配置文件 |
| Gemini CLI | 自动写入配置文件 |
| OpenCode | 插件安装 + 认证 |
| OpenClaw | 插件安装 + 认证 |

## 安装方法

在 Claude Code 中执行：

```
/plugin marketplace add <你的GitHub用户名>/install_skills
/plugin install aicodewith-setup@<你的GitHub用户名>-install_skills
```

## 使用方法

安装后，直接对 Claude Code 说：

- "帮我配置 Claude Code 连接 AICodeWith"
- "配置 Gemini CLI"
- "把所有工具都配置好"
- `/aicodewith-setup:setup`

Claude 会自动引导你完成配置，只需提供你的 API Key。

## API Key 获取

登录 [AICodeWith](https://api.aicodewith.com) → 密钥管理 → 创建密钥

## 线路选择

| 线路 | 地址 | 说明 |
|------|------|------|
| 主线路 | `https://api.aicodewith.com` | 推荐，经 Cloudflare |
| 备用线路 | `https://api.with7.cn` | 国内直连 |

## 维护者说明

当模型列表变更时，只需更新 `models.json` 文件（或 API 端点 `https://api.aicodewith.com/models.json` 返回的内容）。Skill 逻辑无需修改。
