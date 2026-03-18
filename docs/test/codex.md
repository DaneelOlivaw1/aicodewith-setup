# Codex CLI 配置测试

## 前提

- Docker + 已登录 Docker Hub
- AICodeWith API Key
- 测试镜像 `aicodewith-test` 已构建（见 claude-code.md）

## 测试步骤

### 1. 启动容器

```bash
docker run -d --name codex-test aicodewith-test sleep 3600
```

### 2. 安装依赖

```bash
docker exec codex-test npm install -g @openai/codex
docker exec codex-test bash -c 'apt-get update -qq && apt-get install -y -qq git > /dev/null 2>&1'
```

### 3. 初始化 git 仓库（Codex 要求在 git 仓库内运行）

```bash
docker exec codex-test bash -c 'cd /tmp && git init test-repo'
```

### 4. 写入配置

```bash
docker exec codex-test mkdir -p /root/.codex
docker exec codex-test bash -c 'cat > /root/.codex/config.toml << EOF
model = "gpt-5.3-codex"
openai_base_url = "https://api.aicodewith.com/v1"
EOF'
```

### 5. 登录存储 API Key

```bash
docker exec codex-test bash -c 'echo "<你的KEY>" | codex login --with-api-key'
```

### 6. 测试连通性

```bash
docker exec codex-test bash -c 'cd /tmp/test-repo && codex exec "say hi"'
```

预期：返回正常文本响应（如 "hi"）。

### 7. 清理

```bash
docker stop codex-test && docker rm codex-test
```

## 关键发现

| 项目 | 说明 |
|------|------|
| 配置文件格式 | `config.toml`（非 yaml/json） |
| API 端点 | `openai_base_url`（非 `OPENAI_BASE_URL` 环境变量，后者已废弃） |
| 认证方式 | 必须通过 `codex login --with-api-key` 存储，`OPENAI_API_KEY` 环境变量不生效 |
| 运行要求 | 必须在 git 仓库目录内执行 |
| 非交互模式 | 使用 `codex exec` 子命令 |
