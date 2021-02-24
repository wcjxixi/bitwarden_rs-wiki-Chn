# =6.启用 HTTPS

{% hint style="success" %}
对应的[页面地址](https://github.com/dani-garcia/bitwarden_rs/wiki/Enabling-HTTPS)
{% endhint %}

现在几乎都需要启用 HTTPS，才能满足 bitwarden\_rs 的正常操作，这是因为 Bitwarden 网络密码库使用的 [Web Crypto API](https://developer.mozilla.org/en-US/docs/Web/API/SubtleCrypto)，大多数浏览器只有在 HTTPS 环境下才能使用。

有几种方法启用 HTTPS：

* （推荐）把 bitwarden\_rs 放在一个反向代理后面，代表 bitwarden\_rs 处理 HTTPS 连接。
* （不推荐）启用 bitwarden\_rs 内置的 HTTPS 功能（通过 Rocket Web 框架）。Rocket 的 HTTPS 实现相对不成熟且有限。此方法也不支持 WebSocket 通知。

关于这些选项的更多细节，请参考启用 HTTPS 部分。

要使 HTTPS 服务器工作，它还需要 SSL/TLS 证书，因此您需要决定如何获得该证书。同样，有几个选项：

* \(推荐\)使用ACME客户端获取Let's Encrypt证书。一些反向代理（如Caddy）也内置支持使用ACME协议获取证书。
* （推荐）如果你信任Cloudflare来代理你的流量，你可以让他们处理你的SSL/TLS证书的发放。请注意，上游的Bitwarden网络保险库\([https://vault.bitwarden.com/\)运行在Cloudflare后面。](https://vault.bitwarden.com/%29运行在Cloudflare后面。)
* \(不推荐\)建立一个私人CA，并发行你自己的\(自签名\)证书。这有各种陷阱和不便，所以考虑自己的警告。

参考[获取 SSL/TLS 证书](enabling-https.md#getting-ssl-tls-certificates)部分，以了解这些选项的更多细节。

## 启用 HTTPS <a id="enabling-https"></a>

### 通过反向代理 <a id="via-a-reverse-proxy"></a>

有很多常用的反向代理。 在[代理示例](../deployment/proxy-examples.md)中可以找到一些配置示例。 如果您不熟悉反向代理并且没有偏好，请首先考虑 [Caddy](https://caddyserver.com/)，因为它内置了对获取 Let's Encrypt 证书的支持。

### 通过 Rocket <a id="via-rocket"></a>

{% hint style="warning" %}
不建议使用此方法。
{% endhint %}

要对 `bitwarden_rs` 本身启用 HTTPS，请设置如下格式的 `ROCKET_TLS` 环境变量：

```python
ROCKET_TLS={certs="/path/to/certs.pem",key="/path/to/key.pem"}
```

位置：

* `certs`：PEM 格式的 SSL/TLS 证书链的路径
* `key`：PEM 格式的 SSL/TLS 证书对应的私钥文件的路径

说明：

* `ROCKET_TLS` 行中使用的文件_扩展名_不一定非要像示例中那样是 `.PEM`。某些地方可能会使用其他扩展名，例如，`.crt` 作为证书，`.key` 作为私钥。这些文件的_格式_必须是 PEM，即 base64 编码。而 PEM 只是 openssl 的默认格式，因此您可以将 .cert、.cer、.crt 和 .key 文件重命名为 .pem，反之亦然，使用 .crt 或 .key 作为 `ROCKET_TLS` 行中的文件扩展名。
* 使用 RSA 证书/密钥。Rocket 似乎无法处理 ECC 证书/密钥，否则会输出类似下面的误导性错误消息：

  > `[ERROR] environment variable ROCKET_TLS={certs="/ssl/ecdsa.crt",key="/ssl/ecdsa.key"} could not be parsed`

  （环境变量本身的格式没有错误；只是因为 Rocket 无法解析证书/密钥内容。）

* 如果在 Docker 下运行，请记住，bitwarden\_rs 在容器内部运行时将解析 `ROCKET_TLS` 值 ，所以请确保 `certs` 和 `key` 路径是容器内部呈现的样子（可能与 Docker 主机系统上的路径不同）。

```python
docker run -d --name bitwarden \
  -e ROCKET_TLS='{certs="/ssl/certs.pem",key="/ssl/key.pem"}' \
  -v /ssl/keys/:/ssl/ \
  -v /bw-data/:/data/ \
  -p 443:80 \
  bitwardenrs/server:latest
```

您需要挂载 ssl 文件夹（使用 -v 参数），同时需要转发适当的端口（使用 -p 参数），通常是用于 HTTPS 连接的 443 端口。如果您选择的端口号不是 443，例如 3456，请记住在连接到服务时明确提供该端口号，例如：`https://bitwarden.local:3456`。

有关如何在本地系统上设置和使用私有 CA 的更多信息，请参阅[此页面](../other-information/private-ca-and-self-signed-certs-that-work-with-chrome.md)。如果遵循该指南，您的 ROCKET\_TLS 行看起来应该像这样：`-e ROCKET_TLS='{certs="/ssl/bitwarden.crt",key="/ssl/bitwarden.key"}' \`

由于 Android 中可能存在证书验证错误，因此您需要确保你的证书包含有完整的信任链。对于 certbot，这意味着应使用 `fullchain.pem` 而不是 `cert.pem`。

用于获取证书的软件通常使用符号链接。如果是这样的话，需要确保这两个位置能被 docker 容器访问到。

例如：[certbot](https://certbot.eff.org/) 会在 `/etc/letsencrypt/live/mydomain/` 下创建一个包含所需要的 `fullchain.pem` 和 `privkey.pem` 文件的文件夹。

这些文件链接到  `../../archive/mydomain/privkey.pem`。

因此，从 bitwarden 容器中使用，应像这样：

```python
docker run -d --name bitwarden \
  -e ROCKET_TLS='{certs="/ssl/live/mydomain/fullchain.pem",key="/ssl/live/mydomain/privkey.pem"}' \
  -v /etc/letsencrypt/:/ssl/ \
  -v /bw-data/:/data/ \
  -p 443:80 \
  bitwardenrs/server:latest
```

#### 检查证书是否有效 <a id="check-if-certificate-is-valid"></a>

当您的 bitwarden\_rs 服务器对外界可用时，您可以使用 [https://comodosslstore.com/ssltools/ssl-checker.php](https://comodosslstore.com/ssltools/ssl-checker.php) 网站来检查 SSL 证书是否包含证书链。缺少证书链，Android 设备将无法连接。

您也可以使用 [https://www.ssllabs.com/ssltest/analyze.html](https://www.ssllabs.com/ssltest/analyze.html) 网站进行检查，但是它不支持自定义端口。另外，请记住选中“Do not show the results on the boards”复选框，否则您的系统将在“Recently Seen”列表中可见。

如果您运行的是没有与公共 Internet 连接的本地服务器，则可以使用 openssl 工具来验证您的证书。

执行以下操作以验证证书是否随链安装（注意将 vault.domain.com 改为您自己的域名）：

```python
openssl s_client -showcerts -connect vault.domain.com:443

# 或者您使用的其他端口，比如 7070
openssl s_client -showcerts -connect vault.domain.com:7070
```

输出的开头应类似于以下内容（使用 Let's Encrypt 证书）：

```python
CONNECTED(00000003)
depth=2 O = Digital Signature Trust Co., CN = DST Root CA X3
verify return:1
depth=1 C = US, O = Let's Encrypt, CN = Let's Encrypt Authority X3
verify return:1
depth=0 CN = vault.domain.com
verify return:1
```

有 3 个不同深度（请注意，它是从 0 开始的）级别的验证。在接下来的输出中，您应该看到来自 Let's Encryptbase 的使用 base64 编码的证书信息。

## 获取 SSL/TLS 证书 <a id="getting-ssl-tls-certificates"></a>

### 通过 Let's Encrypt <a id="via-lets-encrypt"></a>

### 通过 Cloudflare <a id="via-cloudflare"></a>

