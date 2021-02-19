# 5.代理示例

{% hint style="success" %}
对应的[页面地址](https://github.com/dani-garcia/bitwarden_rs/wiki/Proxy-examples)
{% endhint %}

在此文档中，`<SERVER>` 指用于访问 bitwarden\_rs 的 IP 或域名，如果代理和 bitwarden\_rs 两者在同一系统中运行，简单地使用 `localhost` 即可。默认的代理端口 `80` 用于 Web 服务器，`3012` 用于 WebSocket 服务器。建议将代理配置为侦听启用了 HTTPS 的 `443` 端口。

使用代理时，最好在代理级别而不是在应用程序级别配置 HTTPS，这样也可以保护 WebSockets 连接。

## 目录 <a id="table-of-contents"></a>

* [Caddy 1.x](proxy-examples.md#caddy-1-x)
* [Caddy 2.x](proxy-examples.md#caddy-2-x)
* [Nginx](proxy-examples.md#nginx-by-shauder) \(by shauder\)
* [Nginx with sub-path](proxy-examples.md#nginx-with-sub-path-by-blackdex) \(by BlackDex\)
* [Nginx](proxy-examples.md#nginx-by-ypid) \(by ypid\)
* [Nginx](proxy-examples.md#nginx-nixos-by-tklitschi) \(NixOS\)\(by tklitschi\)
* [Apache](proxy-examples.md#apache-by-fbartels) \(by fbartels\)
* [Apache in a sub-location](proxy-examples.md#apache-in-a-sub-location-by-ss-89) \(by ss89\)
* [Traefik v1](proxy-examples.md#traefik-v1-dockercompose-shi-li) \(docker-compose 示例\)
* [Traefik v2](proxy-examples.md#traefik-v-2-docker-compose-example-by-hwwilliams) \(docker-compose 示例 by hwwilliams\)
* [HAproxy](proxy-examples.md#haproxy-by-blackdex) \(by BlackDex\)

## Caddy 1.x

Caddy 在某些情况下可以自动启用 HTTPS，参考[此文档](https://caddyserver.com/v1/docs/automatic-https)。

```python
:443 {
  tls ${SSLCERTIFICATE} ${SSLKEY}
  # 或使用 'tls self_signed' 生成自签名证书

  # 此设置可能会对某些浏览器产生兼容性问题
  # （例如，在Firefox上下载附件时）。
  # 如果遇到问题，请尝试禁用此功能。
  gzip

  # 协商端点也被代理到 Rocket
  proxy /notifications/hub/negotiate <SERVER>:80 {
    transparent
  }

  # Notifications 重定向到 websockets server
  proxy /notifications/hub <SERVER>:3012 {
    websocket
  }

  # 将 Root 目录代理到 Rocket
  proxy / <SERVER>:80 {
    transparent
  }
}
```

## Caddy 2.x

同样，Caddy 2 在某些情况下也可以自动启用 HTTPS，参考[此文档](https://caddyserver.com/docs/automatic-https)。

```python
# Caddyfile V2.0 配置文件
:80 {
  # 容器中的 80 端口上的 Caddy 到 bitwarden_rs 私有实例
  # 如果 Caddy 背后有另一个反向代理，例如 Synology 上的嵌入式代理，则使用它
  log {
	output file {env.LOG_FILE}
	level INFO
	# roll_size 5MiB #在 Caddy V2.0.0 Beta20 上无法工作 https://caddyserver.com/docs/caddyfile/directives/log#log
	# roll_keep 2 #在 Caddy V2.0.0 Beta20 无法工作 https://caddyserver.com/docs/caddyfile/directives/log#log
  }
  encode gzip

  header / {
       # 启用 cross-site filter (XSS) 并告诉浏览器阻止检测到的攻击
       X-XSS-Protection "1; mode=block"
       # 禁止在框架内渲染站点 (clickjacking protection)
       X-Frame-Options "DENY"
       # 防止搜索引擎收录 (可选)
       X-Robots-Tag "none"
       # 移除服务器名称
       -Server
   }

  # 谈判端点也被代理到 Rocket
  reverse_proxy /notifications/hub/negotiate <SERVER>:80

  # Notifications 重定向到 websockets 服务器
  reverse_proxy /notifications/hub <SERVER>:3012

  # 代理 Root 目录到 Rocket
  reverse_proxy <SERVER>:80
}

#{env.DOMAIN}:443 {
#  # 容器中的 443 端口上的 Caddy 到 bitwarden_rs 私有实例 
#  # 如果 Caddy 暴露到网络中，则使用它 
#
#  log {
#	output file {env.LOG_FILE}
#	level INFO
#   #roll_size 5MiB # 在 Caddy V2.0.0 Beta20 上无法工作 https://caddyserver.com/docs/caddyfile/directives/log#log
#   #rool_keep 30 # 在 Caddy V2.0.0 Beta20 上无法工作 https://caddyserver.com/docs/caddyfile/directives/log#log
#  }
#
#  # 仅取消注释两行中的一行。取决于你提供自己的证书还是从 Let's Encrypt 请求证书
#  tls {env.SSLCERTIFICATE} {env.SSLKEY}
#  tls {env.EMAIL}
#
#  encode gzip
#
#  header / {
#       # 启用 HTTP Strict Transport Security (HSTS)
#       Strict-Transport-Security "max-age=31536000;"
#       # 启用 cross-site filter (XSS) 并告诉浏览器阻止检测到的攻击
#       X-XSS-Protection "1; mode=block"
#       # 禁止在框架内渲染站点 (clickjacking protection)
#       X-Frame-Options "DENY"
#       # 防止搜索引擎收录 (可选)
#       X-Robots-Tag "none"
#       # 移除服务器名称
#       -Server
#   }
#  # 协商端点也被代理到 Rocket
#  reverse_proxy /notifications/hub/negotiate <SERVER>:80
#
#  # Notifications 重定向到 websockets 服务器
#  reverse_proxy /notifications/hub <SERVER>:3012
#
#  # 代理 Root 目录到 Rocket
#  reverse_proxy <SERVER>:80
#}
```

## Nginx \(by shauder\)

```python
server {
  listen 443 ssl http2;
  server_name vault.*;
  
  # 如果使用共享的 SSL，请指定 SSL 配置。
  # 包含 conf.d/ssl/ssl.conf;
  
  # 允许大型附件
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

  # 除 AUTH_TOKEN 外，还可以选择性添加额外的身份认证
  # 如果您不需要，删除这部分即可
  location /admin {
    # 参考: https://docs.nginx.com/nginx/admin-guide/security-controls/configuring-http-basic-authentication/
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

## Nginx with sub-path \(by BlackDex\)

在这个示例中，bitwarden\_rs 的访问地址为 `https://bitwarden.example.tld/vault/`，如果您想使用任何其他的子路径，比如 `bitwarden` 或 `secret-vault`，您需要更改下面示例中相应的地方。

为此，您需要配置 `DOMAIN` 变量以使其匹配，它应类似于：

```python
; Add the sub-path! Else this will not work!
DOMAIN=https://bitwarden.example.tld/vault/
```

```python
# 在这里定义服务器的 IP 和端口。
upstream bitwardenrs-default { server 127.0.0.1:8080; }
upstream bitwardenrs-ws { server 127.0.0.1:3012; }

# 将 HTTP 重定向到 HTTPS
server {
    listen 80;
    listen [::]:80;
    server_name bitwardenrs.example.tld;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name bitwardenrs.example.tld;

    # 根据需要制定 SSL 配置
    #ssl_certificate /path/to/certificate/letsencrypt/live/bitwardenrs.example.tld/fullchain.pem;
    #ssl_certificate_key /path/to/certificate/letsencrypt/live/bitwardenrs.example.tld/privkey.pem;
    #ssl_trusted_certificate /path/to/certificate/letsencrypt/live/bitwardenrs.example.tld/fullchain.pem;

    client_max_body_size 128M;

    ## 使用子路径配置
    # 您的安装的 root 的路径
    location /vault/ {
      proxy_set_header Host $host;
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header X-Forwarded-Proto $scheme;

      proxy_pass http://bitwardenrs-default;
    }

    location /vault/notifications/hub/negotiate {
      proxy_set_header Host $host;
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header X-Forwarded-Proto $scheme;

      proxy_pass http://bitwardenrs-default;
    }

    location /vault/notifications/hub {
      proxy_set_header Upgrade $http_upgrade;
      proxy_set_header Connection $http_connection;
      proxy_set_header X-Real-IP $remote_addr;

      proxy_pass http://bitwardenrs-ws;
    }

    # 除了 ADMIN_TOKEN 之外，还可以选择添加额外的认证
    # 如果你不想要，就把这部分删掉
    location ^~ /vault/admin {
      # 参考: https://docs.nginx.com/nginx/admin-guide/security-controls/configuring-http-basic-authentication/
      auth_basic "Private";
      auth_basic_user_file /path/to/htpasswd_file;

      proxy_set_header Host $host;
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header X-Forwarded-Proto $scheme;

      proxy_pass http://bitwardenrs-default;
    }

}
```

## Nginx \(by ypid\)

使用 DebOps 配置 nginx 作为 bitwarden\_rs 的反向代理的清单示例。 我选择在 URL 中使用 PSK 以获得额外的安全性，从而不会将 API 公开给 Internet 上的每个人，因为客户端应用程序尚不支持客户端证书（我对其进行了测试）。 注意：使用 subpath/PSK 需要修补源代码并重新编译，请参考：[https://github.com/dani-garcia/bitwarden\_rs/issues/241\#issuecomment-436376497](https://github.com/dani-garcia/bitwarden_rs/issues/241#issuecomment-436376497)。 /admin 未经测试。 有关安全性子路径托管的一般讨论，请参阅：[https://github.com/debops/debops/issues/1233](https://github.com/debops/debops/issues/1233)

```python
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

      ## 所有的安全 HTTP 头也需要由 nginx 来设置。
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

      ## 只要能显示出从我们的凭证到服务器的所有域名，就不要使用图标功能。
      - pattern: '/ekkP9wtJ_psk_changeme_Hr9CCTud/icons/'
        options: |-
          access_log off;
          log_not_found off;
          deny all;
```

## Nginx \(NixOS\)\(by tklitschi\)

NixOS  Nginx 配置示例。关于 NixOS 部署的更多信息，请参阅[部署示例](deployment-examples.md)页面。

```python
{ config, ... }:
{
  security.acme.acceptTerms = true;
  security.acme.email = "me@example.com";
  security.acme.certs = {

    "bw.example.com" = {
      group = "bitwarden_rs";
      keyType = "rsa2048";
      allowKeysForGroup = true;
    };
  };

  services.nginx = {
    enable = true;

    recommendedGzipSettings = true;
    recommendedOptimisation = true;
    recommendedProxySettings = true;
    recommendedTlsSettings = true;

    virtualHosts = {
      "bw.example.com" = {
        forceSSL = true;
        enableACME = true;
        locations."/" = {
          proxyPass = "http://localhost:8812"; # 由于某些冲突，这里更改了默认的 rocket 端口
          proxyWebsockets = true;
        };
        locations."/notifications/hub" = {
          proxyPass = "http://localhost:3012";
          proxyWebsockets = true;
        };
        locations."/notifications/hub/negotiate" = {
          proxyPass = "http://localhost:8812";
          proxyWebsockets = true;
        };
      };
    };
  };
}
```

## Apache \(by fbartels\)

记得启用 `mod_proxy_wstunnel`，例如：`a2enmod proxy_wstunnel`。

```python
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

## Apache in a sub-location \(by ss89\)

需确保在 apache 配置中的某个位置加载了 websocket 代理模块。 它看起来像这样：

```python
LoadModule proxy_wstunnel_module modules/mod_proxy_wstunnel.so`
```

```python
<VirtualHost *:443>
    SSLEngine on
    ServerName $hostname.$domainname

    SSLCertificateFile ${SSLCERTIFICATE}
    SSLCertificateKeyFile ${SSLKEY}
    SSLCACertificateFile ${SSLCA}
    ${SSLCHAIN}

    ErrorLog \${APACHE_LOG_DIR}/error.log
    CustomLog \${APACHE_LOG_DIR}/access.log combined

    <Location /bitwarden> # 如果需要，调整此处
        RewriteEngine On
        RewriteCond %{HTTP:Upgrade} =websocket [NC]
        RewriteRule /notifications/hub(.*) ws://<SERVER>:3012/$1 [P,L]
        ProxyPass http://<SERVER>:80/

        ProxyPreserveHost On
        RequestHeader set X-Real-IP %{REMOTE_ADDR}s
    </Location>
</VirtualHost>
```

## Traefik v1 \(docker-compose 示例\)

```python
labels:
    - traefik.enable=true
    - traefik.docker.network=traefik
    - traefik.web.frontend.rule=Host:bitwarden.domain.tld
    - traefik.web.port=80
    - traefik.hub.frontend.rule=Host:bitwarden.domain.tld;Path:/notifications/hub
    - traefik.hub.port=3012
    - traefik.hub.protocol=ws
```

## Traefik v2 \(docker-compose 示例 by hwwilliams\) <a id="traefik-v-2-docker-compose-example-by-hwwilliams"></a>

### 将 Traefik v1 标签迁移到 Traefik v2 <a id="traefik-v-1-labels-migrated-to-traefik-v2"></a>

```python
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

### 迁移的标签加上 HTTP 到 HTTPS 重定向 <a id="migrated-labels-plus-http-to-https-redirect"></a>

这些标签假定 Traefik 中为端口 80 和 443 定义的入口点分别是“web”和“websecure”。

这些标签还假定您已经在 Traefik 中定义了默认的证书解析器。

```python
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

## HAproxy \(by BlackDex\)

将这些行添加到您的 haproxy 配置中。

```python
frontend bitwarden_rs
    bind 0.0.0.0:80
    option forwardfor header X-Real-IP
    http-request set-header X-Real-IP %[src]
    default_backend bitwarden_rs_http
    use_backend bitwarden_rs_ws if { path_beg /notifications/hub } !{ path_beg /notifications/hub/negotiate }

backend bitwarden_rs_http
    # Enable compression if you want
    # compression algo gzip
    # compression type text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript
    server bwrshttp 0.0.0.0:8080

backend bitwarden_rs_ws
    server bwrsws 0.0.0.0:3012
```

