# 1.配置概要

{% hint style="success" %}
对应的[页面地址](https://github.com/dani-garcia/bitwarden_rs/wiki/Configuration-overview)
{% endhint %}

## 配置方式 <a id="configuration-methods"></a>

在 bitwarden\_rs 中，您可以通过环境变量或[管理页面](https://github.com/dani-garcia/bitwarden_rs/wiki/Enabling-admin-page)（将设置写入数据目录下的 `config.json` 文件中）执行配置。但请务必注意，`config.json` 文件中的每个设置都会覆盖相应的环境变量设置。例如，如果您设置环境变量为 `DOMAIN=https://bitwarden.example.com`，但您的 `config.json` 文件中已经包含`"domain": "https://bw.example.com"`，则 bitwarden\_rs 将基于配置文件中的内容生成各种链接（这里为 `https://bw.example.com`）。

造成这种混乱的一个常见原因就是因为启用了管理页面（创建了 `config.json` 文件），通过管理页面更改了某些设置（在 `config.json` 文件中设置相应的值），然后又尝试通过环境变量更改这些设置（此操作无效，因为 `config.json` 覆盖环境变量）。为避免这种混乱，强烈建议您采用其中一种配置方式。也就是说，完全通过环境变量或完全通过 `config.json`（无论是使用管理页面还是直接编辑 `config.json`）进行配置。

## 配置选项 <a id="configuration-options"></a>

可以在以下位置找到可设置的环境变量列表：[https://github.com/dani-garcia/bitwarden\_rs/blob/master/.env.template](https://github.com/dani-garcia/bitwarden_rs/blob/master/.env.template)

如果您启用了[管理页面](enabling-admin-page.md)，则该页面也会显示配置选项的完整列表。

倘若有错误或遗漏，源代码在这里：[https://github.com/dani-garcia/bitwarden\_rs/blob/master/src/config.rs](https://github.com/dani-garcia/bitwarden_rs/blob/master/src/config.rs%20)（搜索 `make_config!`）

## 设置域名 URL <a id="setting-the-domain-url"></a>

确保将 `DOMAIN` 环境变量（或配置文件中的 `domain` 选项）设置为用于访问 bitwarden\_rs 实例的 URL。如果不这样做，可能会出现莫名其妙的功能性问题。一些例子：

* `https://bitwarden.example.com`
* `https://bitwarden.example.com:8443`（非默认端口）
* `https://host.example.com/bitwarden`（[子目录托管](using-an-alternate-base-dir-subdir-subpath.md) -尽可能避免 URL 重写）

