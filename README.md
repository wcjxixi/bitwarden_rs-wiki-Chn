# 关于

这里是 [bitwarden\_rs wiki](https://github.com/dani-garcia/bitwarden_rs/wiki) 页面的中文翻译版。原文有太多的口语化内容，翻译起来很费脑，这里我尽力翻译准确并不使之那么生硬。

译者：[@wcjxixi](mailto:wcjxixi@gmail.com)

致谢 [Google Translate](https://translate.google.com/) 以及 [DeepL](https://www.deepl.com/) ！

{% hint style="warning" %}
个人能力有限，具体请以 [bitwarden\_rs](https://github.com/dani-garcia/bitwarden_rs) 官方页面为准。使用本手册所产生的一切后果，与 @wcjxixi 无关。Use at your own risk！！！
{% endhint %}

bitwarden\_rs 与 Bitwarden 官方版的区别：

* 除不支持官方企业版相关功能（如群组控制、企业策略、日志审核、目录同步、以及基于企业组织层面的 DUO  Security 两步登录）外，其他所有功能均免费支持。并跟随官方版本保持及时更新。
* bitwarden\_rs 比官方版更轻量。官方版使用 .Net 开发，使用MSSQL数据库，要求至少 2GB 内存；bitwarden\_rs 版本改用 SQLite 数据库，可以说对硬件基本没有要求。
* 对于 bitwarden\_rs：仅在部署的时候使用 bitwarden\_rs 镜像，桌面端、移动端、浏览器扩展等均使用官方的应用程序。

