# Gemini CLI 配置测试

## 前提

- Docker + 已登录 Docker Hub
- AICodeWith API Key
- 测试镜像 `aicodewith-test` 已构建（见 claude-code.md）

## 测试步骤

### 1. 启动容器并安装

```bash
docker run -d --name gemini-test aicodewith-test sleep 3600
docker exec gemini-test npm install -g @google/gemini-cli
```

### 2. 写入配置

按照 `skills/setup/SKILL.md` 中 **Gemini CLI** 部分的步骤，在容器内执行配置。

额外需要初始化 `projects.json`（Gemini CLI 首次运行需要）：

```bash
docker exec gemini-test bash -c 'echo "{\"projects\":{},\"nextId\":1}" > ~/.gemini/projects.json'
```

### 3. 测试连通性

```bash
docker exec gemini-test bash -c 'cd /tmp && gemini -p "say hi"'
```

预期：返回正常文本响应（如 "Hi."）。

### 4. 清理

```bash
docker stop gemini-test && docker rm gemini-test
```

## 关键发现

| 项目 | 说明 |
|------|------|
| projects.json | 首次运行前需初始化，否则报 ENOENT 错误 |
| 非交互模式 | 使用 `-p` 参数 |
| .env 位置 | `~/.gemini/.env`，Gemini CLI 自动加载 |
