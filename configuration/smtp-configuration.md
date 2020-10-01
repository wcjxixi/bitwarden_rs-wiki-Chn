# 13.配置 SMTP

{% hint style="success" %}
对应的[页面地址](https://github.com/dani-garcia/bitwarden_rs/wiki/SMTP-configuration)
{% endhint %}

您可以配置 bitwarden\_rs 以通过 SMTP 代理来发送电子邮件：

```php
docker run -d --name bitwarden \
  -e SMTP_HOST=<smtp.domain.tld> \
  -e SMTP_FROM=<bitwarden@domain.tld> \
  -e SMTP_PORT=587 \
  -e SMTP_SSL=true \
  -e SMTP_USERNAME=<username> \
  -e SMTP_PASSWORD=<password> \
  -v /bw-data/:/data/ \
  -p 80:80 \
  bitwardenrs/server:latest
```

当`SMTP_SSL`设置`true`为时（这是默认值），将仅接受 TLSv1.1 和 TLSv1.2 协议，并且`SMTP_PORT`默认为`587`。如果设置为`false`，`SMTP_PORT`则默认设置为`25`并将尝试加密（2020 年 3 月 12 日之前的代码不会尝试加密）。这可能是非常不安全的，仅在您知道您在做什么时才使用此设置。要以显式模式运行 SMTP，请将`SMTP_EXPLICIT_TLS`设置为`true`。

请注意，如果启用了 SMTP 和邀请，邀请将通过电子邮件发送给新用户。您必须使用 bitwarden\_rs 实例的基础 URL 来设置`DOMAIN`配置项，以生成正确的邀请链接：

```php
docker run -d --name bitwarden \
...
-e DOMAIN=https://vault.example.com \
...
```

用户邀请链接有效期为 5 天，此后需要重新发送邀请。

