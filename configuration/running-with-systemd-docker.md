# 19.使用systemd-docker运行

### ！！！设置环境变量上面一段翻译很不完整！！！

{% hint style="success" %}
对应的[页面地址](https://github.com/dani-garcia/bitwarden_rs/wiki/Running-with-systemd-docker)
{% endhint %}

若你喜欢，以下介绍内容还可以用systemd管理Docker容器的生命周期。

首先，使用系统包管理器安装`systemd-docker`包。这是一个包装程序，可以让docker与systemd更完美地集成。

有关完整介绍和配置选项，请参见[GitHub知识库](https://github.com/ibuildthecloud/systemd-docker)。

使用你喜欢的编辑器以root身份创建`/etc/systemd/system/bitwarden.service`文件并使用以下内容：

```php
[Unit]
Description=Bitwarden
After=docker.service
Requires=docker.service

[Service]
TimeoutStartSec=0
ExecStartPre=/usr/bin/docker pull bitwardenrs/server:latest
ExecStart=/usr/bin/systemd-docker --cgroups name=systemd --env run \
  -p 8080:80 \
  -p 8081:3012 \
  -v /opt/bw-data:/data/ \
  --rm --name %n bitwardenrs/server:latest
Restart=always
RestartSec=10s
Type=notify
NotifyAccess=all

[Install]
WantedBy=multi-user.target
```

根据需要调整以上示例。特别要注意`-p`和`-v`选项，因为它们控制容器和主机之间的端口和卷绑定。

对上述选项的解释：

* `TimeoutStartSec`的值0：等待默认启动时间后，停止systemd失败。这要求`ExecStartPre`中的`docker pull`需要一段时间来完成。
* `ExecStartPre`：在运行之前拉取docker标签。
* `Type`的值`notify`：告诉systemd，它期望从已准备就绪的服务中获取通知。
* `NotifyAccess`的值`all`：是由`systemd-docker`请求的。

### 设置环境变量

可以通过两种方式在单元文件中直接指定环境变量：

* 在`[Service]`块中使用`Environment`指令。
* 使用`docker`的`-e`选项。此时，您可以省略上面示例中显示的`--env`选项。

要验证是否正确设置了环境变量，请检查`systemctl show bitwarden.service` 的输出中是否存在`Environment`行。

也可以在单元文件中使用`EnvironmentFile`指令将环境变量存储在单独的文件中。在这种情况下，请在docker命令行中设置`--env`选项，如上面示例中所示，否则将不会处理环境文件。

Systemd可以获取以下格式的文件：

```text
Key="Value"
```

您可以在此[环境示例模版](https://github.com/dani-garcia/bitwarden_rs/blob/21325b7523a68ab3ae8d435ab5b73176db6155ff/.env.template)中找到更多关于环境设置和语法的说明。

但是，systemd项目并没有规定该文件的存储位置。有关此文件的最佳存储位置，请查阅发行版文档。例如，基于RedHat的发行版通常将这些文件放在`/etc/sysconfig/`中。

如果你不确定，只需使用root权限在`/etc/`中创建一个文件，比如`/etc/bitwarden.service.conf`。

在您的单元文件中，在`[Service]`块中添加`EnvironmentFile`指令，该值是上面创建的文件的完整路径。例如：

```php
[Unit]
Description=Bitwarden
After=docker.service
Requires=docker.service

[Service]
EnvironmentFile=/etc/bitwarden.service.conf
TimeoutStartSec=0
-snip-
```

### 运行服务

完成上述安装和配置后，请使用`sudo systemctl daemon-reload`命令重新加载systemd 。然后，使用`sudo systemctl start bitwarden`命令启动Bitwarden服务。

要使服务跟随系统启动，请使用`sudo systemctl enable bitwarden`。

验证容器是否已经启动`systemctl status bitwarden`。

如果在启动服务时遇到`json: cannot unmarshal object into Go value of type string`错误，则应使用最新版本的Go自己编译systemd-docker二进制文件，请参阅此[话题](https://github.com/ibuildthecloud/systemd-docker/issues/50)。

