# VPS-ONE 部署说明

## 1. 服务器要求

- Debian 11+/Ubuntu 20.04+，或 CentOS Stream/RHEL 8+
- 1 核 CPU、1 GB 内存、10 GB 可用磁盘起步
- 已开放 TCP 9080；生产环境建议由 Nginx/Caddy 反代并启用 HTTPS
- CLICD API Key 具备容器读取、创建和查询权限

## 2. 一键安装

```bash
curl -fsSL https://raw.githubusercontent.com/fakawangzhan/vps-oneman-nb-p5/main/install.sh | sudo sh
```

中国大陆网络不稳定时：

```bash
curl -fsSL https://ghfast.top/https://raw.githubusercontent.com/fakawangzhan/vps-oneman-nb-p5/main/install.sh | sudo GITHUB_PROXY=https://ghfast.top USE_CN_MIRROR=1 sh
```

安装脚本自动识别 Debian/RHEL 系、安装 Docker、探测 GitHub、设置 PyPI 和 Docker Hub 国内镜像，并在 Git Clone 失败时回退到 tar.gz 源码包。离线或受限网络可将源码压缩包上传服务器，解压后在源码根目录执行 `sudo USE_CN_MIRROR=1 sh install.sh`。

可选变量：`INSTALL_DIR`、`VPS_ONE_PORT`、`BASE_URL`、`GITHUB_PROXY`、`USE_CN_MIRROR`、`PIP_INDEX_URL`、`DOCKER_REGISTRY`。

## 3. 初始化与后台配置

1. 打开 `http://服务器IP:9080/install`，创建首位管理员。
2. 进入“系统配置”：
   - 站点公开地址必须填写外部可访问的 HTTPS 地址。
   - CLICD 面板地址与 API Key。
   - HashPay 地址、商户 ID、私钥和平台公钥。
   - SMTP Host、Port、加密方式、账号、授权码与发件地址。
3. 分别点击 CLICD、HashPay、SMTP 测试按钮。
4. 在“套餐管理”配置 CLICD 节点、镜像、资源、网络和有效月份。
5. HashPay 回调地址为 `https://你的域名/payments/hashpay/callback`。

SMTP 常用配置：465 + `ssl`，或 587 + `starttls`。部分邮件服务必须使用 SMTP 授权码而非网页登录密码；发件地址通常必须与账号或服务端授权地址一致。

## 4. 本次关键修复

- CLICD `expires_at` 统一发送严格日期 `YYYY-MM-DD`，不再发送 `YYYY-MM-DD HH:MM:SS`。
- 容器创建前按订单资源名查询，降低支付回调重试造成重复创建的风险。
- 已有实例但缺少邮件任务时自动补建邮件任务。
- 邮件任务校验实例、订单、用户和套餐，邮件展示日期而非 Python datetime 文本。
- SMTP 支持 SSL、STARTTLS 和 plain，含地址、端口、认证、收件人拒绝及超时检查。
- 后台任务带指数退避，启动后自动恢复超时的 running 任务。
- SQLite 使用 WAL、busy timeout、关键组合索引与单写锁。该模式适合轻量单机，不应横向启动多个应用副本。

## 5. 运维命令

在安装目录执行：

```bash
docker compose ps
docker compose logs -f --tail=200
docker compose restart
docker compose pull && docker compose build --pull && docker compose up -d
docker compose exec vps-one python -c "import urllib.request; print(urllib.request.urlopen('http://127.0.0.1:9080/healthz').read())"
```

备份数据库卷：

```bash
docker compose stop
docker run --rm -v vps-oneman-nb-p5_vps_one_data:/data -v "$PWD":/backup alpine tar czf /backup/vps-one-data.tar.gz -C /data .
docker compose start
```

恢复前必须停止服务，解压备份到同一数据卷。请同时保存 `.env`；丢失 `MASTER_KEY` 后无法解密后台保存的 API/SMTP 密钥。

## 6. 验收流程

1. 后台 CLICD 测试连接成功。
2. 后台 SMTP 测试邮件到达测试邮箱。
3. 使用 HashPay 测试订单完成支付。
4. 后台任务中 `provision` 和 `mail_instance` 均为 done。
5. 客户中心显示容器信息，CLICD 中仅有一个对应订单资源。
6. 邮箱收到套餐、订单号、实例、IP、SSH 端口、初始密码、管理链接和到期日期。

若邮件未到：先检查后台任务错误，再检查垃圾箱、发件域 SPF/DKIM/DMARC 和邮件服务投递日志。若 CLICD 返回 400，任务错误会保存服务端的 `message/detail/error`，可在修正套餐后等待自动重试或重新触发履约。
