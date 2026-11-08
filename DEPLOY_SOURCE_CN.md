# Sub2API Linux + Docker 部署指南

> 本指南针对**本项目已做的二次开发**（重新设计的前端界面）编写。
> 关键点：不能直接用上游镜像 `weishaw/sub2api:latest`，必须用本仓库源码构建。

---

## 一、前置条件

| 项 | 要求 |
|---|---|
| 系统 | Linux x86_64 / arm64 |
| Docker | 20.10+ |
| Docker Compose | v2（`docker compose` 而非 `docker-compose`） |
| 内存 | ≥ 2 GB（构建镜像时建议 ≥ 4 GB） |
| 磁盘 | ≥ 10 GB |
| 网络 | 构建时需能访问 `goproxy.cn`（Dockerfile 已配好国内代理） |

安装 Docker（Debian/Ubuntu）：

```bash
curl -fsSL https://get.docker.com | sh
sudo systemctl enable --now docker
docker compose version   # 确认是 v2
```

---

## 二、获取源码

```bash
# 方式 A：git clone
git clone https://github.com/MeiGu1314/sub2api-2.git
cd sub2api-2

# 方式 B：上传压缩包后解压
# scp sub2api-2.zip root@your-server:/opt/
# cd /opt && unzip sub2api-2.zip && cd sub2api-2
```

---

## 三、配置环境变量

```bash
cd deploy
cp .env.example .env
```

编辑 `deploy/.env`，**至少填写以下 4 项**：

```ini
# 数据库密码（必填，compose 会强制校验）
POSTGRES_PASSWORD=换成你的强密码

# 管理员账号（留空则随机生成并打印在首次启动日志里）
ADMIN_EMAIL=you@example.com
ADMIN_PASSWORD=至少8位的强密码

# JWT 密钥（留空会导致每次重启后所有用户被登出）
# 生成：openssl rand -hex 32
JWT_SECRET=粘贴32字节hex

# TOTP 加密密钥（用于二次验证与支付恢复）
# 生成：openssl rand -hex 32
TOTP_ENCRYPTION_KEY=粘贴32字节hex
```

其他常用项（按需调整）：

```ini
SERVER_PORT=8080          # 对外端口
TZ=Asia/Shanghai          # 时区
BIND_HOST=0.0.0.0         # 监听地址（只走反代可改 127.0.0.1）
```

---

## 四、构建并启动（关键步骤）

**必须叠加 `docker-compose.source.yml`**，否则会拉取上游镜像（不含你的前端改动）：

```bash
cd deploy

# 首次构建 + 启动（约 5-15 分钟，取决于网络与机器性能）
docker compose -f docker-compose.yml -f docker-compose.source.yml up -d --build

# 查看构建/启动日志
docker compose -f docker-compose.yml -f docker-compose.source.yml logs -f sub2api
```

首次启动会自动执行数据库迁移、生成 `config.yaml`、创建管理员账号。

**请留意日志中的管理员凭据**（若 `.env` 里没填 `ADMIN_EMAIL` / `ADMIN_PASSWORD`）：

```bash
docker compose -f docker-compose.yml -f docker-compose.source.yml logs sub2api | grep -i "admin"
```

---

## 五、验证

```bash
# 容器状态（应有 sub2api / sub2api-postgres / sub2api-redis 三个 Up）
docker compose -f docker-compose.yml -f docker-compose.source.yml ps

# 健康检查
curl http://localhost:8080/health
# 期望：{"status":"ok"}
```

浏览器访问 `http://服务器IP:8080`，应看到**重新设计后的深色科技风首页**。
如果不是深色首页，说明用错了镜像（见第六节排查）。

---

## 六、验证前端改动是否生效

这是本次部署最容易踩的坑：

```bash
# 确认用的是本地构建的镜像，而不是 weishaw/sub2api
docker compose -f docker-compose.yml -f docker-compose.source.yml images

# 期望看到 sub2api-local:latest
# 如果看到 weishaw/sub2api:latest，说明没加 -f docker-compose.source.yml
```

页面上可识别的三个标志：

1. **首页**：深色底（`#05070d`）+ 荧光青配色 + 请求列表窗口，而非浅色卡片墙
2. **登录后仪表盘**：余额超大字号 + 今日焦点卡 + Token 构成条
3. **侧边栏**：浅灰蓝底，选中项为浅青底深青字

---

## 七、常用运维命令

