# 自建 RustDesk 服务器（Tronyi 远程桌面）

目标：给「纯内部/自用 + 公网远程协助」提供一台**自己的**信令 + 中继服务器，客户端连自己的服务器，
不经过 RustDesk 官方公共节点（更私密、更可控）。

架构只有两个进程：

| 进程 | 作用 | 端口 |
|---|---|---|
| `hbbs` | 信令 / ID 注册 / NAT 打洞（rendezvous） | TCP **21115**、TCP **21116**、UDP **21116**、TCP 21118（Web 客户端，可选） |
| `hbbr` | 中继（两端打洞失败时转发流量） | TCP **21117**、TCP 21119（Web 客户端，可选） |

> 最少需要：**TCP 21115、TCP 21116、UDP 21116、TCP 21117**。
> 不做浏览器网页访问的话，21118/21119 可以不开放。

---

## 0. 关键前置：阿里云安全组放行端口

本项目的服务器是阿里云 `8.137.193.93`，**当前安全组只放了 80/443/22/5667**，直接部署会连不上。
需要去 **阿里云控制台 → ECS → 安全组 → 入方向** 手动加规则（这一步只能在控制台做，脚本改不了）：

| 协议 | 端口范围 | 授权对象 |
|---|---|---|
| TCP | 21115 | 0.0.0.0/0 |
| TCP | 21116 | 0.0.0.0/0 |
| UDP | 21116 | 0.0.0.0/0 |
| TCP | 21117 | 0.0.0.0/0 |

（用网页客户端再加 TCP 21118、21119。）

---

## 方案 A：Docker 部署（推荐，服务器已装 docker 时）

```bash
cd deploy/server
cp .env.example .env          # 确认 RELAY_IP=8.137.193.93
docker compose up -d
docker compose logs -f        # 首次启动会生成密钥
```

取客户端要填的公钥（`hbbs` 首次运行生成在 `./data/id_ed25519.pub`）：

```bash
cat data/id_ed25519.pub
# 或： docker compose exec hbbs cat /root/id_ed25519.pub
```

---

## 方案 B：二进制 + systemd（服务器没有 docker 时）

1. 从 https://github.com/rustdesk/rustdesk-server/releases 下载 `rustdesk-server-linux-amd64.zip`，
   解出 `hbbs`、`hbbr`，放到 `/opt/rustdesk/`：

   ```bash
   sudo mkdir -p /opt/rustdesk && cd /opt/rustdesk
   sudo unzip ~/rustdesk-server-linux-amd64.zip
   sudo chmod +x hbbs hbbr
   ```

2. 安装本目录 `systemd/` 下的两个 unit：

   ```bash
   sudo cp systemd/hbbs.service systemd/hbbr.service /etc/systemd/system/
   sudo systemctl daemon-reload
   sudo systemctl enable --now hbbs hbbr
   ```

   首次启动后公钥在 `/opt/rustdesk/id_ed25519.pub`。

3. 查看状态 / 日志：

   ```bash
   sudo systemctl status hbbs hbbr
   sudo journalctl -u hbbs -n 50 --no-pager
   ```

---

## 客户端如何连

在客户端「设置 → 网络」里填：

- **ID 服务器**：`8.137.193.93`（或用域名）
- **中继服务器**：留空（默认走 ID 服务器即可），需要时填 `8.137.193.93:21117`
- **Key**：粘贴上面的 `id_ed25519.pub` 内容

然后被控端会得到一个 9 位 ID，主控端输入 ID + 密码即可远程。

---

## 验证

在服务器上确认端口在听：

```bash
ss -tulnp | grep -E '2111[5-9]'
```

自测连通（从外部机器）：客户端能显示「就绪 / Ready」，且能拿到 ID，即 hbbs 通了；
实际建立会话走通，即 hbbr 中继也通了。

> 和现有的 nginx（80/443）、webapp（5000-5003）、frp（7001/5667）都不冲突，端口独立。
