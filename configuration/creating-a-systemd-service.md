# 18.创建为系统服务

{% hint style="success" %}
对应的[页面地址](https://github.com/dani-garcia/bitwarden_rs/wiki/Setup-as-a-systemd-service)
{% endhint %}

这里的文档要求您已经[编译了bitwarden\_rs二进制文件](https://github.com/dani-garcia/bitwarden_rs/wiki/Building-binary)。如果生成了docker映像，则需要查看[使用systemd-docker运行](running-with-systemd-docker.md)。

### 设置

要确保bitwarden\_rs在系统启动的时候启动并使用systemd的其他功能（例如，隔离、日志记录等），需要一个`.service`文件。以下是一些可用的：

```php
[Unit]
Description=Bitwarden Server (Rust Edition)
Documentation=https://github.com/dani-garcia/bitwarden_rs
# If you use a database like mariadb,mysql or postgresql, 
# you have to add them like the following and uncomment them 
# by removing the `# ` before it. This makes sure that your 
# database server is started before bitwarden_rs ("After") and has 
# started successfully before starting bitwarden_rs ("Requires").

# Only sqlite
After=network.target

# MariaDB
# After=network.target mariadb.service
# Requires=mariadb.service

# Mysql
# After=network.target mysqld.service
# Requires=mysqld.service

# PostgreSQL
# After=network.target postgresql.service
# Requires=postgresql.service


[Service]
# The user/group bitwarden_rs is run under. the working directory (see below) should allow write and read access to this user/group
User=bitwarden_rs
Group=bitwarden_rs
# The location of the .env file for configuration
EnvironmentFile=/etc/bitwarden_rs.env
# The location of the compiled binary
ExecStart=/usr/bin/bitwarden_rs
# Set reasonable connection and process limits
LimitNOFILE=1048576
LimitNPROC=64
# Isolate bitwarden_rs from the rest of the system
PrivateTmp=true
PrivateDevices=true
ProtectHome=true
ProtectSystem=strict
# Only allow writes to the following directory and set it to the working directory (user and password data are stored here)
WorkingDirectory=/var/lib/bitwarden_rs
ReadWriteDirectories=/var/lib/bitwarden_rs
# Allow bitwarden_rs to bind ports in the range of 0-1024
AmbientCapabilities=CAP_NET_BIND_SERVICE

[Install]
WantedBy=multi-user.target
```

更改所有路径以匹配您的安装（`WorkingDirectory`与`ReadWriteDirectory`应相同），将`bitwarden_rs.service`文件命名并将其放入`/etc/systemd/system`。

如果必须更改现有的systemd文件（您安装的软件包提供给您的），则可以使用下面的命令来添加更改：

```php
$ sudo systemctl edit bitwarden_rs.service
```

请运行以下命令，以使systemd知道您的新文件或您所做的任何更改：

```php
$ sudo systemctl daemon-reload
```

### 用法

要启动此“服务”，请运行：

```php
$ sudo systemctl start bitwarden_rs.service
```

要启用自动启动，请运行：

```php
$ sudo systemctl enable bitwarden_rs.service
```

同样的，你可以使用`stop`，`restart`和`disable`。

#### 更新bitwarden\_rs

编译新版本的bitwarden\_rs之后，您可以复制已编译的（新）二进制文件并替换现有的（旧）二进制文件，然后重新启动服务：

```php
$ sudo systemctl restart bitwarden_rs.service
```

#### 卸载bitwarden\_rs

在执行其他操作之前，应停止并禁用该服务：

```php
$ sudo systemctl disable --now bitwarden_rs.service
```

然后，您可以删除`.env`二进制文件、web-vault文件夹（如果已安装）和用户数据（如果需要）。请记住，还要删除专门创建的用户、组和防火墙规则（如果需要）和systemd文件。

删除systemd文件后，您应该通过以下方式使systemd意识到这一点：

```php
$ sudo systemctl daemon-reload
```

#### 查看日志和状态

如果要查看日志记录输出，请运行：

```php
$ journalctl -u bitwarden_rs.service
```

或查看更简洁的服务状态，请运行：

```php
$ systemctl status bitwarden_rs.service
```

### 故障排除

#### 旧版systemd的沙盒选项

在RHEL 7（和debian 8）中，使用的systemd不支持某些隔离选项（[\#445](https://github.com/dani-garcia/bitwarden_rs/issues/445)，[\#363](https://github.com/dani-garcia/bitwarden_rs/issues/363)）。这可能导致出现如下错误之一：

```php
Failed at step NAMESPACE spawning /home/bitwarden_rs/bitwarden_rs: Permission denied
```

或者：

```php
Failed to parse protect system value
```

要解决这一点，你可以将含有 `PrivateTmp`、`PrivateDevices`、`ProtectHome`、`ProtectSystem`和`ReadWriteDirectories`的部分或全部行前面放置\#符号来注释掉。尽管将所有这些注释掉可能会起作用，但不建议这样做，因为这些都是很好的安全措施。要查看您的systemd支持哪些选项，请运行以下命令来查看输出：

```php
$ systemctl --version
```

检查您的systemd版本并与[systemd/NEWS.md](https://github.com/systemd/systemd/blob/master/NEWS)进行比较。

编辑`.service`文件后，请不要忘记在启动（或重启）服务之前运行如下命令：

```php
$ sudo systemctl daemon-reload
```

### 更多信息

有关.service文件的更多信息，请参阅[systemd.service](https://www.freedesktop.org/software/systemd/man/systemd.service.html)（用于安全性配置）和[systemd.exec](https://www.freedesktop.org/software/systemd/man/systemd.exec.html)手册页。

