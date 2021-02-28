# 1.总览

{% hint style="success" %}
对应的[页面地址](https://github.com/dani-garcia/bitwarden_rs/wiki/Configuration-overview)
{% endhint %}

\[**译者注**\]：

1. 某些设置只能通过环境变量配置，如管理页面中 Read-Only Config 部分的配置项目（如日志记录）
2. 通过管理页面修改配置后马上生效
3. 若直接修改 config.json 文件，则需要重启 bitwarden 才能生效（因为只有启动时才会读取 config.json 文件）

## 配置方式 <a id="configuration-methods"></a>

在 bitwarden\_rs 中，您可以通过环境变量或[管理页面](enabling-admin-page.md)（将设置写入数据目录下的 `config.json` 文件中）执行配置。但请务必注意，`config.json` 文件中的每个设置都会覆盖相应的环境变量设置。例如，如果您设置环境变量为 `DOMAIN=https://bitwarden.example.com`，但您的 `config.json` 文件中已经包含 `"domain": "https://bw.example.com"`，则 bitwarden\_rs 将基于配置文件中的内容生成各种链接（`https://bw.example.com`）。

造成这种混乱的一个常见原因就是因为启用了管理页面（创建了 `config.json` 文件），通过管理页面更改了某些设置（在 `config.json` 文件中设置相应的值），然后又尝试通过环境变量更改这些设置（此操作无效，因为 `config.json` 覆盖环境变量）。为避免这种混乱，强烈建议您采用其中一种配置方式。也就是说，完全通过环境变量或完全通过 `config.json`（无论是使用管理页面还是直接编辑 `config.json`）进行配置。

请注意，管理页面中的 `Read-Only Config` 部分下的配置设置只能通过环境变量来设置，所以你必须重启 bitwarden\_rs 才能使对它们的更改生效。如果你将这些环境变量保存在一个名为 `.env` 的文件中，你可以按照以下方式加载它们：

* 如果是独立版的 bitwarden\_rs，将 `.env` 放到当前工作目录中，bitwarden\_rs 会在启动时尝试加载这个文件。
* 对于 Docker，使用 `docker run --env-file <env-file> ...`（让 Docker 加载 `.env` 文件）或 `docker run -v /path/to/.env:/.env`（让 bitwarden\_rs 从容器内部加载 `.env` 文件）。
* 对于 Docker Compose，使用 [`env_file`](https://docs.docker.com/compose/environment-variables/#the-env_file-configuration-option) 指令。

## 配置选项 <a id="configuration-options"></a>

可以在以下位置找到可设置的环境变量列表：[https://github.com/dani-garcia/bitwarden\_rs/blob/master/.env.template](https://github.com/dani-garcia/bitwarden_rs/blob/master/.env.template)

如果您启用了[管理页面](enabling-admin-page.md)，则管理页面也会显示配置选项的完整列表。

倘若有错误或遗漏，源代码在这里：[https://github.com/dani-garcia/bitwarden\_rs/blob/master/src/config.rs](https://github.com/dani-garcia/bitwarden_rs/blob/master/src/config.rs%20)（检索 `make_config! {`）

或者，如果您的（基于 Chromium）浏览器支持文本片段，则可以直接参考此链接：[https://github.com/dani-garcia/bitwarden\_rs/blob/master/src/config.rs\#LC290:~:text=make\_config!%20%7B,-folders](https://github.com/dani-garcia/bitwarden_rs/blob/master/src/config.rs#LC290:~:text=make_config!%20%7B,-folders)

## 设置域名 URL <a id="setting-the-domain-url"></a>

确保将 `DOMAIN` 环境变量（或配置文件中的 `domain` 选项）设置为用于访问 bitwarden\_rs 实例的 URL。如果不这样做，可能会出现莫名其妙的功能性问题。一些示例：

* `https://bitwarden.example.com`
* `https://bitwarden.example.com:8443`（非默认端口）
* `https://host.example.com/bitwarden`（[子目录托管](using-an-alternate-base-dir-subdir-subpath.md) -尽可能避免 URL 重写）

