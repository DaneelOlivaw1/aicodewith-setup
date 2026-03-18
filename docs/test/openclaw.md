# OpenClaw 配置测试

## 前提

- Docker + 已登录 Docker Hub
- AICodeWith API Key
- 测试镜像 `aicodewith-test` 已构建（见 claude-code.md）

## 测试步骤

### 1. 启动容器并安装

```bash
docker run -d --name openclaw-test aicodewith-test sleep 3600
docker exec openclaw-test bash -c 'apt-get update -qq && apt-get install -y -qq git > /dev/null 2>&1'
docker exec openclaw-test npm install -g openclaw@latest
```

### 2. 写入配置

按照 `skills/setup/SKILL.md` 中 **OpenClaw** 部分的步骤，在容器内执行配置（通过 `docker exec` 写入 `~/.openclaw/openclaw.json`）。

### 3. 验证配置

```bash
docker exec openclaw-test openclaw config validate
```

预期：`Config valid: ~/.openclaw/openclaw.json`

### 4. 测试连通性

```bash
docker exec openclaw-test openclaw agent --local --to "+10000000000" -m "say hi" --json
```

预期：JSON 输出中 `payloads[0].text` 包含正常响应，`meta.agentMeta.provider` 为 `aicodewith`。

### 5. 清理

```bash
docker stop openclaw-test && docker rm openclaw-test
```

## 关键发现

| 项目 | 说明 |
|------|------|
| 配置路径 | `~/.openclaw/openclaw.json` |
| api 适配器 | 使用 `openai-responses`（非 `openai-completions`） |
| 安装依赖 | 需要 git，否则 npm install 会失败 |
| 非交互测试 | `openclaw agent --local --to <号码> -m <消息> --json` |
| 配置验证 | `openclaw config validate` 可预检配置格式 |
