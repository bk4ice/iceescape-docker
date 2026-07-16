# IceEscape Docker 快速启动

通过 Docker Compose 快速部署 IceEscape 摄影点位管理平台。

本项目使用预构建的 Docker 镜像，无需源代码即可启动。

## 环境要求

- Docker 20.10+
- Docker Compose 2.0+
- 至少 2GB 可用内存
- 至少 5GB 可用磁盘空间

## 快速开始

### 1. 克隆本仓库

```bash
git clone https://github.com/bk4ice/iceescape-docker.git
cd iceescape-docker
```

### 2. 配置环境变量

```bash
# 复制环境变量模板
cp .env.example .env

# 编辑 .env 文件，必须配置以下项：
# - POSTGRES_PASSWORD  (数据库密码，请设置强密码)
# - ADMIN_PASSWORD     (管理员密码，请设置强密码)
# - SECRET_KEY         (JWT 密钥，使用 openssl rand -hex 32 生成)
# - AMAP_KEY           (高德地图 API Key，申请地址: https://lbs.amap.com/)
# - MODELS__PROVIDERS__ALIYUN__API_KEY (阿里云 DashScope API Key，申请地址: https://dashscope.aliyun.com/)
#
# 可选配置（按需开启）：
# - QWEATHER_SUB / QWEATHER_KID  (和风天气凭证，需配合私钥文件)
# - DOMAIN / SSL_CERT_PATH      (生产环境 HTTPS 域名和证书)
# - MAP_PROFILE                 (地图提供商: amap 或 osm)
```

### 3. 准备 Secrets 目录（可选）

如果需要使用和风天气功能，请将和风天气的私钥文件放入 `secrets/` 目录：

```bash
cp /path/to/your-private-key.pem ./secrets/qweather_private_key.pem
```

详见 [secrets/README.md](secrets/README.md)。

### 4. 启动服务

**本地部署（HTTP，推荐快速体验）：**

无需域名和 SSL 证书，直接启动：

```bash
docker compose up -d
```

**生产环境（HTTPS，可选）：**

如果需要对外公开访问，通过 Nginx 终端 SSL，后端端口不对外暴露。部署前需准备 SSL 证书和域名。

```bash
# 1. 将 SSL 证书放入 ssl/ 目录
mkdir -p ssl
cp /path/to/your/cert.pem ssl/cert.pem
cp /path/to/your/cert.key ssl/cert.key

# 2. 确认 .env 中已配置域名和证书路径
# DOMAIN=your-domain.com
# SSL_CERT_PATH=./ssl/cert.pem
# SSL_KEY_PATH=./ssl/cert.key

# 3. 启动生产环境
docker compose -f docker-compose.prod.yml up -d
```

### 5. 访问应用

- **本地部署：**
  - 前端: http://localhost:3000
  - 后端 API: http://localhost:8000
  - API 文档: http://localhost:8000/docs
- **生产环境（HTTPS）：**
  - 前端: https://your-domain.com
  - 后端 API: https://your-domain.com/api/v1（通过 Nginx 反代，端口 8000 不对外暴露）
  - API 文档: 生产环境已关闭（DEBUG=false）

### 6. 管理后台登录

访问 `/admin` 路径进入管理后台，使用 `.env` 中配置的管理员账户登录：

- 用户名：`.env` 中的 `ADMIN_USERNAME`（默认 `admin`）
- 密码：`.env` 中的 `ADMIN_PASSWORD`

**注意：** 如果 `.env` 中未设置 `ADMIN_PASSWORD`，管理员登录将被禁用。

## 目录结构

```
docker-quickstart/
├── .env.example              # 环境变量模板
├── .gitignore
├── docker-compose.yml        # 开发环境（HTTP）
├── docker-compose.prod.yml   # 生产环境（HTTPS）
├── nginx.prod.conf           # 生产环境 Nginx 配置模板
├── README.md
├── secrets/                  # 敏感文件目录（和风天气私钥等）
│   └── README.md
├── ssl/                      # SSL 证书目录（生产环境使用）
│   └── README.md
├── uploads/                  # 上传文件（运行时自动创建）
├── saved_spots/              # 收藏数据（运行时自动创建）
└── cache/                    # 缓存数据（运行时自动创建）
```

## 常用命令

### 查看服务状态

```bash
docker compose ps
```

### 查看日志

```bash
# 查看所有服务日志
docker compose logs -f

# 查看特定服务日志
docker compose logs -f backend
docker compose logs -f frontend
docker compose logs -f db
```

### 重启服务

```bash
docker compose restart
docker compose restart backend
```

### 停止服务

```bash
# 停止所有服务
docker compose down

# 停止并删除数据卷（慎用，会删除所有数据）
docker compose down -v
```

### 更新服务

```bash
# 拉取最新镜像
docker compose pull

# 重新启动
docker compose up -d
```

## 数据持久化

项目使用 Docker volumes 持久化数据：