```bash
# 定义别名简化长命令（加到 ~/.bashrc）
alias dc='docker compose -f docker-compose.yml -f docker-compose.source.yml'

# 查看日志
dc logs -f sub2api

# 重启
dc restart sub2api

# 停止（保留数据）
dc down

# 停止并删除数据卷（危险：会清空数据库）
dc down -v

# 更新代码后重新构建
git pull
dc up -d --build
```

---

## 八、数据备份

数据存放在 Docker 卷中：

| 卷名 | 内容 |
|---|---|
| `deploy_sub2api_data` | `config.yaml`、上传文件 |
| `deploy_sub2api_postgres_data` | 数据库 |
| `deploy_sub2api_redis_data` | Redis 持久化 |

手动备份数据库：

```bash
docker exec sub2api-postgres pg_dump -U sub2api sub2api \
  | gzip > backup_$(date +%Y%m%d).sql.gz
```

项目内置备份功能（管理后台 → 数据库备份）也支持 S3 兼容存储。

---

## 九、反向代理 + HTTPS（建议生产使用）

应用本身不处理 TLS，建议前置 Nginx 或 Caddy。

### Nginx 示例

```nginx
server {
    listen 443 ssl http2;
    server_name api.example.com;

    ssl_certificate     /etc/letsencrypt/live/api.example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/api.example.com/privkey.pem;

    # 关键：允许带下划线的请求头，否则粘性会话（session_id）失效
    underscores_in_headers on;

    client_max_body_size 100m;

    location / {
        proxy_pass http://127.0.0.1:8080;
        proxy_http_version 1.1;
        proxy_set_header Host              $host;
        proxy_set_header X-Real-IP         $remote_addr;
        proxy_set_header X-Forwarded-For   $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # SSE / 流式响应必须关闭缓冲
        proxy_buffering off;
        proxy_cache off;
        proxy_read_timeout 3600s;
        proxy_send_timeout 3600s;
    }
}
```

> `proxy_buffering off` 对流式输出是必须的，否则 AI 回复会整段憋到最后才返回。

用 Caddy 更简单（自动申请证书）：

```
api.example.com {
    reverse_proxy 127.0.0.1:8080
}
```

项目也提供了 `deploy/Caddyfile` 可参考。

---

## 十、故障排查

| 现象 | 原因 | 处理 |
|---|---|---|
| 页面还是旧样式 | 用了上游镜像 | 加 `-f docker-compose.source.yml --build` |
| `POSTGRES_PASSWORD is required` | `.env` 没填密码 | 填 `POSTGRES_PASSWORD` |
| 构建卡住/超时 | 拉不到 Go 依赖 | 检查服务器能否访问 `goproxy.cn` |
| 启动后立即退出 | 数据库连不上 | `dc logs postgres` 看数据库是否就绪 |
| 登录后 401 | `JWT_SECRET` 变了 | 固定 `.env` 里的 `JWT_SECRET` |
| 流式回复很慢 | 反代开了 buffering | Nginx 加 `proxy_buffering off` |
| 粘性会话不生效 | Nginx 丢弃下划线请求头 | 加 `underscores_in_headers on` |

查看容器内日志文件：

```bash
docker exec sub2api ls -la /app/data/logs/
docker exec sub2api tail -100 /app/data/logs/sub2api.log
```

---

## 十一、关于「检查更新」功能

本项目 `backend/internal/service/update_service.go` 里的更新源已改为：

```go
githubRepo = "MeiGu1314/sub2api-2"
```

**该仓库目前没有任何 Release**，所以后台点「检查更新」会返回
`warning: "GitHub API returned 404"`。

要启用自动更新，需要在仓库发布 Release，且资产命名必须满足：

| 要求 | 值 |
|---|---|
| tag | `v0.2.20` 形式（需大于当前版本才提示更新） |
| 压缩包名 | 必须含 `{os}_{arch}`，如 `sub2api_linux_amd64.tar.gz` |
| 校验文件 | 必须叫 `checksums.txt` |
| 下载域名 | 仅允许 `github.com` / `objects.githubusercontent.com` |

仓库根目录已有 `.goreleaser.yaml` 和 `.github/workflows/release.yml`，
打 tag 推上去即可自动产出符合要求的资产：

```bash
git tag v0.2.20
git push origin v0.2.20
```

> Docker 部署下，原地更新会在容器内替换二进制，重启容器后还原。
> **Docker 场景建议用 `docker compose up -d --build` 更新，而非用后台自更新。**
