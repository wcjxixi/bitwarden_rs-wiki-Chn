# \*6.使私有 CA 和自签名证书兼容 Chrome

{% hint style="success" %}
对应的[页面地址](https://github.com/dani-garcia/vaultwarden/wiki/Private-CA-and-self-signed-certs-that-work-with-Chrome)
{% endhint %}

{% hint style="warning" %}
此方法仅用于测试和开发。绝大多数用户不应该使用这种方法，因为它需要在你的每台设备上加载一个证书，这既容易出错，又需要后期的维护。相反，把精力集中在通过 [Let's Encrypt](https://letsencrypt.org/getting-started/) 获取的真实证书上。如果你的 vaultwarden 实例不处于公共互联网中，此方法甚至也可以工作（[示例](../deployment/https/running-a-private-vaultwarden-instance-with-lets-encrypt-certs.md)）。

此方法不受支持。请不要开启 GitHub 问题，也不要在讨论区发帖询问如何让这个方法能正常工作。
{% endhint %}

为了使 bitwarden 能够正常使用自签名证书，Chrome 需要该证书在证书的备用名称字段中包含域名。

创建一个 CA 密钥（您自己的小型本地证书颁发机构）：

```python
openssl genpkey -algorithm RSA -aes128 -out private-ca.key -outform PEM -pkeyopt rsa_keygen_bits:2048
```

注意：您也可以使用较旧的 `-des3` 来代替 `-aes128`。

创建一个 CA 证书：

```python
openssl req -x509 -new -nodes -sha256 -days 3650 -key private-ca.key -out self-signed-ca-cert.crt
```

注意：`-nodes` 参数将阻止在测试/安全环境中为私钥（密钥对）设置密码短语，否则每次启动/重启服务器时都必须输入密码短语。

创建一个 vaultwarden 密钥：

```python
openssl genpkey -algorithm RSA -out vaultwarden.key -outform PEM -pkeyopt rsa_keygen_bits:2048
```

创建 vaultwarden 证书请求文件：

```python
openssl req -new -key vaultwarden.key -out vaultwarden.csr
```

创建具有以下内容的文本文件 `vaultwarden.ext`，将域名更改为您设置的域名：

```python
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
extendedKeyUsage = serverAuth
subjectAltName = @alt_names

[alt_names]
DNS.1 = vaultwarden.local
DNS.2 = www.vaultwarden.local
```

创建从根 CA 签名的 vaultwarden 证书：

```python
openssl x509 -req -in vaultwarden.csr -CA self-signed-ca-cert.crt -CAkey private-ca.key -CAcreateserial -out vaultwarden.crt -days 365 -sha256 -extfile vaultwarden.ext
```

注意：自 2019 年 4 月起，iOS 13+ 和 macOS 15+ 的服务器证书的到期日不能超过 825，并且必须包含 ExtendedKeyUsage 扩展。详见 [https://support.apple.com/zh-cn/HT210176](https://support.apple.com/en-us/HT210176)。

将根证书和 vaultwarden 证书添加到客户端计算机。

更多参考，请参阅这里：[https://deliciousbrains.com/ssl-certificate-authority-for-local-https-development/](https://deliciousbrains.com/ssl-certificate-authority-for-local-https-development/)。

