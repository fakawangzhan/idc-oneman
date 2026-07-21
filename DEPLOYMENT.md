# IDC-ONEMAN 部署说明 

演示DEMO:https://p02--vps--mgsq65kksm7q.code.run/

## 1. 服务器要求

- Debian 11+/Ubuntu 20.04+，或 CentOS Stream/RHEL 8+
- 1 核 CPU、1 GB 内存、10 GB 可用磁盘起步
- 已开放 TCP 9080；生产环境建议由 Nginx/Caddy 反代并启用 HTTPS
- CLICD API Key / HashPay 商户

## 2. 一键安装

```bash
curl -fsSL https://raw.githubusercontent.com/fakawangzhan/idc-oneman/main/install.sh | sudo sh
```

中国大陆网络不稳定时：

```bash
curl -fsSL https://ghfast.top/https://raw.githubusercontent.com/fakawangzhan/idc-oneman/main/install.sh | sudo GITHUB_PROXY=https://ghfast.top USE_CN_MIRROR=1 sh
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
5. HashPay 回调地址为 `https://你的域名/hashpay/callback`。

## 4. 运维命令

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

## 5. 流程

1. 后台 CLICD 切小鸡。
2. 后台 SMTP 邮件发送小鸡关键信息。
3. 使用 HashPay 订单完成支付。
4. IDC-ONEMAN 添加销售套餐 管理用户。


感谢开源代码 CLICD / HashPay / EdgeKey,大佬们有实力的请随意进行深度二开，JUST DO IT!人人皆是OneMan！
