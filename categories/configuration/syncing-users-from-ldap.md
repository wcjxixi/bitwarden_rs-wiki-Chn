# 从LDAP同步用户

{% hint style="success" %}
对应的[页面地址](https://github.com/dani-garcia/bitwarden_rs/wiki/Syncing-users-from-LDAP)
{% endhint %}

LDAP集成使用小型服务来执行，该小型服务用于查询LDAP并邀请用户加入您的Bitwarden实例。该服务的名称被非正式地命名为[bitwarden\_rs\_ldap](https://github.com/ViViDboarder/bitwarden_rs_ldap)。

它尚未以二进制形式分发，但是有可用的Docker映像[vividboarder/bitwarden\_rs\_ldap](https://hub.docker.com/r/vividboarder/bitwarden_rs_ldap)。

部署之前，您必须[启用bitwarden\_rs管理页面](https://github.com/dani-garcia/bitwarden_rs/wiki/Enabling-admin-page)。这将启用LDAP同步服务API，以用于邀请用户。配置LDAP同步服务时，将使用您设置的`ADMIN_TOKEN`。您还必须确保**不禁用**邀请功能。请再次检查环境变量`INVITATIONS_ALLOWED`是否未设置为`false`。

另外也建议在你的Bitwarden实例中[启用发送电子邮件功能](smtp-configuration.md)，以便通知您的用户注册他们的账户。如果不这样做，虽然他们也可以使用他自己的LDAP电子邮件地址注册，但是您必须自己通知他们。

完成这些步骤后，您就可以配置和部署LDAP同步服务了。最新的说明文档可在它的[自述文件](https://github.com/ViViDboarder/bitwarden_rs_ldap)中找到，此文档也涉及了使用bitwarden\_rs实例、LDAP实例以及您用于查找用户的LDAP查询的连接信息，来创建`config.toml`文件的说明。

