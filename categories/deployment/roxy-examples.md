# 代理示例

{% hint style="success" %}
对应的[页面地址](https://github.com/dani-garcia/bitwarden_rs/wiki/Proxy-examples)
{% endhint %}

在本文档中，`<SERVER>`指用于访问bitwarden\_rs的IP或域名，如果proxy和bitwarden\_rs都在同一系统中运行，简单地使用`localhost`即可。默认情况下，代理端口`80`用于Web服务器，`3012`用于WebSocket服务器。建议将代理配置为侦听启用了HTTPS的`443`端口。

使用代理时，最好在代理级别而不是在应用程序级别配置HTTPS，这样也可以保护WebSockets连接。

### 目录

* [Caddy 1.x](roxy-examples.md#caddy-1-x)
* [Caddy 2.x](roxy-examples.md#caddy-2-x)
* [Nginx](roxy-examples.md#nginx-by-shauder) \(by shauder\)
* [Nginx](roxy-examples.md#nginx-by-ypid) \(by ypid\)
* [Apache](roxy-examples.md#apache-by-fbartels) \(by fbartels\)
* [Apache in a sub-location](roxy-examples.md#apache-in-a-sub-location-by-ss-89) \(by ss89\)
* [Traefik v1](roxy-examples.md#traefik-v1-docker-compose-example) \(docker-compose example\)
* [Traefik v2](roxy-examples.md#traefik-v2-docker-compose-example-by-hwwilliams) \(docker-compose example by hwwilliams\)

### Caddy 1.x

Caddy在某些情况下可以自动启用HTTPS，参考[此文档](https://caddyserver.com/v1/docs/automatic-https)。

```php
:443 {
  tls ${SSLCERTIFICATE} ${SSLKEY}
  # or 'tls self_signed' to generate a self-signed certificate
  gzip

  # The negotiation endpoint is also proxied to Rocket
  proxy /notifications/hub/negotiate <SERVER>:80 {
    transparent
  }

  # Notifications redirected to the websockets server
  proxy /notifications/hub <SERVER>:3012 {
    websocket
  }

  # Proxy the Root directory to Rocket
  proxy / <SERVER>:80 {
    transparent
  }
}
```

### Caddy 2.x

同样，Caddy 2在某些情况下也可以自动启用HTTPS，参考[此文档](https://caddyserver.com/docs/automatic-https)。

```php
# Caddyfile V2.0 config file
:80 {
  #Caddy on port 80 in container to bitwarden_rs private instance
  #Use it if Caddy behind another reverse proxy such as the one embedded on Synology
  log {
	output file {env.LOG_FILE}
	level INFO
	#roll_size 5MiB #Not working on Caddy V2.0.0 Beta20 https://caddyserver.com/docs/caddyfile/directives/log#log
	#roll_keep 2 #Not working on Caddy V2.0.0 Beta20 https://caddyserver.com/docs/caddyfile/directives/log#log
  }
  encode gzip

  header / {
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
  reverse_proxy /notifications/hub/negotiate <SERVER>:80

  # Notifications redirected to the websockets server
  reverse_proxy /notifications/hub <SERVER>:3012

  # Proxy the Root directory to Rocket
  reverse_proxy <SERVER>:80
}

#{env.DOMAIN}:443 {
#  #Caddy on port 443 in container to bitwarden_rs private instance 
#  #Use it if Caddy exposed to the net 
#
#  log {
#	output file {env.LOG_FILE}
#	level INFO
#   #roll_size 5MiB #Not working on Caddy V2.0.0 Beta20 https://caddyserver.com/docs/caddyfile/directives/log#log
#   #rool_keep 30 #Not working on Caddy V2.0.0 Beta20 https://caddyserver.com/docs/caddyfile/directives/log#log
#  }
#
#  # Uncomment only one of the 2 lines. Depending if you provide your own cert or request one from Let's Encrypt
#  tls {env.SSLCERTIFICATE} {env.SSLKEY}
#  tls {env.EMAIL}
#
#  encode gzip
#
#  header / {
#       # Enable HTTP Strict Transport Security (HSTS)
#       Strict-Transport-Security "max-age=31536000;"
#       # Enable cross-site filter (XSS) and tell browser to block detected attacks
#       X-XSS-Protection "1; mode=block"
#       # Disallow the site to be rendered within a frame (clickjacking protection)
#       X-Frame-Options "DENY"
#       # Prevent search engines from indexing (optional)
#       X-Robots-Tag "none"
#       # Server name removing
#       -Server
#   }
#  # The negotiation endpoint is also proxied to Rocket
#  reverse_proxy /notifications/hub/negotiate <SERVER>:80
#
#  # Notifications redirected to the websockets server
#  reverse_proxy /notifications/hub <SERVER>:3012
#
#  # Proxy the Root directory to Rocket
#  reverse_proxy <SERVER>:80
#}
```

### Nginx \(by shauder\)

```php
server {
  listen 443 ssl http2;
  server_name vault.*;
  
  # Specify SSL config if using a shared one.
  #include conf.d/ssl/ssl.conf;
  
  # Allow large attachments
  client_max_body_size 128M;

  location / {
    proxy_pass http://<SERVER>:80;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
  }
  
  location /notifications/hub {
    proxy_pass http://<SERVER>:3012;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
  }
  
  location /notifications/hub/negotiate {
    proxy_pass http://<SERVER>:80;
  }

  # Optionally add extra authentication besides the AUTH_TOKEN
  # If you don't want this, leave this part out
  location /admin {
    # See: https://docs.nginx.com/nginx/admin-guide/security-controls/configuring-http-basic-authentication/
    auth_basic "Private";
    auth_basic_user_file /path/to/htpasswd_file;

    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;

    proxy_pass http://<SERVER>:80;
  }

}
```

### Nginx \(by ypid\)

使用DebOps配置nginx作为bitwarden\_rs的反向代理的清单示例。 我选择在URL中使用PSK以获得额外的安全性，从而不会将API公开给Internet上的每个人，因为客户端应用程序尚不支持客户端证书（我对其进行了测试）。 注意：使用subpath/PSK需要修补源代码并重新编译，请参考：[https：//github.com/dani-garcia/bitwarden\_rs/issues/241\#issuecomment-436376497](https://github.com/dani-garcia/bitwarden_rs/issues/241#issuecomment-436376497)。 /admin未经测试。 有关安全性子路径托管的一般讨论，请参阅：[https://github.com/debops/debops/issues/1233](https://github.com/debops/debops/issues/1233)

```php
bitwarden__fqdn: 'vault.example.org'

nginx__upstreams:

  - name: 'bitwarden'
    type: 'default'
    enabled: True
    server: 'localhost:8000'

nginx__servers:

  - name: '{{ bitwarden__fqdn }}'
    filename: 'debops.bitwarden'
    by_role: 'debops.bitwarden'
    favicon: False
    root: '/usr/share/bitwarden_rs/web-vault'

    location_list:

      - pattern: '/'
        options: |-
          deny all;

      - pattern: '= /ekkP9wtJ_psk_changeme_Hr9CCTud'
        options: |-
          return 307 $scheme://$host$request_uri/;

      ## All the security HTTP headers would then need to be set by nginx as well.
      # - pattern: '/ekkP9wtJ_psk_changeme_Hr9CCTud/'
      #   options: |-
      #     alias /usr/share/bitwarden_rs/web-vault/;

      - pattern: '/ekkP9wtJ_psk_changeme_Hr9CCTud/'
        options: |-
          proxy_set_header Host              $host;
          # proxy_set_header X-Real-IP         $remote_addr;
          # proxy_set_header X-Forwarded-For   $proxy_add_x_forwarded_for;
          proxy_set_header X-Forwarded-Proto $scheme;
          proxy_set_header X-Forwarded-Port  443;

          proxy_pass http://bitwarden;

      ## Do not use the icons features as long as it reveals all domains from
      ## our credentials to the server.
      - pattern: '/ekkP9wtJ_psk_changeme_Hr9CCTud/icons/'
        options: |-
          access_log off;
          log_not_found off;
          deny all;
```

### Apache \(by fbartels\)

```php
<VirtualHost *:443>
    SSLEngine on
    ServerName bitwarden.$hostname.$domainname

    SSLCertificateFile ${SSLCERTIFICATE}
    SSLCertificateKeyFile ${SSLKEY}
    SSLCACertificateFile ${SSLCA}
    ${SSLCHAIN}

    ErrorLog \${APACHE_LOG_DIR}/bitwarden-error.log
    CustomLog \${APACHE_LOG_DIR}/bitwarden-access.log combined

    RewriteEngine On
    RewriteCond %{HTTP:Upgrade} =websocket [NC]
    RewriteRule /notifications/hub(.*) ws://<SERVER>:3012/$1 [P,L]
    ProxyPass / http://<SERVER>:80/

    ProxyPreserveHost On
    ProxyRequests Off
    RequestHeader set X-Real-IP %{REMOTE_ADDR}s
</VirtualHost>
```

### Apache in a sub-location \(by ss89\)

确保在apache配置中的某个位置加载了websocket代理模块。 它看起来像这样：

```php
LoadModule proxy_wstunnel_module modules/mod_proxy_wstunnel.so`
```

```php
<VirtualHost *:443>
    SSLEngine on
    ServerName $hostname.$domainname

    SSLCertificateFile ${SSLCERTIFICATE}
    SSLCertificateKeyFile ${SSLKEY}
    SSLCACertificateFile ${SSLCA}
    ${SSLCHAIN}

    ErrorLog \${APACHE_LOG_DIR}/error.log
    CustomLog \${APACHE_LOG_DIR}/access.log combined

    <Location /bitwarden> #adjust here if necessary
        RewriteEngine On
        RewriteCond %{HTTP:Upgrade} =websocket [NC]
        RewriteRule /notifications/hub(.*) ws://<SERVER>:3012/$1 [P,L]
        ProxyPass http://<SERVER>:80/

        ProxyPreserveHost On
        RequestHeader set X-Real-IP %{REMOTE_ADDR}s
    </Location>
</VirtualHost>
```

### Traefik v1 \(docker-compose example\)

```php
labels:
    - traefik.enable=true
    - traefik.docker.network=traefik
    - traefik.web.frontend.rule=Host:bitwarden.domain.tld
    - traefik.web.port=80
    - traefik.hub.frontend.rule=Host:bitwarden.domain.tld;Path:/notifications/hub
    - traefik.hub.port=3012
    - traefik.hub.protocol=ws
```

### Traefik v2 \(docker-compose example by hwwilliams\)

#### 将Traefik v1标签迁移到Traefik v2

```php
labels:
  - traefik.enable=true
  - traefik.docker.network=traefik
  - traefik.http.routers.bitwarden-ui.rule=Host(`bitwarden.domain.tld`)
  - traefik.http.routers.bitwarden-ui.service=bitwarden-ui
  - traefik.http.services.bitwarden-ui.loadbalancer.server.port=80
  - traefik.http.routers.bitwarden-websocket.rule=Host(`bitwarden.domain.tld`) && Path(`/notifications/hub`)
  - traefik.http.routers.bitwarden-websocket.service=bitwarden-websocket
  - traefik.http.services.bitwarden-websocket.loadbalancer.server.port=3012
```

#### 迁移的标签加上HTTP到HTTPS重定向

这些标签假定Traefik中为端口80和443定义的入口点分别是“web”和“websecure”。

这些标签还假定您已经在Traefik中定义了默认的证书解析器。

```php
labels:
  - traefik.enable=true
  - traefik.docker.network=traefik
  - traefik.http.middlewares.redirect-https.redirectScheme.scheme=https
  - traefik.http.middlewares.redirect-https.redirectScheme.permanent=true
  - traefik.http.routers.bitwarden-ui-https.rule=Host(`bitwarden.domain.tld`)
  - traefik.http.routers.bitwarden-ui-https.entrypoints=websecure
  - traefik.http.routers.bitwarden-ui-https.tls=true
  - traefik.http.routers.bitwarden-ui-https.service=bitwarden-ui
  - traefik.http.routers.bitwarden-ui-http.rule=Host(`bitwarden.domain.tld`)
  - traefik.http.routers.bitwarden-ui-http.entrypoints=web
  - traefik.http.routers.bitwarden-ui-http.middlewares=redirect-https
  - traefik.http.routers.bitwarden-ui-http.service=bitwarden-ui
  - traefik.http.services.bitwarden-ui.loadbalancer.server.port=80
  - traefik.http.routers.bitwarden-websocket-https.rule=Host(`bitwarden.domain.tld`) && Path(`/notifications/hub`)
  - traefik.http.routers.bitwarden-websocket-https.entrypoints=websecure
  - traefik.http.routers.bitwarden-websocket-https.tls=true
  - traefik.http.routers.bitwarden-websocket-https.service=bitwarden-websocket
  - traefik.http.routers.bitwarden-websocket-http.rule=Host(`bitwarden.domain.tld`) && Path(`/notifications/hub`)
  - traefik.http.routers.bitwarden-websocket-http.entrypoints=web
  - traefik.http.routers.bitwarden-websocket-http.middlewares=redirect-https
  - traefik.http.routers.bitwarden-websocket-http.service=bitwarden-websocket
  - traefik.http.services.bitwarden-websocket.loadbalancer.server.port=3012
```

