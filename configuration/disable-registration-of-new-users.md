# 2.禁用新用户注册

{% hint style="success" %}
对应的[页面地址](https://github.com/dani-garcia/bitwarden_rs/wiki/Disable-registration-of-new-users)
{% endhint %}

默认情况下，新用户可以注册，如果要禁用该功能，请将 `SIGNUPS_ALLOWED` 环境变量设置为 `false`：

```php
docker run -d --name bitwarden \
  -e SIGNUPS_ALLOWED=false \
  -v /bw-data/:/data/ \
  -p 80:80 \
  bitwardenrs/server:latest
```

## 禁用组织邀请 <a id="disabling-organization-invitations"></a>

即使 `SIGNUPS_ALLOWED=false`，作为组织所有者或管理员的现有用户仍然可以邀请新用户。如果你也想禁用这个功能，请参阅[禁用邀请](disable-invitations.md)。

## 将注册限制为某些电子邮件域 <a id="restricting-registrations-to-certain-email-domains"></a>

您可以通过设置 `SIGNUPS_DOMAINS_WHITELIST` 来限制只能某些域的电子邮件地址注册。示例：

* `SIGNUPS_DOMAINS_WHITELIST=example.com` （单个域）
* `SIGNUPS_DOMAINS_WHITELIST=example.com,example.net,example.org` （多个域）

如果设置了 `SIGNUPS_DOMAINS_WHITELIST`，`SIGNUPS_ALLOWED=false`的值将被忽略。

你可能还想设置 `SIGNUPS_VERIFY=true`，以要求新注册的用户在成功登录前进行电子邮件验证。这将防止有人用一个拥有正确域名的假电子邮件地址注册。

## 通过管理页面发出邀请 <a id="invitations-via-the-admin-page"></a>

bitwarden\_rs 管理员可以通过[管理页面](enabling-admin-page.md)邀请任何人，不受以上限制。

