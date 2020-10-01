# 2.禁用新用户注册

{% hint style="success" %}
对应的[页面地址](https://github.com/dani-garcia/bitwarden_rs/wiki/Disable-registration-of-new-users)
{% endhint %}

默认情况下，新用户可以注册，如果要禁用该功能，请将`SIGNUPS_ALLOWED`环境变量设置为`false`：

```php
docker run -d --name bitwarden \
  -e SIGNUPS_ALLOWED=false \
  -v /bw-data/:/data/ \
  -p 80:80 \
  bitwardenrs/server:latest
```

注意：虽然用户不能自己注册，但仍然可以由已经注册的用户邀请注册。如果您还想禁用邀请，请参阅[禁用邀请](disable-invitations.md)。

您还可以禁用注册功能，但某些域的电子邮件地址除外。 例如：

* `SIGNUPS_DOMAINS_WHITELIST=example.com` （单个域）
* `SIGNUPS_DOMAINS_WHITELIST=example.com,example.net,example.org` （多个域）

您仍然需要设置`SIGNUPS_ALLOWED=false`。另外，在[＃728](https://github.com/dani-garcia/bitwarden_rs/pull/728) 查看注意事项。

