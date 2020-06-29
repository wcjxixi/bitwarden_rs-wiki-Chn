# 5.禁用管理令牌

{% hint style="success" %}
对应的[页面地址](https://github.com/dani-garcia/bitwarden_rs/wiki/Disable-admin-token)
{% endhint %}

**重要提示**：任何人都将可以访问您的管理页面。

如果您要使用其他方法对`/admin`页面进行身份验证，则可以将`DISABLE_ADMIN_TOKEN`变量设置为true。这将禁用内置的`ADMIN_TOKEN`身份验证功能，但同时会启用管理面板。有权访问URL的任何人都可以访问管理面板。您需要采取额外的步骤来（包括外部和本地）来保护她。

```php
docker run -d --name bitwarden \
  -e DISABLE_ADMIN_TOKEN=true \
  -v /bw-data/:/data/ \
  -p 80:80 \
  bitwardenrs/server:latest
```

