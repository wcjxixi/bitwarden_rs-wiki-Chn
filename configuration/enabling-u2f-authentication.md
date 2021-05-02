# 7.启用 U2F 身份验证

{% hint style="success" %}
对应的[页面地址](https://github.com/dani-garcia/bitwarden_rs/wiki/Enabling-U2F-authentication)
{% endhint %}

要启用 U2F 身份验证，您必须使用带有效证书（使用内置的 HTTPS 选项或使用反向代理）的 HTTPS 域名访问 vaultwarden。我们建议使用 Let's Encrypt 提供的免费证书。

之后，您需要将 `DOMAIN` 环境变量设置为与访问 vaultwarden 相同的地址：

```python
docker run -d --name vaultwarden \
  -e DOMAIN=https://vw.domain.tld \
  -v /vw-data/:/data/ \
  -p 80:80 \
  vaultwarden/server:latest
```

请注意，该值必须包含 `https://`，并且如果不使用默认的 `443`，在末尾还必须包含一个端口（格式为 `https://vw.domain.tld:port`）。

