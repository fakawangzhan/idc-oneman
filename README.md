# IDC OneMan

基于 FastAPI、SQLite 与 Docker Compose 的轻量 VPS 自动销售和交付系统。

## 功能

- 用户注册、套餐购买与 HashPay 支付回调
- 支付成功后通过 CLICD 创建并自动启动唯一订单容器
- 实时容器状态、到期日、开机、关机和重启
- CLICD 子用户凭据加密保存，仅订单所属用户查看
- 产品凭据自动发送至用户注册邮箱
- SQLite WAL、持久化任务、失败重试和异常任务恢复
- 中国大陆网络镜像及 Git/tar.gz 下载回退

## 一键安装

```bash
curl -fsSL https://raw.githubusercontent.com/fakawangzhan/idc-oneman/main/install.sh | sudo sh
```

中国大陆网络：

```bash
curl -fsSL https://ghfast.top/https://raw.githubusercontent.com/fakawangzhan/idc-oneman/main/install.sh | sudo GITHUB_PROXY=https://ghfast.top USE_CN_MIRROR=1 sh
```

安装完成后访问 `http://服务器IP:9080/install`。完整配置、升级、备份、恢复和故障处理请查看 [DEPLOYMENT.md](./DEPLOYMENT.md)。

## 本地验证

```bash
cp .env.example .env
docker compose build --pull
docker compose up -d
docker compose ps
curl -fsS http://127.0.0.1:9080/healthz
```

运行测试：

```bash
python -m pytest -q
```

## 架构边界

系统保持 SQLite 单机架构并仅运行一个 Uvicorn worker，适合轻量高并发读取和可靠订单履约，不支持多节点横向扩展。生产环境必须启用 HTTPS、设置高强度 `SECRET_KEY` 与 `MASTER_KEY`，并妥善备份 `.env` 和数据卷。
