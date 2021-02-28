# 5.使用 Cloudflare DNS 的 Caddy 2.x

{% hint style="success" %}
对应的[页面地址](https://github.com/dani-garcia/bitwarden_rs/wiki/Caddy-2.x-with-Cloudflare-DNS)
{% endhint %}

Dockerfile（Caddy 构建器）：

```python
FROM caddy:builder AS builder
RUN xcaddy build --with github.com/caddy-dns/cloudflare

FROM caddy:latest
COPY --from=builder /usr/bin/caddy /usr/bin/caddy
```

构建命令：

```python
docker build -t [YOUR-NAME]/caddycfdns .
```

Caddyfile（作为反向代理）：

```python
https://[YOUR-DOMAIN]:443 {

  tls {
        dns cloudflare [API-KEY]
  }

  encode gzip

  header / {
       # 启用 HTTP Strict Transport Security (HSTS)
       Strict-Transport-Security "max-age=31536000;"
       # 启用 cross-site filter (XSS) 并告诉浏览器阻止检测到的攻击
       X-XSS-Protection "1; mode=block"
       # 禁止在框架内呈现网站（clickjacking protection）
       X-Frame-Options "DENY"
       # 防止搜索引擎编制索引（可选）
       X-Robots-Tag "none"
       # 服务器名称移除
       -Server
   }
  # 协商端点也代理到 Rocket
  reverse_proxy /notifications/hub/negotiate bitwarden:80

  # 通知重定向到 websockets 服务器
  reverse_proxy /notifications/hub bitwarden:3012

  # 将 Root 目录代理到 Rocket
  reverse_proxy bitwarden:80 {
       # 将真正的远程 IP 发送给 Rocket，以便 bitwarden_rs 可以将其
       # 放入日志，以便 fail2ban 可以禁止正确的 IP。
       header_up X-Real-IP {remote_host}
  }
}
```

docker-compose.yml：

```python
version: '3'

services:
  bitwarden:
    image: bitwardenrs/server
    restart: always
    volumes:
      - $PWD/bw-data:/data
    environment:
      WEBSOCKET_ENABLED: 'true' # 启用 websockets
      SIGNUPS_ALLOWED: 'false'   # 设置为 false 以禁用注册
      DOMAIN: 'https://[DOMAIN]'
      SMTP_HOST: '[MAIL-SERVER]'
      SMTP_FROM: '[E-MAIL]'
      SMTP_PORT: '587'
      SMTP_SSL: 'true'
      SMTP_USERNAME: '[E-MAIL]'
      SMTP_PASSWORD: '[SMTP-PASS]'
#      ADMIN_TOKEN: '[RAND. GENERATE]'
#      YUBICO_CLIENT_ID: '[OPTIONAL]'
#      YUBICO_SECRET_KEY: '[OPTIONAL]'

  caddy:
    image: [YOUR-NAME]/caddycfdns
    restart: always
    volumes:
      - $PWD/Caddyfile:/etc/caddy/Caddyfile
      - caddy_data:/data
      - caddy_config:/config
      - caddy_log:/logs
    ports:
      - [PRIVATE-IP]:443:443
    environment:
      ACME_AGREE: 'true'
      CLOUDFLARE_EMAIL: '[YOUR-EMAIL]'
      CLOUDFLARE_API_TOKEN: '[YOUR-TOKEN]'
      DOMAIN: '[DOMAIN]'

volumes:
  caddy_data:
  caddy_config:
  caddy_log:
```

