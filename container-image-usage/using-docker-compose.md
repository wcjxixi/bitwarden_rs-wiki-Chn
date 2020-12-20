# 4.使用 Docker Compose

{% hint style="success" %}
对应的[页面地址](https://github.com/dani-garcia/bitwarden_rs/wiki/Using-Docker-Compose)
{% endhint %}

Docker Compose 是一个用于定义和配置多容器应用程序的工具。在我们的例子中，我们希望 Bitwarden\_rs 服务器和代理都将 WebSocket 请求重定向到正确的地方。

本指南基于 [\#126 \(comment\)](https://github.com/dani-garcia/bitwarden_rs/issues/126#issuecomment-417872681)。[这里](https://github.com/sosandroid/docker-bitwarden_rs-caddy-synology)也有另一种基于 Bitwarden\_rs 和 Caddy 2.0 的解决方案。

基于以下内容创建 `docker-compose.yml` 文件：

```python
# docker-compose.yml
version: '3'

services:
  bitwarden:
    image: bitwardenrs/server
    restart: always
    volumes:
      - ./bw-data:/data
    environment:
      WEBSOCKET_ENABLED: 'true' # 请求使用 websockets
      SIGNUPS_ALLOWED: 'true'   # 设置为 false 表示禁用注册

  caddy:
    image: abiosoft/caddy
    restart: always
    volumes:
      - ./Caddyfile:/etc/Caddyfile:ro
      - caddycerts:/root/.caddy
    ports:
      - 80:80 # Let's Encrypt 需要使用此端口
      - 443:443
    environment:
      ACME_AGREE: 'true'              # 同意 Let's Encrypt 用户协议
      DOMAIN: 'bitwarden.example.org' # 修改为您自己的! 用于自动生成 Let's Encrypt SSL
      EMAIL: 'bitwarden@example.org'  # 修改为您自己的! 可选。提供给 Let's Encrypt

volumes:
  caddycerts:
```

并创建相应的 `Caddyfile` 文件（不需要修改）：

```python
# Caddyfile
{$DOMAIN} {
    tls {$EMAIL}
    gzip

    header / {
        # 启用 HTTP Strict Transport Security (HSTS)
        Strict-Transport-Security "max-age=31536000;"
        # 启用 cross-site filter (XSS) 并告诉浏览器阻止检测到的攻击
        X-XSS-Protection "1; mode=block"
        # 禁止在框架内渲染站点 (clickjacking protection)
        X-Frame-Options "DENY"
        # 防止搜索引擎收录 (可选)
        #X-Robots-Tag "none"
    }

    # 协商端点也被代理到 Rocket
    proxy /notifications/hub/negotiate bitwarden:80 {
        transparent
    }

    # Notifications 重定向到 websockets 服务器
    proxy /notifications/hub bitwarden:3012 {
        websocket
    }

    # 将 root 目录代理到 Rocket
    proxy / bitwarden:80 {
        transparent
    }
}
```

运行以下命令创建并启动容器。它为反向代理在两个容器之间创建私有网络，这样就只有 caddy 暴露在外面了。

```python
docker-compose up -d
```

停止并销毁容器。

```python
docker-compose down
```

如果不需要 WebSocket 通知，则可以单独运行 Bitwarden\_rs。如果你和我一样，在 Raspberry Pi 上使用 bitwardenrs/server:raspberry 镜像运行 Bitwarden\_rs，请按我的示例操作。这是我的示例：

```python
# docker-compose.yml
version: '3'

services:
 bitwarden:
  image: bitwardenrs/server
  restart: always
  volumes:
      - ./bw-data:/data
      - ./ssl:/ssl
  ports:
    - 443:80
  environment:
   ROCKET_TLS: '{certs = "/ssl/fullchain.pem", key = "/ssl/key.pem"}'
   LOG_FILE: '/data/bitwarden.log'
   SIGNUPS_ALLOWED: 'true'
```

即使服务器运行在位于 NAT 后面的家庭网络上，我也想使用 Let's Encrypt 证书。我参考了这里的说明 [https://github.com/Neilpang/acme.sh/wiki/DNS-alias-mode](https://github.com/Neilpang/acme.sh/wiki/DNS-alias-mode)。

首先设置域名的 CNAME 记录，并通过 CloudFlare 导出 CF\_Key 和 CF\_Email 或  CF\_Token 和 CF\_Account\_ID。

然后颁发证书。参考 [https://github.com/Neilpang/acme.sh/wiki/dnsap](https://github.com/Neilpang/acme.sh/wiki/dnsapi)。

最后安装证书。

```python
acme.sh --install-cert -d home.example.com --key-file /home/pi/ssl/key.pem --fullchain-file /home/pi/ssl/fullchain.pem
```

或简单地使用

```python
acme.sh --issue -d home.example.com --challenge-alias otherdomain.com --dns dns_cf --key-file /home/pi/ssl/key.pem --fullchain-file /home/pi/ssl/fullchain.pem
```

我的域名的 A 记录指向 docker-compose.yml 文件最后一行绑定的 IP 地址，这样就不会出现证书相关的警告了。

