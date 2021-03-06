# 主页

{% hint style="success" %}
对应的[页面地址](https://github.com/dani-garcia/vaultwarden/wiki)
{% endhint %}

vaultwarden 是一个使用 Rust 编写的非官方 Bitwarden 服务器实现，它与[官方 Bitwarden 客户端](https://bitwarden.com/download/)兼容，对于不希望使用官方的占用大量资源的自托管部署而言，它是理想的选择。

vaultwarden 面向个人、家庭和小型组织。尽管开发主要对大型组织有用的功能（例如，单点登录、目录同步等）并不是优先事项，但欢迎实现此类功能的高质量 PR。

## 支持的特性 <a id="supported-features"></a>

vaultwarden 实现大多数功能所需的 Bitwarden API，其中包括：

* 网页界面（等效于 [https://vault.bitwarden.com/](https://vault.bitwarden.com/%29)）
* 个人密码库支持
* [组织](https://help.bitwarden.in/getting-started/getting-started-with-organizations)密码库支持
* [密码共享](https://help.bitwarden.in/organizations/sharing)和[访问控制](https://help.bitwarden.in/organizations/user-types-and-access-control)
* [集合](https://help.bitwarden.in/organizations/collections)
* [文件附件](https://help.bitwarden.in/your-vault/file-attachments)
* [文件夹](https://help.bitwarden.in/your-vault/folders)
* [收藏](https://help.bitwarden.in/your-vault/favorites)
* [网站图标](https://help.bitwarden.in/security/privacy-when-using-website-icons)
* [Bitwarden 验证器（TOTP）](https://help.bitwarden.in/your-vault/bitwarden-authenticator-totp)
* [Bitwarden Send](https://help.bitwarden.in/bitwarden-send/about-send)
* 用于桌面端/浏览器客户端/扩展的[实时同步](https://bitwarden.com/blog/post/live-sync/)（仅 WebSocket）
* [回收站](https://help.bitwarden.in/your-vault/vault-items#items-in-the-trash)（软删除）
* [电子邮件](https://help.bitwarden.in/two-step-login/two-step-login-via-email)、[Duo](https://help.bitwarden.in/two-step-login/two-step-login-via-duo)、[Yubikey](https://help.bitwarden.in/two-step-login/two-step-login-via-yubikey) 和 [FIDO U2F](https://help.bitwarden.in/two-step-login/two-step-login-via-fido-u2f) 方式的两步登录
* [目录连接器](https://help.bitwarden.in/directory-connector/about-directory-connector)支持（基本实现，但不支持群组）
* 某些企业策略：
  * [主密码](https://help.bitwarden.in/organizations/enterprise-policies#master-password)
  * [密码生成器](https://help.bitwarden.in/organizations/enterprise-policies#password-generator)
  * [个人所有权](https://help.bitwarden.in/organizations/enterprise-policies#personal-ownership)
  * [禁用 Send](https://help.bitwarden.in/organizations/enterprise-policies#disable-send)

## 缺少的特性 <a id="missing-features"></a>

话题 [\#246](https://github.com/dani-garcia/vaultwarden/issues/246) 包含了完整的功能请求列表，既有官方服务器中缺少的功能，也有 vaultwarden 中特有的增强功能。

为了与官方服务器做简单的比较，本章节总结了官方服务器中已经实现但在 vaultwarden 中目前还没有实现的功能。

在时间允许的情况下，可能会添加的功能（欢迎贡献）：

* [紧急访问](https://help.bitwarden.in/security/emergency-access)
* [Bitwarden 公共 API](https://help.bitwarden.in/organizations/bitwarden-public-api)
* [事件日志](https://help.bitwarden.in/organizations/event-logs)
* 用于移动客户端（Android/iOS）的[实时同步](https://bitwarden.com/blog/post/live-sync/)（推送通知）
* 某些企业策略：
  * [两步登录](https://help.bitwarden.in/organizations/enterprise-policies#two-step-login)（[\#981](https://github.com/dani-garcia/bitwarden_rs/issues/981)）
  * [单一组织](https://help.bitwarden.in/organizations/enterprise-policies#single-organization)

除非做出贡献，否则可能不会添加的功能：

* [单点登录（SSO）](https://help.bitwarden.in/login-with-sso/about-login-with-sso)
* [群组](https://help.bitwarden.in/organizations/groups)
* [自定义角色](https://help.bitwarden.in/organizations/user-types-and-access-control#custom-role)

## 保持联系 <a id="get-in-touch"></a>

要提出问题、提供建议、请求新功能或获得有关配置或安装软件的帮助，请[使用论坛](https://vaultwarden.discourse.group/)。

如果您发现任何与 vaultwarden 本身有关的 bug 或崩溃，请[创建一个话题](https://github.com/dani-garcia/vaultwarden/issues)。并先确保不存在任何类似的话题！

我们通常在 [\#vaultwarden:matrix.org](https://matrix.to/#/#vaultwarden:matrix.org) 房间闲逛，如果您喜欢聊天，欢迎随时加入我们！

