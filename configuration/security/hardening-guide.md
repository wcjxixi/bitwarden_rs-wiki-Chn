# 1.强化指南

{% hint style="success" %}
对应的[页面地址](https://github.com/dani-garcia/bitwarden_rs/wiki/Hardening-Guide)
{% endhint %}

## 应用程序配置 <a id="application-configuration"></a>

下面的小节涵盖了 vaultwarden 本身相关的强化。

### 禁用注册和（可选）邀请 <a id="disable-registration-and-optionally-invitations"></a>

默认情况下，vaultwarden 允许任何匿名用户在未被邀请的情况下在服务器上注册新帐户。这是在服务器上创建第一个用户所必需的，但建议您在管理面板中（如果启用了管理面板的话）或[使用环境变量](../disable-registration-of-new-users.md)将其禁用，以防止攻击者在 vaultwarden 服务器上创建帐户。

vaultwarden 还允许注册用户邀请其他新用户在服务器上创建帐户并加入其组织。只要您信任用户，这不会带来直接风险，但是可以在管理面板或[使用环境变量](../disable-registration-of-new-users.md)将其禁用。

### 禁用显示密码提示 <a id="disable-password-hint-display"></a>

vaultwarden 在登录页面上显示密码提示，以适应未配置 SMTP 的小型/本地部署，攻击者可能会滥用这些密码来促进针对服务器用户的猜测密码攻击。可以在管理面板中通过取消选中 `Show password hints` 选项或[使用环境变量](../disable-registration-of-new-users.md)来禁用它。

## HTTPS / TLS 配置 <a id="https-tls-configuration"></a>

下面的小节涵盖了 HTTPS/TLS 相关的强化。

### 严格 SNI <a id="strict-sni"></a>

[SNI](https://zh.wikipedia.org/wiki/%E6%9C%8D%E5%8A%A1%E5%99%A8%E5%90%8D%E7%A7%B0%E6%8C%87%E7%A4%BA) 是网络浏览器请求 HTTPS 服务器为特定网站（如 `vaultwarden.example.com`）提供 SSL/TLS 证书的方式。假设`vaultwarden.example.com` 的 IP 地址是 `1.2.3.4`。理想情况下，你希望你的实例只能通过 https://vaultwarden.example.com ，并且不能通过 https://1.2.3.4 进行访问。这是因为 IP 地址会因为各种原因被不断扫描，如果能通过这种方式检测到你的 vaultwarden 实例，就会成为一个更明显的目标。例如，一个简单的 [Shodan 搜索](https://www.shodan.io/search?query=bitwarden)就会发现一些通过 IP 地址访问的 Bitwarden 实例。

### 反向代理 <a id="reverse-proxying"></a>

一般来说，你应该避免通过 vaultwarden 内置的 [Rocket TLS 支持](../../deployment/https/enabling-https.md)启用 HTTPS，特别是当你的实例是公开访问的时候。Rocket 本身列出了如下[警告](https://rocket.rs/v0.4/guide/configuration/#configuring-tls)：

> Rocket's built-in TLS is not considered ready for production use. It is intended for development use only. （Rocket 内置的 TLS 还不能用于生产。它只用于开发用途。）

比如，Rocket TLS 不支持严格 SNI 或 ECC 证书（仅 RSA）。

请参看[代理示例](../../deployment/proxy-examples.md)，以了解反向代理配置的案例。

## Docker 配置 <a id="docker-configuration"></a>

下面的小节涵盖了 Docker 相关的强化。

### 以非 root 用户运行 <a id="run-as-a-non-root-user"></a>

vaultwarden Docker 镜像被配置为默认以 root 用户的身份运行容器进程。这允许 vaultwarden 在没有权限问题的情况下读取/写入 [bind-mounted](https://docs.docker.com/storage/bind-mounts/) 到容器中的任何数据，即使这些数据是由另一个用户（例如，你在 Docker 主机上的用户账户）拥有的。默认配置在安全性和可用性之间取得了很好的平衡--在一个无权限的 Docker 容器中以 root 身份运行，本身就提供了合理的隔离度，同时也让那些不是非常精通如何在 Linux 上管理所有权/权限的用户更容易进行设置。然而，作为通用策略，从安全的角度来说，以所需的最低权限运行进程是更好的；对于用 Rust 等内存安全语言编写的程序来说，这一点就不那么重要了，但请注意，vaultwarden 也使用了一些用 C 语言编写的库代码（例如 SQLite、OpenSSL、MySQL、PostgreSQL 等）。

要在 Docker 中以非 root 用户（uid/gid 1000）的身份运行容器进程（vaultwarden）：

```python
docker run -u 1000:1000 -e ROCKET_PORT=8080 -p <host-port>:8080 \
       [...other args...] \
       vaultwarden/server:latest
```

在许多 Linux 发行版中，默认用户的 uid/gid 为 1000（运行 `id` 命令进行验证），所以如果你想在不换成其他用户的情况下轻松地访问你的 vaultwarden 数据，这是一个很好的值，但你可以根据需要调整 uid/gid。请注意，你很可能需要指定一个数字 uid/gid，因为 vaultwarden 容器不共享用户/组名到 uid/gid 的相同映射（例如，将容器中的 `/etc/passwd` 和 `/etc/group` 文件与 Docker 主机上的文件对比）。

`ROCKET_PORT` 默认为 80，这是一个[特权端口](https://www.w3.org/Daemon/User/Installation/PrivilegedPorts.html)；当以非 root 用户身份运行时，它需要是 1024 或更高，否则当 vaultwarden 试图在该端口上绑定和监听连接时，你会得到一个权限拒绝的错误。

要在 `docker-compose` 中进行同样的操作：

```python
services:
  vaultwarden:
    image: vaultwarden/server:latest
    container_name: vaultwarden
    user: 1000:1000
    environment:
      - ROCKET_PORT=8080

    ... other configuration ...
```

由于这里修改的是 `ROCKET_PORT`，所以一定要更新你的反向代理配置，将 vaultwarden 的流量代理到 8080 端口（或者你选择的任何更高的端口），而不是 80。

### 挂载数据到容器中 <a id="mounting-data-into-the-container"></a>

一般来说，只有 vaultwarden 正常运行所需要的数据才应该被挂载到 vaultwarden 容器中（通常情况下，这只是你的数据目录，也许还有一个包含 SSL/TLS 证书和私钥的目录）。不要挂载你的整个主目录，例如，`/var/run/docker.sock` 等，除非你有特定的原因，并且知道你在做什么。

另外，如果你不希望 vaultwarden 修改你挂载的数据（例如，certs），可以通过在卷规范中添加 `:ro` 来[只读挂载它](https://docs.docker.com/storage/bind-mounts/#use-a-read-only-bind-mount)（例如，`docker run -v /home/username/bitwarden-ssl:/ssl:ro`）。

## 杂项 <a id="miscellaneous"></a>

### 暴力破解 <a id="brute-force-mitigation"></a>

当不使用双重身份认证时，（理论上）可以暴力破解用户密码，从而获得对其帐户的访问权限。缓解此问题的一种相对简单的方法是设置 fail2ban，设置后，在过多的失败登录尝试后将阻止访问者的 IP 地址。但是：在多个反向代理（例如 cloudflare）后面使用此功能时，应格外注意。请参阅：[Fail2Ban 设置](fail2ban-setup.md)。

### 隐藏在子目录下 <a id="hiding-under-a-subdir"></a>

通常，Bitwarden 实例驻留在子域的根目录下（即 `bitwarden.example.com`，而不是 `bitwarden.example.com/some/path`）。上游的 Bitwarden 服务器目前只支持子域根目录，而 vaultwarden 则增加了对[备用基础目录](../using-an-alternate-base-dir-subdir-subpath.md)的支持。对于某些用户来说，这很有用，因为他们只能访问一个子域，却想在不同的目录下运行多个服务。在这种情况下，他们通常可以做一些显而易见的选择，比如使用 `mysubdomain.example.com/bitwarden`。然而，你也可以通过把 vaultwarden 放在类似 `mysubdomain.example.com/bitwarden/<mysecretstring>` 这样的目录下来提供额外的保护层，其中 `<mysecretstring>` 有效地充当一个密码。也许有人会说这是[通过隐藏实现安全](https://en.wikipedia.org/wiki/Security_through_obscurity)，但实际上这是深度防御 -- 子目录的隐蔽性只是额外的一层安全保护，而不是为了成为主要的安全手段（用户主密码的强度仍然是主要的安全手段）。

