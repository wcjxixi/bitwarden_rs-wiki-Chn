# 15.禁用或覆盖密码库接口托管

{% hint style="success" %}
对应的[页面地址](https://github.com/dani-garcia/bitwarden_rs/wiki/Disabling-or-overriding-the-Vault-interface-hosting)
{% endhint %}

为方便起见，bitwarden\_rs 映像还将托管网页密码库界面的静态文件。您可以通过设置 `WEB_VAULT_ENABLED` 环境变量来完全禁用静态文件的托管。

```python
docker run -d --name bitwarden \
  -e WEB_VAULT_ENABLED=false \
  -v /bw-data/:/data/ \
  -p 80:80 \
  bitwardenrs/server:latest
```

或者，您可以覆盖密码库文件并提供自己的静态文件来进行托管。您可以通过在容器中挂载你自己的文件路径（而不是 `/web-vault` 目录）来实现。只需确保此目录中至少包含 `index.html` 文件即可。

```python
docker run -d --name bitwarden \
  -v /path/to/static/files_directory:/web-vault \
  -v /bw-data/:/data/ \
  -p 80:80 \
  bitwardenrs/server:latest
```

请注意，您还可以通过为 `WEB_VAULT_FOLDER` 环境变量设置路径来更改 bitwarden\_rs 查找静态文件的路径。  