- **数据库数据**：`postgres_data` 命名卷
- **上传文件**：bind mount 到宿主机 `./uploads` 目录
- **收藏数据**：bind mount 到宿主机 `./saved_spots` 目录
- **缓存数据**：bind mount 到宿主机 `./cache` 目录

容器重启后数据不会丢失。如需备份数据，请参考以下命令：

### 数据库备份

```bash
# 备份 PostgreSQL 数据库
docker compose exec db pg_dump -U iceescape iceescape > backup.sql

# 恢复数据库
docker compose exec -T db psql -U iceescape iceescape < backup.sql
```

### 文件备份

```bash
# 备份上传文件
tar -czf uploads-backup.tar.gz uploads/

# 备份收藏数据
tar -czf saved-spots-backup.tar.gz saved_spots/
```

## 环境变量说明

### 必须配置的变量

| 变量名 | 说明 | 默认值 | 备注 |
|--------|------|--------|------|
| `POSTGRES_PASSWORD` | PostgreSQL 密码 | 无 | **必填**，请设置强密码 |
| `ADMIN_PASSWORD` | 管理员密码 | 无 | **必填**，不设置则禁用管理员登录 |
| `SECRET_KEY` | JWT 签名密钥 | 自动生成 | 生产环境建议显式设置 |
| `AMAP_KEY` | 高德地图 API Key | 无 | 地图和地理编码功能必需 |
| `MODELS__PROVIDERS__ALIYUN__API_KEY` | 阿里云 DashScope API Key | 无 | AI 智能体功能必需 |

### 生产环境变量

| 变量名 | 说明 | 默认值 |
|--------|------|--------|
| `DOMAIN` | 域名 | your-domain.com |
| `SSL_CERT_PATH` | SSL 证书路径 | ./ssl/cert.pem |
| `SSL_KEY_PATH` | SSL 私钥路径 | ./ssl/cert.key |
| `TRUST_PROXY_HEADERS` | 信任反代头 | true |
| `SECRETS_DIR` | Secrets 目录路径 | ./secrets |

### 可选配置

| 变量名 | 说明 | 默认值 |
|--------|------|--------|
| `QWEATHER_SUB` | 和风天气订阅 ID | 无 |
| `QWEATHER_KID` | 和风天气 KID | 无 |
| `QWEATHER_PRIVATE_KEY_PATH` | 和风天气私钥路径 | /app/secrets/qweather_private_key.pem |
| `MAP_PROFILE` | 地图提供商 (`amap` 或 `osm`) | amap |
| `RATE_LIMIT_ENABLED` | 启用限流 | true |
| `SECURITY_HEADERS_ENABLED` | 启用安全响应头 | true |

完整配置项请参考 `.env.example`。

## 服务架构

```
用户浏览器 ──→ Nginx (frontend) ──→ 后端 API (backend)
                    │                      │
                    │                      ├──→ PostgreSQL (db)
                    │                      └──→ 爬虫服务 (data-api)
                    │
                    └──→ 静态文件 (React SPA)
```

| 服务 | 镜像 | 端口(本地) | 说明 |
|------|------|-----------|------|
| frontend | `bllxk/iceescape-frontend:latest` | 3000 | Nginx + React SPA |
| backend | `bllxk/iceescape-backend:latest` | 8000 | FastAPI 后端 |
| data-api | `bllxk/mediacrawler-api:latest` | 8001 | 爬虫服务 |
| db | `postgres:15-alpine` | 5432 | PostgreSQL 数据库 |

## 故障排查

### 服务无法启动

1. 检查 Docker 是否正常运行：`docker ps`
2. 检查端口是否被占用：`netstat -tuln | grep -E '3000|8000|5432'`
3. 查看服务日志：`docker compose logs`

### 数据库连接失败

1. 检查数据库服务状态：`docker compose ps db`
2. 检查环境变量配置：`docker compose config`
3. 查看数据库日志：`docker compose logs db`

### 前端无法访问后端

1. 检查后端是否启动：`curl http://localhost:8000/health`
2. 开发环境确认 `VITE_API_BASE_URL` 指向 `http://localhost:8000`
3. 生产环境 `VITE_API_BASE_URL` 应为空（通过 Nginx 反代）
4. 检查 CORS 配置：`FRONTEND_URL` 需与实际访问地址一致

### 管理员无法登录

1. 确认 `.env` 中 `ADMIN_PASSWORD` 已设置且非空
2. 查看后端日志确认启动状态：`docker compose logs backend`

## 安全建议

1. **永远不要**将 `.env` 文件提交到版本控制
2. **永远不要**将 `ssl/` 或 `secrets/` 目录中的文件提交到版本控制
3. **定期更新** Docker 镜像：`docker compose pull && docker compose up -d`
4. **使用强密码**（至少 16 位，包含大小写字母、数字和特殊字符）
5. **限制网络访问**：生产环境使用防火墙，仅开放 80/443 端口
6. **定期备份**：自动化数据库和文件备份
7. **使用 HTTPS**：生产环境已内置 SSL 支持，将证书放入 `ssl/` 目录并通过 `docker-compose.prod.yml` 启动即可

## 许可证

MIT License
