# 4.使用 Docker Compose

{% hint style="success" %}
对应的[页面地址](https://github.com/dani-garcia/bitwarden_rs/wiki/Using-Docker-Compose)
{% endhint %}

[Docker Compose](https://docs.docker.com/compose/) 是一个用于定义和配置多容器应用程序的工具。在我们的例子中，我们希望 Bitwarden\_rs 服务器和代理都将 WebSocket 请求重定向到正确的地方。

本示例假定您已[安装](https://docs.docker.com/compose/install/) Docker Compose，并且您的 bitwarden\_rs 实例具有一个可以公开访问的域名（比如 `bitwarden.example.com`）。

首先创建一个新的目录并切换到该目录。接下来，创建如下的 `docker-compose.yml` 文件（注意将 `DOMAIN` 和 `EMAIL` 变量替换为相应的值）：

```python
version: '3'

services:
  bitwarden:
    image: bitwardenrs/server:latest
    container_name: bitwarden
    restart: always
    environment:
      - WEBSOCKET_ENABLED=true  # 启用 WebSocket 通知
    volumes:
      - ./bw-data:/data

  caddy:
    image: caddy:2
    container_name: caddy
    restart: always
    ports:
      - 80:80  #  ACME HTTP-01 验证需要
      - 443:443
    volumes:
      - ./Caddyfile:/etc/caddy/Caddyfile:ro
      - ./caddy-config:/config
      - ./caddy-data:/data
    environment:
      - DOMAIN=bitwarden.example.com  # 您的域名
      - EMAIL=admin@example.com       # 用于 ACME 注册的电子邮件地址
      - LOG_FILE=/data/access.log
```

相同的目录下创建如下的 `Caddyfile` 文件（此文件不需要修改）：

```python
{$DOMAIN}:443 {
  log {
    level INFO
    output file {$LOG_FILE} {
      roll_size 10MB
      roll_keep 10
    }
  }

  # 使用 ACME HTTP-01 验证方式为已配置的域名获取证书
  tls {$EMAIL}

  # 此设置可能会在某些浏览器上出现兼容性问题（例如，在 Firefox 上下载附件）
  # 如果遇到问题，请尝试禁用此功能
  encode gzip

  # Notifications 重定向到 WebSocket 服务器
  reverse_proxy /notifications/hub bitwarden:3012

  # 将任何其他东西代理到 Rocket
  reverse_proxy bitwarden:80 {
       # 把真实的远程 IP 发送给 Rocket，让 bitwarden_rs 把其放在日志中
       # 这样 fail2ban 就可以阻止正确的 IP 了
       header_up X-Real-IP {remote_host}
  }
}
```

运行以下命令创建并启动容器。它为反向代理在两个容器之间创建私有网络，这样就只有 caddy 暴露在外面了：

```python
docker-compose up -d
```

停止并销毁容器：

```python
docker-compose down
```

[此处](https://github.com/sosandroid/docker-bitwarden_rs-caddy-synology)提供了一个类似的适用于 Synology 的基于 Caddy 的示例。

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

最后安装证书：

```python
acme.sh --install-cert -d home.example.com --key-file /home/pi/ssl/key.pem --fullchain-file /home/pi/ssl/fullchain.pem
```

或简单地使用：

```python
acme.sh --issue -d home.example.com --challenge-alias otherdomain.com --dns dns_cf --key-file /home/pi/ssl/key.pem --fullchain-file /home/pi/ssl/fullchain.pem
```

我的域名的 A 记录指向 docker-compose.yml 文件最后一行绑定的 IP 地址，这样就不会出现证书相关的警告了。

