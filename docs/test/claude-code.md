# Claude Code 配置测试

在 Docker 干净环境中验证 SKILL.md 的 Claude Code 配置方法。

## 前提

- Docker 已安装并启动
- 已登录 Docker Hub（避免拉取限流）
- 有可用的 AICodeWith API Key

## 测试镜像

`test/Dockerfile`:

```dockerfile
FROM node:22-slim
RUN npm install -g @anthropic-ai/claude-code
WORKDIR /root
```

构建：

```bash
docker build -t aicodewith-test ./test
```

## 测试步骤

### 1. 启动容器

```bash
docker run -d --name cc-test aicodewith-test sleep 3600
```

### 2. 写入配置（模拟 SKILL.md 步骤）

```bash
# 创建目录
docker exec cc-test mkdir -p /root/.claude

# 写入 settings.json
docker exec cc-test bash -c 'cat > /root/.claude/settings.json << EOF
{
  "env": {
    "ANTHROPIC_API_KEY": "<你的KEY>",
    "ANTHROPIC_BASE_URL": "https://api.aicodewith.com"
  }
}
EOF'

# 写入 .claude.json
docker exec cc-test bash -c 'cat > /root/.claude.json << EOF
{
  "hasCompletedOnboarding": true
}
EOF'
```

### 3. 验证配置文件

```bash
docker exec cc-test cat /root/.claude/settings.json
docker exec cc-test cat /root/.claude.json
```

### 4. 测试连通性

```bash
docker exec cc-test claude -p "say hi"
```

预期：返回正常文本响应（如 "Hi! What are you working on?"）。

### 5. 清理

```bash
docker stop cc-test && docker rm cc-test
```

## 备用线路测试

将 `ANTHROPIC_BASE_URL` 替换为 `https://api.with7.cn`，重复上述步骤验证备用线路。

## 常见问题

| 现象 | 原因 | 解决 |
|------|------|------|
| `docker build` 报 429 Too Many Requests | Docker Hub 未登录限流 | `docker login` |
| `claude -p` 超时 | 网络不通或 BASE_URL 错误 | 检查容器网络，换备用线路 |
| `claude -p` 报 401 | API Key 无效或过期 | 重新创建 Key |
