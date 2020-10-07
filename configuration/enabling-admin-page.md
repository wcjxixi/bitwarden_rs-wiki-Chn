# 4.启用管理页面

{% hint style="success" %}
对应的[页面地址](https://github.com/dani-garcia/bitwarden_rs/wiki/Enabling-admin-page)
{% endhint %}

**重要说明**：强烈建议在启用此功能之前激活 HTTPS，以避免可能的 [MITM](https://www.jianshu.com/p/a825de42ccbc) 攻击。

该页面允许服务器管理员查看并删除所有已注册的用户。它也允许邀请新用户，即使禁用了注册功能。

要启用管理页面，您需要设置一组身份验证令牌。该令牌可以是任何字符，但建议使用随机生成的长字符串，比如运行 `openssl rand -base64 48` 命令生成的。**将此令牌保存在安全的地方，它是你访问服务器管理区域的密码！**

要设置令牌，请使用 `ADMIN_TOKEN` 变量：

```python
docker run -d --name bitwarden \
  -e ADMIN_TOKEN=some_random_token_as_per_above_explanation \
  -v /bw-data/:/data/ \
  -p 80:80 \
  bitwardenrs/server:latest
```

此后，管理页面将在 `/admin` 子目录中可用。

在管理页面首次保存设置时，将自动在 `DATA_FOLDER` 文件夹中生成 `config.json` 文件。该文件中的值优先于环境变量值。

**注意：**更改 `ADMIN_TOKEN` 后，当前已登录管理页面的人仍可以使用旧的登录令牌[长达 20 分钟时间](https://github.com/dani-garcia/bitwarden_rs/blob/master/src/api/admin.rs#L87)。

