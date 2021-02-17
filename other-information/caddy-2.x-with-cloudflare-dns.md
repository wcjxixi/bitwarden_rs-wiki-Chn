# 8.使用 Cloudflare DNS 的 Caddy 2.x

{% hint style="success" %}
对应的[页面地址](https://github.com/dani-garcia/bitwarden_rs/wiki/Caddy-2.x-with-Cloudflare-DNS)
{% endhint %}

Dockerfile（Caddy 构建器）：

```text
FROM caddy:builder AS builder
RUN xcaddy build --with github.com/caddy-dns/cloudflare

FROM caddy:latest
COPY --from=builder /usr/bin/caddy /usr/bin/caddy
```

构建命令：

```text
docker build -t [YOUR-NAME]/caddycfdns .
```

Caddyfile（作为反向代理）：

```text
https://[YOUR-DOMAIN]:443 {

  tls {
        dns cloudflare [API-KEY]
  }

  encode gzip

  header / {
       # Enable HTTP Strict Transport Security (HSTS)
       Strict-Transport-Security "max-age=31536000;"
       # Enable cross-site filter (XSS) and tell browser to block detected attacks
       X-XSS-Protection "1; mode=block"
       # Disallow the site to be rendered within a frame (clickjacking protection)
       X-Frame-Options "DENY"
       # Prevent search engines from indexing (optional)
       X-Robots-Tag "none"
       # Server name removing
       -Server
   }
  # The negotiation endpoint is also proxied to Rocket
  reverse_proxy /notifications/hub/negotiate bitwarden:80

  # Notifications redirected to the websockets server
  reverse_proxy /notifications/hub bitwarden:3012

  # Proxy the Root directory to Rocket
  reverse_proxy bitwarden:80 {
       # Send the true remote IP to Rocket, so that bitwarden_rs can put this in the
       # log, so that fail2ban can ban the correct IP.
       header_up X-Real-IP {remote_host}
  }
}
```

docker-compose.yml：

```text
version: '3'

services:
  bitwarden:
    image: bitwardenrs/server
    restart: always
    volumes:
      - $PWD/bw-data:/data
    environment:
      WEBSOCKET_ENABLED: 'true' # Required to use websockets
      SIGNUPS_ALLOWED: 'false'   # set to false to disable signups
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

