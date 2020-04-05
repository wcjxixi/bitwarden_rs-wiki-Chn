# 启用HTTPS

{% hint style="success" %}
对应的[页面地址](https://github.com/dani-garcia/bitwarden_rs/wiki/Enabling-HTTPS)
{% endhint %}

要对`bitwarden_rs`本身启用HTTPS，请按照以下说明设置`ROCKET_TLS`环境变量。但是，由于Rocket的TLS支持相对不成熟，因此除非您确实需要最小化依赖关系，否则通常最好使用更成熟的[反向代理](https://github.com/dani-garcia/bitwarden_rs/wiki/Proxy-examples)方式。

该选项的值必须遵循以下格式：

```php
ROCKET_TLS={certs="/path/to/certs.pem",key="/path/to/key.pem"}
```

路径：

* certs：PEM格式的证书链文件的路径
* key：PEM格式的证书私钥文件的路径

说明：

* `ROCKET_TLS`行中使用的文件_扩展名_不一定非要像示例中那样是PEM。重要的是需要的文件_格式_是PEM，即base64编码格式。由于PEM格式是openssl的默认格式，因此您可以将.cert、.cer、.crt和.key文件重命名为.pem，反之亦然，使用.crt或.key作为`ROCKET_TLS`行中的文件扩展名。
* 使用RSA证书/密钥。Rocket似乎无法处理ECC证书/密钥，否则会输出类似下面的误导性错误消息：

  > `[ERROR] environment variable ROCKET_TLS={certs="/ssl/ecdsa.crt",key="/ssl/ecdsa.key"} could not be parsed`

  （环境变量本身的格式没有错误；这是因为Rocket无法解析证书/密钥内容。）

```php
docker run -d --name bitwarden \
  -e ROCKET_TLS='{certs="/ssl/certs.pem",key="/ssl/key.pem"}' \
  -v /ssl/keys/:/ssl/ \
  -v /bw-data/:/data/ \
  -p 443:80 \
  bitwardenrs/server:latest
```

您需要挂载ssl文件夹（使用-v参数），同时需要转发适当的端口（使用-p参数），通常使用HTTPS端口443。如果您选择的端口号不是443，例如3456，请记住在连接到服务时明确提供该端口号，例如：`https://bitwarden.local:3456`。

有关如何在本地系统上设置和使用私有CA的更多信息，请参阅此[WiKi页面](https://github.com/dani-garcia/bitwarden_rs/wiki/Private-CA-and-self-signed-certs-that-work-with-Chrome)。如果遵循该指南，您的ROCKET\_TLS行看起来应该像这样：

```php
-e ROCKET_TLS='{certs="/ssl/bitwarden.crt",key="/ssl/bitwarden.key"}' \
```

由于Android中可能存在证书验证错误，因此您需要确保你的证书包拥有完整的信任链。对于certbot，这意味着使用`fullchain.pem`而不是`cert.pem`。

用于获取证书的软件通常使用符号链接。如果是这样的话，需要确保这两个位置能被docker容器访问到。

例如：[certbot](https://certbot.eff.org/)将在`/etc/letsencrypt/live/mydomain/`下创建一个包含所需要的`fullchain.pem`和`privkey.pem`文件的文件夹

这些文件链接到 `../../archive/mydomain/privkey.pem`

因此，从bitwarden容器中这样使用：

```php
docker run -d --name bitwarden \
  -e ROCKET_TLS='{certs="/ssl/live/mydomain/fullchain.pem",key="/ssl/live/mydomain/privkey.pem"}' \
  -v /etc/letsencrypt/:/ssl/ \
  -v /bw-data/:/data/ \
  -p 443:80 \
  bitwardenrs/server:latest
```

#### 检查证书是否有效

当您的bitwarden\_rs服务器可供外界使用时，您可以使用[https://comodosslstore.com/ssltools/ssl-checker.php](https://comodosslstore.com/ssltools/ssl-checker.php)网站来检查SSL证书是否包含证书链。缺少证书链，Android设备将无法连接。

您也可以使用[https://www.ssllabs.com/ssltest/analyze.html](https://www.ssllabs.com/ssltest/analyze.html)网站进行检查，但是不支持自定义端口。另外，请记住选中“Do not show the results on the boards”复选框，否则您的系统将在“Recently Seen”列表中可见。

如果您运行的本地服务器没有与公共Internet的连接，则可以使用openssl工具来验证您的证书。

执行以下操作以验证证书是否随链安装（注意将vault.domain.com改为您自己的域名）：

```php
openssl s_client -showcerts -connect vault.domain.com:443

# 或者您使用的其他端口，比如7070
openssl s_client -showcerts -connect vault.domain.com:7070
```

输出的开头应类似于以下内容（使用Let's Encrypt证书）：

```php
CONNECTED(00000003)
depth=2 O = Digital Signature Trust Co., CN = DST Root CA X3
verify return:1
depth=1 C = US, O = Let's Encrypt, CN = Let's Encrypt Authority X3
verify return:1
depth=0 CN = vault.domain.com
verify return:1
```

有3个不同深度（请注意，深度从0开始）级别的验证。在接下来的输出中，您应该看到来自Let's Encryptbase的使用base64编码的证书信息。

