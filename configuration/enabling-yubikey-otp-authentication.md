# 启用Yubikey OPT身份验证

{% hint style="success" %}
对应的[页面地址](https://github.com/dani-garcia/bitwarden_rs/wiki/Enabling-Yubikey-OTP-authentication)
{% endhint %}

要启用YubiKey身份验证，必须设置`YUBICO_CLIENT_ID`和`YUBICO_SECRET_KEY`变量。

如果`YUBICO_SERVER`未指定，它将使用默认的YubiCloud服务器地址。您可以在[这里](https://upgrade.yubico.com/getapikey/)为默认的YubiCloud生成`YUBICO_CLIENT_ID`和`YUBICO_SECRET_KEY`。

备注：

* 为了生成API密钥或在OTP服务器上使用YubiKey，必须对其进行注册。在[YubiKey个性化工具](https://www.yubico.com/products/services-software/personalization-tools/use/)中配置好你的密钥后，您可以在[此处](https://upload.yubico.com/)注册默认服务器。
* 由于上游的问题，服务器版本1.6.0或更早版本的aarch64不支持Yubikey功能（请参阅[＃262](https://github.com/dani-garcia/bitwarden_rs/issues/262)）。

```php
docker run -d --name bitwarden \
  -e YUBICO_CLIENT_ID=12345 \
  -e YUBICO_SECRET_KEY=ABCDEABCDEABCDEABCDE= \
  -v /bw-data/:/data/ \
  -p 80:80 \
  bitwardenrs/server:latest
```

