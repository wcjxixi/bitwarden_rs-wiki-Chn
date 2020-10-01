# 3.私有 CA 和自签名证书兼容 Chrome

{% hint style="success" %}
对应的[页面地址](https://github.com/dani-garcia/bitwarden_rs/wiki/Private-CA-and-self-signed-certs-that-work-with-Chrome)
{% endhint %}

为了使 bitwarden 能够正确使用自签名证书，Chrome 需要该证书在证书的备用名称字段中包含域名。

创建 CA 密钥文件（您自己的小型本地证书颁发机构）：

```python
openssl genpkey -algorithm RSA -aes128 -out private-ca.key -outform PEM -pkeyopt rsa_keygen_bits:2048
```

注意：您也可以使用较旧的 `-des3` 来代替 `-aes128`。

创建 CA 证书文件：

```python
openssl req -x509 -new -nodes -sha256 -days 3650 -key private-ca.key -out self-signed-ca-cert.crt
```

注意：该 `-nodes` 参数将阻止在测试/安全环境中为私钥（密钥对）设置密码短语，否则每次启动/重启服务器时都必须输入密码短语。

创建 bitwarden 密钥文件：

```python
openssl genpkey -algorithm RSA -out bitwarden.key -outform PEM -pkeyopt rsa_keygen_bits:2048
```

创建 bitwarden 证书请求文件：

```python
openssl req -new -key bitwarden.key -out bitwarden.csr
```

 创建具有以下内容的文本文件 `bitwarden.ext`，将域名更改为您自己的。

```python
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
extendedKeyUsage = serverAuth
subjectAltName = @alt_names

[alt_names]
DNS.1 = bitwarden.local
DNS.2 = www.bitwarden.local
```

创建从根 CA 签名的 bitwarden 证书文件：

```python
openssl x509 -req -in bitwarden.csr -CA self-signed-ca-cert.crt -CAkey private-ca.key -CAcreateserial -out bitwarden.crt -days 365 -sha256 -extfile bitwarden.ext
```

注意：自 2019 年 4 月起，iOS 13+ 和 macOS 15+ 的服务器证书的到期日不能超过 825，并且必须包含 ExtendedKeyUsage 扩展。详见 [https://support.apple.com/zh-cn/HT210176](https://support.apple.com/en-us/HT210176)。

将根证书和 bitwarden 证书添加到客户端计算机。

更多参考，请参阅这里：[https://deliciousbrains.com/ssl-certificate-authority-for-local-https-development/](https://deliciousbrains.com/ssl-certificate-authority-for-local-https-development/)。

