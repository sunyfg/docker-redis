# Redis Docker Compose（本地开发）

面向 macOS（Intel 与 Apple Silicon）本地开发的精简、生产意识型 Redis 7 环境。
通过 `.env` 统一配置：密码认证、AOF 持久化、健康检查与 Docker 命名卷。

## 环境要求

- Docker Desktop for Mac（Docker Engine + Compose v2）
- `redis-cli`（可选，用于宿主机直连；使用 `docker compose exec` 则无需安装）

## 快速开始

```bash
cp .env.example .env   # 仅首次需要，然后编辑 REDIS_PASSWORD
docker compose up -d
```

## 启动 Redis

```bash
docker compose up -d
```

## 查看状态

```bash
docker compose ps
```

Redis 应显示为 `healthy`（健康检查执行 `redis-cli -a "$REDIS_PASSWORD" ping`）。

## 查看日志

```bash
docker compose logs -f redis
```

## 连接 Redis

先从本地 `.env` 读取真实密码，然后在容器内连接：

```bash
docker compose exec redis sh
```

```bash
redis-cli -a "$REDIS_PASSWORD"
```

从宿主机连接（需要本地安装 `redis-cli`；切勿将真实密码写进 README —— 请查看你的 `.env`）：

```bash
redis-cli -h 127.0.0.1 -p 6379 -a '<REDIS_PASSWORD>'
```

## 测试连接

```redis
PING
```

预期输出：

```text
PONG
```

## 基本读写

```redis
SET demo:message "hello redis"
GET demo:message
```

## Redis 信息

```redis
INFO
```

## 持久化配置

```redis
CONFIG GET appendonly
CONFIG GET appendfsync
```

预期输出：`appendonly yes`、`appendfsync everysec`（AOF 已启用）。

## 停止 Redis

```bash
docker compose down
```

## 停止并删除 Redis 数据

> **警告：** 以下命令会删除本地全部 Redis 数据（移除 `redis_data` 卷）：

```bash
docker compose down -v
```

再次启动：

```bash
docker compose up -d
```

你将获得一个全新的 Redis 实例。

## 应用连接参数

| 参数     | 值                        |
|----------|---------------------------|
| host     | `127.0.0.1`               |
| port     | `6379`（见 `.env`）       |
| password | 见本地 `.env`             |
| database | `0`                       |

标准 Redis URL（从 `.env` 填入密码）：

```text
redis://:<PASSWORD>@127.0.0.1:6379/0
```

## 项目结构

```text
.
├── compose.yaml     # Redis 服务：认证、AOF、健康检查、命名卷
├── .env             # 本地密钥（git 已忽略）
├── .env.example     # .env 模板
├── .gitignore
└── README.md
```

## 说明

- 持久化：AOF（`appendonly yes`、`appendfsync everysec`）写入挂载在 `/data` 的 `redis_data` 命名卷。`docker compose down` 保留卷；`docker compose down -v` 删除卷。
- 密码通过 `REDIS_PASSWORD` 环境变量注入容器，并在命令中以 `$$REDIS_PASSWORD` 引用 —— 绝不在 `compose.yaml` 中硬编码。
- 未固定 `platform`：官方 `redis:latest` 镜像在 Intel 与 Apple Silicon Mac 上均可原生运行。