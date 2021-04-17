# 2.使用 Let's Encrypt 证书运行私有 bitwarden\_rs 实例

{% hint style="success" %}
对应的[页面地址](https://github.com/dani-garcia/bitwarden_rs/wiki/Running-a-private-bitwarden_rs-instance-with-Let%27s-Encrypt-certs)
{% endhint %}

假设你希望运行一个只能从本地网络访问的 bitwarden\_rs 实例，但你又希望此实例启用 HTTPS，此 HTTPS 证书由一个被广泛接受的 CA 而不是你自己的[私有 CA](../../other-information/private-ca-and-self-signed-certs-that-work-with-chrome.md) 来签署（以避免将专用CA证书加载到所有设备中的麻烦）。

本文将演示如何使用 [Caddy](https://caddyserver.com/) Web 服务器创建这样的设置，Caddy 内置了对诸多 DNS 提供商的 ACME 支持。我们将通过 ACME [DNS 验证方式](https://letsencrypt.org/docs/challenge-types/#dns-01-challenge)获取 Let's Encrypt 证书来配置 Caddy -- 在这里使用通常的 HTTP 验证方式的话会有问题，因为它依赖于 Let's Encrypt 服务器能够访问到你的内部 Web 服务器。

涵盖了两个 DNS 提供商：

* [Duck DNS](https://www.duckdns.org/) -- 为你提供一个 `duckdns.org` 下的子域名（例如 `my-bwrs.duckdns.org`）。如果你没有自己的域名，此选项是最简单的。 
* [Cloudflare](https://www.cloudflare.com/) -- 这可以让你把你的 bitwarden\_rs 实例放在你拥有或控制的域名下。请注意，Cloudflare 可以只作为一个 DNS 提供商使用（即不使用 Cloudflare 最著名的代理功能）。如果你目前没有自己的域名，你也许可以在 [Freenom](https://www.freenom.com/) 获得一个免费的域名。

当然也可以使用其他的网络服务器、[ACME 客户端](https://letsencrypt.org/docs/client-options/)和 DNS 提供商的组合来创建类似的设置，但你必须解决细节上的差异。

## 获取自定义 Caddy 构建 <a id="getting-a-custom-caddy-build"></a>

由于大多数人不使用 DNS 验证方式，为每个 DNS 提供商自定义实现，因此 Caddy 默认没有内置此验证方式的支持。

最简单的方式是通过 [https://caddyserver.com/download](https://caddyserver.com/download) 获取带有 DNS 验证模块的 Caddy 版本。选择您的平台，选中 `github.com/caddy-dns/cloudflare`（用于 Cloudflare）和/或 `github.com/caddy-dns/lego-deprecated`（用于 Duck DNS），然后点击下载。

如果你喜欢从源代码构建，你可以使用 [`xcaddy`](https://caddyserver.com/docs/build#xcaddy)。例如，要创建一个包含 Cloudflare 和 Duck DNS 支持的构建：

```python
xcaddy build --with github.com/caddy-dns/cloudflare --with github.com/caddy-dns/lego-deprecated
```

将 `caddy` 二进制 移动到 `/usr/local/bin/caddy` 或其他合适的目录中。（可选）运行语句 `sudo setcap cap_net_bind_service=+ep /usr/local/bin/caddy` 以允许 `caddy` 而在特权端口（&lt; 1024）上监听，而无须以 root 身份运行。

## Duck DNS 设置 <a id="duck-dns-setup"></a>

如果您还没有账户，请在 [https://www.duckdns.org/](https://www.duckdns.org/) 创建一个。给您的 bitwarden\_rs 实例创建一个子域名（例如，`my-bwrs.duckdns.org`），将其 IP 地址设置为你的 bitwarden\_rs 主机的私有 IP（例如，192.168.1.100）。记下你的账户的 token 值（[UUID](https://en.wikipedia.org/wiki/UUID) 格式的字符串）。Caddy 将需要此 token 来完成 DNS 验证。

创建一个名为 `Caddyfile` 的文件，内容如下：

```python
{$DOMAIN}:443 {
    tls {
        dns lego_deprecated duckdns
    }
    reverse_proxy localhost:8080
    reverse_proxy /notifications/hub localhost:3012
}
```

创建一个名为 `caddy.env` 的文件，内容如下（替换相应的值）：

```python
DOMAIN=my-bwrs.duckdns.org
DUCKDNS_TOKEN=00112233-4455-6677-8899-aabbccddeeff
```

运行命令以启动 `caddy`：

```python
caddy run -envfile caddy.env
```

运行命令以启动 `bitwarden_rs`：

```python
export ROCKET_PORT=8080
export WEBSOCKET_ENABLED=true

./bitwarden_rs
```

您现在应该可以通过 `https://my-bwrs.duckdns.org` 访问到您的实例了。

## Cloudflare 设置 <a id="cloudflare-setup"></a>

如果您还没有账户，请在 [https://www.cloudflare.com/](https://www.cloudflare.com/) 创建一个；您还需要到您的域名注册商那里将名称服务器设置为 Cloudflare 分配给您的值。为您的 bitwarden\_rs 实例创建一个子域名（例如，`bwrs.example.com`），将其 IP 地址设置为您的 bitwarden\_rs 主机的私有 IP（例如，`192.168.1.100`）。例如：

![](https://camo.githubusercontent.com/0e3cc1847c048fa874c2ca42d79b734d2eee88e0b36bfae7a52e1cf2a04a0b91/68747470733a2f2f692e696d6775722e636f6d2f4242767934596a2e706e67)

创建一个用于 DNS 验证的 API token（更多背景知识，请参阅 [https://github.com/libdns/cloudflare/blob/master/README.md](https://github.com/libdns/cloudflare/blob/master/README.md)）：

1. 点击右上角的个人图标并导航到 `My Profile`，然后选择  `API Tokens` 选项卡。
2. 点击 `Create Token` 按钮，然后点击`Edit zone DNS` 右边的 `Use template`。
3. 编辑 `Token name` 字段（如果您希望使用更具描述性的名称）。
4. 在 `Permissions` 下应出现一个权限：`Zone / DNS / Edit`。添加其他权限：`Zone / Zone / Read`。
5. 在 `Zone Resources` 下，设置 `Include / Specific zone / example.com`（使用您自己的域名替换 `example.com`）。
6. 在 `TTL` 下，为您的 tokan 设置一个变为非活动状态的 End Date。您也可以在以后设置。
7. 创建 token 并复制 token 值。

您的 token 列表看起来应该像这样：

![](https://camo.githubusercontent.com/3317aacd91dd3a80b0a9689929ded89c1b384749ec7eda07bebccae2d79ceba0/68747470733a2f2f692e696d6775722e636f6d2f466f4f763957772e706e67)

创建一个名为 `Caddyfile` 的文件，内容如下：

```python
{$DOMAIN}:443 {
    tls {
        dns cloudflare {env.CLOUDFLARE_API_TOKEN}
    }
    reverse_proxy localhost:8080
    reverse_proxy /notifications/hub localhost:3012
}
```

创建一个名为 `caddy.env` 的文件，内容如下（替换相应的值）：

```python
DOMAIN=bwrs.example.com
CLOUDFLARE_API_TOKEN=<your-api-token>
```

运行命令以启动 `caddy`：

```python
caddy run -envfile caddy.env
```

运行命令以启动 `bitwarden_rs`：

```python
export ROCKET_PORT=8080
export WEBSOCKET_ENABLED=true

./bitwarden_rs
```

您现在应该可以通过 `https://bwrs.example.com` 访问到您的实例了。

## 使用 `lego` CLI 获取证书 <a id="getting-certs-using-the-lego-cli"></a>

在上面的 DuckDNS 例子中，Caddy 使用 `lego` 库通过 DNS 验证获取证书。`lego`  也有一个 CLI，你可以直接使用它来获取证书，例如，如果你想使用 Caddy 以外的反向代理。 \(注意：这个例子使用 `lego`，但也有其他独立的 ACME 客户端支持 DNS 验证方式（参见 [DNS 验证](running-a-private-bitwarden_rs-instance-with-lets-encrypt-certs.md#dns-challenge)部分）。

下面是一个如何做到这一点的例子。

1. 从 [https://github.com/go-acme/lego/releases](https://github.com/go-acme/lego/releases) 下载预建的 `lego` 二进制文件到您的系统中。解压内容到某个目录，比如 `/usr/local/lego`。
2. 从那个目录中，运行 `DUCKDNS_TOKEN=<token> ./lego -a --dns duckdns -d my-bwrs.duckdns.org -m me@example.com run`（用合适的值替换令牌、域名和电子邮件地址）。这将使你在 Let's Encrypt 注册，并为你的域名获取一个证书。
3. 设置一个每周的 cron 作业来运行 `DUCKDNS_TOKEN=<token> ./lego --dns duckdns -d my-bwrs.duckdns.org -m me@example.com renew`。这将在你的证书即将到期时更新它。

{% hint style="warning" %}
`lego` 默认请求 ECC/ECDSA 证书。如果你使用 bitwarden\_rs 中内置的 [Rocket HTTPS 服务器](enabling-https.md#via-rocket)，你需要请求 RSA 证书。在上面的 `lego` 命令中，添加选项 `--key-type rsa2048`。
{% endhint %}

在这个例子中，用于配置您的反向代理所生成的输出为：

* `/usr/local/lego/.lego/certificates/my-bwrs.duckdns.org.crt` （证书）
* `/usr/local/lego/.lego/certificates/my-bwrs.duckdns.org.key` （私钥）

## 参考 <a id="references"></a>

### DNS 验证 <a id="dns-challenge"></a>

* [https://caddy.community/t/how-to-use-dns-provider-modules-in-caddy-2/8148](https://caddy.community/t/how-to-use-dns-provider-modules-in-caddy-2/8148)
* [https://community.letsencrypt.org/t/dns-providers-who-easily-integrate-with-lets-encrypt-dns-validation/86438](https://community.letsencrypt.org/t/dns-providers-who-easily-integrate-with-lets-encrypt-dns-validation/86438)

### Caddy Cloudflare 组件 <a id="caddy-cloudflare-module"></a>

* [https://github.com/caddy-dns/cloudflare](https://github.com/caddy-dns/cloudflare)

### Caddy Duck DNS 组件 <a id="caddy-duck-dns-module"></a>

* [https://github.com/caddy-dns/lego-deprecated](https://github.com/caddy-dns/lego-deprecated)
* [https://go-acme.github.io/lego/dns/duckdns/](https://go-acme.github.io/lego/dns/duckdns/)

