# 运维手册（RUNBOOK）

## 日常操作

### 启动守护进程

```bash
uv run dong-core
```

默认监听 `127.0.0.1:7437`，按 `Ctrl+C` 优雅退出。

### 验证连通

```bash
uv run dong ping
# → pong server=0.0.1 uptime=12ms latency=2ms
```

### 停止守护进程

```bash
kill $(pgrep -f dong-core)
```

---

## 配置

优先级（低 → 高）：**内建默认值 → `~/.dong/config.toml` → `.env` → 系统环境变量**。

### `~/.dong/config.toml`

```toml
[core]
host = "127.0.0.1"
port = 7437

[logging]
level  = "INFO"
file   = "~/.dong/logs/core.log"
format = "text"    # "text" | "json"
```

### `.env`

从 `.env.example` 复制后修改，存放本机配置与密钥（不提交 git）：

```bash
cp .env.example .env
```

### 系统环境变量

| 变量 | 默认值 | 说明 |
|------|--------|------|
| `DONG_CONFIG` | `~/.dong/config.toml` | 覆盖配置文件路径 |
| `DONG_HOST` | `127.0.0.1` | TCP 监听地址 |
| `DONG_PORT` | `7437` | TCP 监听端口 |
| `DONG_LOG_LEVEL` | `INFO` | 日志级别（DEBUG / INFO / WARNING / ERROR） |
| `DONG_LOG_FILE` | `~/.dong/logs/core.log` | 日志文件路径（留空则仅输出 stderr） |
| `DONG_LOG_FORMAT` | `text` | 日志格式（`text` 或 `json`） |

---

## 开发

```bash
uv run ruff check src tests scripts   # lint
uv run mypy src                       # 类型检查
uv run pytest tests/ -v               # 全量测试
uv run pytest tests/unit/ -v         # 仅单元测试（无需启动 daemon）

make docs                             # 重新生成 WIRE_PROTOCOL.md
make verify-s0                        # 完整验证（lint + 类型 + 测试 + 协议同源检查）
```

---

## 日志

```bash
tail -f ~/.dong/logs/core.log
```

---

## 常见错误

| 报错 | 原因 | 处理 |
|------|------|------|
| `core already running at 127.0.0.1:7437` | 已有守护进程在运行 | `kill $(pgrep -f dong-core)` |
| `core not running` | 未启动守护进程 | `uv run dong-core` |
| `Address already in use` | 端口被其他进程占用 | `DONG_PORT=8000 uv run dong-core` |
| `Config error: DONG_PORT must be an integer` | `.env` 或环境变量中端口值非整数 | 检查 `DONG_PORT` 的值 |
