# 4.使用 Docker Compose

{% hint style="success" %}
对应的[页面地址](https://github.com/dani-garcia/bitwarden_rs/wiki/Using-Docker-Compose)
{% endhint %}

[Docker Compose](https://docs.docker.com/compose/) 是一个用于定义和配置多容器应用程序的工具。在我们的例子中，我们希望 Bitwarden\_rs 服务器和代理都将 WebSocket 请求重定向到正确的地方。

## 带有 HTTP 验证的 Caddy <a id="caddy-with-http-challenge"></a>

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

## 带有 DNS 验证的 Caddy <a id="caddy-with-dns-challenge"></a>

这个示例和上一个示例一样，但适用于您不希望您的实例被公开访问的情况（即您只能从您的本地网络访问它）。这个示例使用 Duck DNS 作为 DNS 提供商。更多的背景资料，以及如何设置 Duck DNS 的细节，请参考[使用 Let's Encrypt 证书运行私有 bitwarden\_rs 实例](../deployment/https/running-a-private-bitwarden_rs-instance-with-lets-encrypt-certs.md)。

首先创建一个新目录，并切换到该目录。接下来，创建下面的 `docker-compose.yml` 文件，确保为 `DOMAIN` 和 `EMAIL` 变量替换适当的值。

```python
version: '3'

services:
  bitwarden:
    image: bitwardenrs/server:latest
    container_name: bitwarden
    restart: always
    environment:
      - WEBSOCKET_ENABLED=true  # 启用 WebSocket 通知。
    volumes:
      - ./bw-data:/data

  caddy:
    image: caddy:2
    container_name: caddy
    restart: always
    ports:
      - 80:80
      - 443:443
    volumes:
      - ./caddy:/usr/bin/caddy  # 您的 Caddy 自定义构建
      - ./Caddyfile:/etc/caddy/Caddyfile:ro
      - ./caddy-config:/config
      - ./caddy-data:/data
    environment:
      - DOMAIN=bitwarden.example.com  # 您的域名
      - EMAIL=admin@example.com       # 用于 ACME 注册的电子邮件地址
      - DUCKDNS_TOKEN=<token>         # 您的 Duck DNS 令牌
      - LOG_FILE=/data/access.log
```

常规的 Caddy 构建（包括 Docker 映像中的构建）不包含 DNS 验证模块，因此接下来，您需要[获取自定义 Caddy 构建](../deployment/https/running-a-private-bitwarden_rs-instance-with-lets-encrypt-certs.md#getting-a-custom-caddy-build)。将自定义构建重命名为 `caddy` 并将其移动到与 `docker-compose.yml` 相同的目录下。确保 `caddy` 文件是可执行的（例如 `chmod a + x caddy`）。上面的 `docker-compose.yml` 文件会将自定义构建绑定挂载到 `caddy:2` 容器中，并替换常规的构建。

在同一目录下，创建下面的 `Caddyfile` 文件（这个文件不需要做修改）。

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
  tls {
    dns lego_deprecated duckdns
  }

  # 此设置可能会在某些浏览器上出现兼容性问题（例如，在 Firefox 上下载附件）
  # 如果遇到问题，请尝试禁用此功能
  encode gzip

  # Notifications 重定向到 WebSocket 服务器
  reverse_proxy /notifications/hub bitwarden:3012

  # 代理所有，但除了 Rocket
  reverse_proxy bitwarden:80
}
```

与 HTTP 验证的示例一样，运行下面的命令以创建并启动容器：

```python
docker-compose up -d
```

