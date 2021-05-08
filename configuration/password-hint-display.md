# 13.显示密码提示

{% hint style="success" %}
对应的[页面地址](https://github.com/dani-garcia/vaultwarden/wiki/Password-hint-display)
{% endhint %}

通常，密码提示是通过电子邮件发送的。但是，由于 vaultwarden 是为小型或个人部署而设计的，所以密码提示在密码提示页面上也是可用的（\[**译者注**\]：我测试目前这个配置并不起能作用），因此您不需要非得配置电子邮件服务。如果要禁用此功能，可以使用 `SHOW_PASSWORD_HINT` 变量：

```python
docker run -d --name vaultwarden \
  -e SHOW_PASSWORD_HINT=false \
  -v /vw-data/:/data/ \
  -p 80:80 \
  vaultwarden/server:latest
```

