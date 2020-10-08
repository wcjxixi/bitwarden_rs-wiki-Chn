# 16.日志记录

{% hint style="success" %}
对应的[页面地址](https://github.com/dani-garcia/bitwarden_rs/wiki/Logging)
{% endhint %}

## 记录到文件 <a id="logging-to-a-file"></a>

从 1.5.0 版本开始支持记录到文件。您可以使用 `LOG_FILE` 环境变量来指定日志文件的路径：

```python
docker run -d --name bitwarden \
...
  -e LOG_FILE=/data/bitwarden.log \
...
```

请注意，如果您使用的是 docker 镜像，则很可能要使用从主机 OS 挂载的文件路径（例如 data 文件夹）。

## 更改日志级别 <a id="change-the-log-level"></a>

为了减少日志消息的数量，您可以将日志级别设置为“warn”（默认为“info”）。[日志级别](https://docs.rs/log/0.4.7/log/enum.Level.html#variants)可使用 `LOG_LEVEL` 环境变量进行调整，同时还需要设置 `EXTENDED_LOGGING=true`。注意：使用日志级别“warn”或“error”仍然允许 [Fail2Ban](fail2ban-setup.md) 正常工作。

`LOG_LEVEL` 选项包括："trace"，"debug"，"info"，"warn"，"error"以及"off"。

```python
docker run -d --name bitwarden \
...
  -e LOG_LEVEL=warn -e EXTENDED_LOGGING=true \
...
```

