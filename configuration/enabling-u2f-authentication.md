# 8.启用 U2F 身份认证

{% hint style="success" %}
对应的[页面地址](https://github.com/dani-garcia/bitwarden_rs/wiki/Enabling-U2F-authentication)
{% endhint %}

要启用 U2F 身份认证，您必须使用带有效证书（使用内置的 HTTPS 选项或使用反向代理）的 HTTPS 域名访问 bitwarden\_rs。我们建议使用 Let's Encrypt 提供的免费证书。

之后，您需要将 `DOMAIN` 环境变量设置为与访问 bitwarden\_rs 相同的地址：

```python
docker run -d --name bitwarden \
  -e DOMAIN=https://bw.domain.tld \
  -v /bw-data/:/data/ \
  -p 80:80 \
  bitwardenrs/server:latest
```

请注意，该值必须包含 `https://`，如果不使用 `443`，在末尾还必须包含一个端口（格式为 `https://bw.domain.tld:port`）。

