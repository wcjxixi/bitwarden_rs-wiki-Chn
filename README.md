# Vaultwarden Wiki 中文版

{% hint style="success" %}
[2021-04-27](https://github.com/dani-garcia/vaultwarden/releases/tag/1.21.0)：bitwarden\_rs 项目更名为 vaultwarden
{% endhint %}

这里是 [vaultwarden Wiki](https://github.com/dani-garcia/vaultwarden/wiki)（以前叫 bitwarden\_rs）的中文翻译版。

原文有太多口语化内容，翻译起来比较费脑，这里我尽力翻译准确并使之不那么生硬。

译者：[@wcjxixi](mailto:wcjxixi@gmail.com)

致谢 [Google Translate](https://translate.google.com/) 以及 [DeepL](https://www.deepl.com/) ！

{% hint style="warning" %}
个人能力有限，具体请以 [vaultwarden Wiki](https://github.com/dani-garcia/vaultwarden/wiki) 官方页面为准。使用本手册所产生的一切后果，与 @wcjxixi 无关。Use at your own risk！！！
{% endhint %}

## vaultwarden 是什么 <a id="what-is-vaultwarden"></a>

vaultwarden 是一个用于本地搭建 Bitwarden 服务器的第三方 Docker 项目。兼容 Bitwarden 官方客户端，仅在部署的时候使用 vaultwarden 镜像，桌面端、移动端、浏览器扩展等客户端均使用 Bitwarden 官方的客户端。

vaultwarden 很轻量，对于不希望使用官方的占用大量资源的自托管部署而言，它是理想的选择。

## vaultwarden 与 Bitwarden 官方版的区别 <a id="difference-between-vaultwarden-and-official-bitwarden"></a>

* 除不支持官方企业版的部分功能（如目录同步、SSO、群组、自定义角色以及基于企业组织层面的 Duo  Security 两步登录）外，其他大部分功能均**免费**支持。并跟随官方版本保持及时更新。
* vaultwarden 比官方版更轻量。官方版使用 .Net 开发，使用 MSSQL 数据库，要求至少 2GB 内存；vaultwarden 使用 Rust 编写，改用 SQLite 数据库（现在也支持 MySQL 和 PostgreSQL），运行时只需要 10M 内存，可以说对硬件基本没有要求。

