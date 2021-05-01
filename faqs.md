# FAQ

{% hint style="success" %}
对应的[页面地址](https://github.com/dani-garcia/bitwarden_rs/FAQs)
{% endhint %}

## Bitwarden\_RS 是否与 Bitwarden 项目或 8bit Solutions LLC 有关？ <a id="is-bitwarden_rs-associated-with-the-bitwarden-project-or-8-bit-solutions-llc"></a>

简短的回答，**没有**。两个项目的开发人员之间有时会有联系，但没有合作。除此之外，Bitwarden\_RS 项目仅使用 8bit Solutions LLC 提供的 Web Vault 并打了一些补丁，以使其与我们的实现兼容。

## Bitwarden\_RS 能否连接到 Oracle MySQL V8.x 数据库？ <a id="can-bitwarden_rs-connect-to-an-oracle-mysql-v-8-x-database"></a>

使用 Oracle MySQL v8.x，当你试图启动 Bitwarden\_RS 时，可能会出现以下警告：

```python
[bitwarden_rs::util][WARN] Can't connect to database, retrying: DieselConError.
[CAUSE] BadConnection(
    "Authentication plugin \'caching_sha2_password\' cannot be loaded: /usr/lib/x86_64-linux-gnu/mariadb18/plugin/caching_sha2_password.so: cannot open shared object file: No such file or directory",
)
```

Oracle MySQL v8.x 默认使用更安全的密码散列方法，这很好，但目前不被我们的构建所支持。 你需要以一种特定的方式创建 Bitwarden\_RS 用户，以便它使用旧的本地密码散列。

```python
-- Use this on MySQLv8 installations
CREATE USER 'bitwarden_rs'@'localhost' IDENTIFIED WITH mysql_native_password BY 'yourpassword';
```

如果您已经创建了用户，并且只想更改散列方法，请使用以下命令：

```python
-- Change password type from caching_sha2_password to native
ALTER USER 'bitwarden_rs'@'localhost' IDENTIFIED WITH mysql_native_password BY 'yourpassword';
```

另外可参阅：[使用 MariaDB - 创建数据库和用户](configuration/database/using-the-mariadb-mysql-backend.md#create-database-and-user)

## 我的客户端（桌面端、移动端、网页端）无法正常工作，我无法登录或它提示证书无效。 <a id="my-client-desktop-mobile-web-does-not-work-i-can-not-login-or-it-complains-about-invalid-certificates"></a>

要正常工作没有任何问题，Bitwarden 客户端需要一个安全的连接。虽然有些客户端可以在没有安全连接的情况下工作，但我们并不推荐这样做。

大多数时候，当人们在使用证书时仍然有问题，他们使用的是所谓的自签名证书。虽然这些证书可以提供安全连接，但一些平台不允许或不支持这样做。

我们建议使用像 Let's Encrypt 这样的服务来提供一个有效的、被大多数设备默认接受的证书。 请参阅以下页面：

* [启用 HTTPS](deployment/https/enabling-https.md)
* [使用 Let's Encrypt 证书运行私有 bitwarden\_rs 实例](deployment/https/running-a-private-bitwarden_rs-instance-with-lets-encrypt-certs.md)

## 为什么我的所有密码库项目都看不到图标？ <a id="why-do-i-see-no-icons-for-all-my-vault-items"></a>

没有显示图标的原因有很多。如果只是几个密码库项目，可能是我们无法提取它。有些网站启用了一些保护措施，导致我们的实施失败。他们中的大多数需要 Javascript 才能工作。

这也可能是 Bitwarden\_RS 服务器无法访问互联网或未解决 DNS 查询。你可以检查 `/admin/diagnostics` 页面，看看你是否能解决 DNS 查询以及是否有连接到互联网。如果都没问题，也有可能是防火墙或外发互联网代理阻止了这些请求。

## 可以将 Bitwarden\_RS 作为 Azure WebApp 运行吗？ <a id="can-i-run-bitwarden_rs-as-an-azure-webapp"></a>

不幸的是，Azure WebApp 使用 CIFS/Samba 作为卷存储，不支持锁定。这导致不能使用 SQLite 数据库文件。 

有两种方法解决：

1. 不要使用 SQLite，而使用 MariaDB/MySQL 或 Posgresql 作为数据库后端。
2. 尝试将 `ENABLE_DB_WAL` 环境变量的值设置为 `false` 以禁用 WAL。这需要在一个新的文件上完成，所以你需要移除之前创建的 `db.sqlite3` 文件，并再次重启 Bitwarden\_RS 应用程序。

## 我在 FAQ 中找不到答案，下一步该怎么做？ <a id="i-did-not-find-my-answer-here-in-the-faq-what-to-do-next"></a>

那么，请尝试在我们精彩的 [Wiki](./) 中搜索和点击。如果这对你没有帮助，请尝试查看 [Github 讨论](https://github.com/dani-garcia/bitwarden_rs/discussions)或 [Vaultwarden 论坛](https://bitwardenrs.discourse.group/)。如果这也没有解决，你可以尝试搜索开放的和已关闭的[话题](https://github.com/dani-garcia/bitwarden_rs/issues)。

如果你仍然没有找到答案，你可以在 [Github 讨论](https://github.com/dani-garcia/bitwarden_rs/discussions)或 [Vaultwarden 论坛](https://bitwardenrs.discourse.group/)上发起一个主题，或者加入我们的[聊天室](https://matrix.to/#/#bitwarden_rs:matrix.org)。

