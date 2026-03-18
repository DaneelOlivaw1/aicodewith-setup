# OpenCode 配置测试

## 前提

- Docker + 已登录 Docker Hub
- AICodeWith API Key
- 测试镜像 `aicodewith-test` 已构建（见 claude-code.md）

## 测试步骤

### 1. 启动容器并安装

```bash
docker run -d --name opencode-test aicodewith-test sleep 3600
docker exec opencode-test npm install -g opencode-ai
```

### 2. 写入配置

按照 `skills/setup/SKILL.md` 中 **OpenCode** 部分的步骤，在容器内执行配置（通过 `docker exec` 写入 `opencode.json`）。

### 3. 测试连通性

```bash
docker exec opencode-test bash -c 'cd /tmp && opencode run -m "aicodewith/<模型名>" "say hi"'
```

预期：返回正常文本响应（如 "Hi"）。

### 4. 清理

```bash
docker stop opencode-test && docker rm opencode-test
```

## 关键发现

| 项目 | 说明 |
|------|------|
| npm 包 | 必须用 `@ai-sdk/openai`（Responses API），不能用 `@ai-sdk/openai-compatible`（Chat Completions 已废弃） |
| 配置路径 | `~/.config/opencode/opencode.json` |
| 模型指定 | `-m "provider/model"` 格式，如 `aicodewith/gpt-5.4` |
| 非交互模式 | `opencode run` 子命令 |
| 首次运行 | 会执行数据库迁移，需要额外等待时间 |
