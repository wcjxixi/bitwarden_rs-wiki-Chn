# 7.增强指南

{% hint style="success" %}
对应的[页面地址](https://github.com/dani-garcia/bitwarden_rs/wiki/Hardening-Guide)
{% endhint %}

### 禁用注册和（可选）邀请

默认情况下，bitwarden\_rs允许任何匿名用户在不首先被邀请的情况下在服务器上注册新帐户。这是在服务器上创建第一个用户所必需的，但建议您在管理面板中（如果启用了管理面板的话）或[使用环境变量](../configuration/disable-registration-of-new-users.md)将其禁用，以防止攻击者在bitwarden\_rs服务器上创建帐户。

bitwarden\_rs还允许注册用户邀请其他新用户在服务器上创建帐户并加入其组织。只要您信任用户，这不会带来直接风险，但是可以在管理面板或[环境变量](../configuration/disable-invitations.md)中将其禁用。

### 启用HTTPS

#### TLS强化

### 禁用显示密码提示

bitwarden\_rs在登录页面上显示密码提示，以适应未配置SMTP的小型/本地部署，攻击者可能会滥用这些密码来促进针对服务器用户的猜测密码攻击。可以在管理面板中通过取消选中该`Show password hints`选项或使用[环境变量](../configuration/password-hint-display.md)来禁用它。

### 禁用图标提取

### SMTP强化

### 暴力破解

当不使用双重身份验证时，（理论上）可以暴力破解用户密码，从而获得对其帐户的访问权限。缓解此问题的一种相对简单的方法是设置fail2ban，设置后，将在过多的失败登录尝试后阻止访问者的IP地址。但是：在多个反向代理（例如cloudflare）后面使用此功能时，应格外注意。请参阅：[Fail2Ban设置](https://github.com/dani-garcia/bitwarden_rs/wiki/Fail2Ban-Setup)

