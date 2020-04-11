# 使用Docker Compose

{% hint style="success" %}
对应的[页面地址](https://github.com/dani-garcia/bitwarden_rs/wiki/Using-Docker-Compose)
{% endhint %}

Docker Compose是一个用于定义和配置多容器应用程序的工具。在我们的例子中，我们希望Bitwarden\_rs服务器和代理都将WebSocket请求重定向到正确的地方。

本指南基于[\#126 \(comment\)](https://github.com/dani-garcia/bitwarden_rs/issues/126#issuecomment-417872681)。[这里](https://github.com/sosandroid/docker-bitwarden_rs-caddy-synology)也有另一种基于Bitwarden\_RS和Caddy 2.0的解决方案

使用以下内容创建`docker-compose.yml`文件：

```yaml
# docker-compose.yml
version: '3'

services:
  bitwarden:
    image: bitwardenrs/server
    restart: always
    volumes:
      - ./bw-data:/data
    environment:
      WEBSOCKET_ENABLED: 'true' # 请求使用websockets
      SIGNUPS_ALLOWED: 'true'   # 设置为false表示禁用注册

  caddy:
    image: abiosoft/caddy
    restart: always
    volumes:
      - ./Caddyfile:/etc/Caddyfile:ro
      - caddycerts:/root/.caddy
    ports:
      - 80:80 # Let's Encrypt需要使用此端口
      - 443:443
    environment:
      ACME_AGREE: 'true'              # 同意Let's Encrypt用户协议
      DOMAIN: 'bitwarden.example.org' # 修改为您自己的! 用于自动生成Let's Encrypt SSL
      EMAIL: 'bitwarden@example.org'  # 修改为您自己的! 可选。提供给Let's Encrypt

volumes:
  caddycerts:
```

并创建相应的`Caddyfile`文件（不需要修改）：

```yaml
# Caddyfile
{$DOMAIN} {
    tls {$EMAIL}
    gzip

    header / {
        # 启用HTTP Strict Transport Security (HSTS)
        Strict-Transport-Security "max-age=31536000;"
        # 启用cross-site filter (XSS) 并告诉浏览器对检测到的攻击进行阻止
        X-XSS-Protection "1; mode=block"
        # 禁止在框架内渲染站点 (clickjacking protection)
        X-Frame-Options "DENY"
        # 阻止搜索引擎编制索引 (可选)
        #X-Robots-Tag "none"
    }

    # 将negotiation endpoint代理到Rocket
    proxy /notifications/hub/negotiate bitwarden:80 {
        transparent
    }

    # Notifications重定向到websockets服务器
    proxy /notifications/hub bitwarden:3012 {
        websocket
    }

    # 将root目录代理到Rocket
    proxy / bitwarden:80 {
        transparent
    }
}
```

运行以下命令创建并启动容器。它为反向代理在两个容器之间创建私有网络，这样就只有caddy暴露在外面了。

```text
docker-compose up -d
```

停止并销毁容器。

```text
docker-compose down
```

如果不需要WebSocket通知，则可以单独运行Bitwarden\_rs。如果你和我一样，在Raspberry Pi上使用bitwardenrs/server:raspberry镜像运行Bitwarden\_rs，请按我的示例操作。这是我的示例：

```yaml
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

即使服务器运行在位于NAT后面的家庭网络上，我也想使用Let's Encrypt证书。参考这里的说明[https://github.com/Neilpang/acme.sh/wiki/DNS-alias-mode](https://github.com/Neilpang/acme.sh/wiki/DNS-alias-mode)。

首先设置域名，并通过CloudFlare导出CF\_Key和CF\_Email或CF\_Token以及CF\_Account\_ID。

然后颁发证书。参考[https://github.com/Neilpang/acme.sh/wiki/dnsapi](https://github.com/Neilpang/acme.sh/wiki/dnsapi)

最后安装证书。

```text
acme.sh --install-cert -d home.example.com --key-file /home/pi/ssl/key.pem --fullchain-file /home/pi/ssl/fullchain.pem
```

或简单地使用

```text
acme.sh --issue -d home.example.com --challenge-alias otherdomain.com --dns dns_cf --key-file /home/pi/ssl/key.pem --fullchain-file /home/pi/ssl/fullchain.pem
```

我的域名的A记录指向docker-compose.yml文件最后一行的域名绑定的IP地址，这样就不会有关于证书的警告了。

