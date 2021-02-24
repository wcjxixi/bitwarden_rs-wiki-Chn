# =17.Fail2Ban 设置

{% hint style="success" %}
对应的[页面地址](https://github.com/dani-garcia/bitwarden_rs/wiki/Fail2Ban-Setup)
{% endhint %}

设置 Fail2ban 可以阻止攻击者暴力破解您的密码库登录。如果您的实例是公开的，这一点尤其重要。

## 目录 <a id="table-of-contents"></a>

* [预先说明](fail2ban-setup.md#pre-requisite)
* [安装](fail2ban-setup.md#installation)
  * [Debian / Ubuntu / Raspian](fail2ban-setup.md#debian-ubuntu-raspian)
  * [Fedora / Centos](fail2ban-setup.md#fedora-centos)
  * [群晖 DSM](fail2ban-setup.md#synology-dsm)
* [为网页密码库设置](fail2ban-setup.md#setup-for-web-vault)
  * [筛选](fail2ban-setup.md#filter)
  * [Jail](fail2ban-setup.md#jail)
* [为管理页面设置](fail2ban-setup.md#setup-for-admin-page)
  * [筛选](fail2ban-setup.md#filter-1)
  * [Jail](fail2ban-setup.md#jail-1)
* [测试 Fail2Ban](fail2ban-setup.md#testing-fail-2-ban)
* [SELinux 中的问题](fail2ban-setup.md#selinux-problems)

## 预先说明 <a id="pre-requisite"></a>

* 以下示例使用 `vi` 指令编辑。您可以在[这里](https://pc.net/resources/commands/vi)查看它的基本使用方法。当然，您也可以使用您想使用的任何文本编辑器。
* 从 1.5.0 版开始，Bitwarden\_rs 支持记录到文件。请设置[日志记录](logging.md)。
* 尝试使用错误的帐户信息登录到网页版密码库，并检查日志文件中如下格式的记录项：

```python
[YYYY-MM-DD hh:mm:ss][bitwarden_rs::api::identity][ERROR] Username or password is incorrect. Try again. IP: XXX.XXX.XXX.XXX. Username: email@domain.com.
```

## 安装 <a id="installation"></a>

### Debian / Ubuntu / Raspian Pi OS

```python
sudo apt-get install fail2ban -y
```

### Fedora / Centos

需要 EPEL 库（CentOS 7）

```python
sudo yum install epel-release
sudo yum install fail2ban -y
```

### 群晖 DSM <a id="synology-dsm"></a>

使用 Synology 的话，由于各种原因需要做更多的工作。完整的解决方案发布在[这里](https://github.com/sosandroid/docker-fail2ban-synology)。主要问题是：

1. 嵌入式 IP 禁令系统不适用于 Docker 容器
2. 嵌入式 iptables 不支持 `REJECT` 块类型
3. Docker GUI 不允许某些高级设置
4. 修改系统配置不符合升级要求

因此，我们将在 Docker 容器中使用 Fail2ban。[Crazy-max/docker-fail2ban](https://github.com/crazy-max/docker-fail2ban) 提供了一个很好的解决方案，并且 Synology 的 docker GUI 将被忽略。通过 SSH 的命令行，执行下列步骤（根据您的 Synology 配置调整 `volumeX`）。

1、获取 root 权限

```python
sudo -i
```

2、创建持久性文件夹

```python
mkdir -p /volumeX/docker/fail2ban/action.d/
mkdir -p /volumeX/docker/fail2ban/jail.d/
mkdir -p /volumeX/docker/fail2ban/filter.d/
```

3、将 `REJECT` 替换为 `DROP` 块类型

```python
# /volumeX/docker/fail2ban/action.d/iptables-common.local

[Init]
blocktype = DROP
[Init?family=inet6]
blocktype = DROP
```

4、创建 docker-compose 文件

```python
# /volumeX/docker/fail2ban/docker-compose.yml

version: '3'
services:
	fail2ban:
		container_name: fail2ban
		restart: always
		image: crazymax/fail2ban:latest
		environment: 
		- TZ=Europe/Paris
		- F2B_DB_PURGE_AGE=30d
		- F2B_LOG_TARGET=/data/fail2ban.log
		- F2B_LOG_LEVEL=INFO
		- F2B_IPTABLES_CHAIN=INPUT

		volumes:
		- /volumeX/docker/fail2ban:/data
		- /volumeX/docker/bw-data:/bitwarden:ro

		network_mode: "host"

		privileged: true
		cap_add:
			- NET_ADMIN
			- NET_RAW
```

5、使用命令行启动容器

```python
cd /volumeX/docker/fail2ban
docker-compose up -d
```

您现在应该看到该容器在 Synolog 的 Docker GUI 中运行了。在配置筛选器和 jail 后，您必须重新加载。

## 为网页密码库设置 <a id="setup-for-web-vault"></a>

按照惯例，`path_f2b` 代表 Fail2ban 工作所需的路径。这取决于您的系统，例如，在 Synology 上，是 `/volumeX/docker/fail2ban/`，但在其他系统上是 `/etc/fail2ban/`。

### Filter <a id="filter"></a>

创建文件并使用如下内容

```python
# path_f2b/filter.d/bitwarden_rs.local

[INCLUDES]
before = common.conf

[Definition]
failregex = ^.*Username or password is incorrect\. Try again\. IP: <ADDR>\. Username:.*$
ignoreregex =
```

**提示：**如果在 `fail2ban.log` 中出现以下错误消息（CentOS 7，fail2ban v0.9.7）  
`fail2ban.filter [5291]: ERROR No 'host' group in '^.*Username or password is incorrect\. Try again\. IP: <ADDR>\. Username:.*$'`  
请将 `bitwarden.local` 中的 `<HOST>` 改为 `<ADDR>`。

**提示：**如果您在 `bitwarden.log` 中看到 127.0.0.1 是登录失败的 IP 地址，那么您可能正在使用反向代理，而 fail2ban 无法正常工作：

```python
[YYYY-MM-DD hh:mm:ss][bitwarden_rs::api::identity][ERROR] Username or password is incorrect. Try again. IP: 127.0.0.1. Username: email@example.com.
```

要解决这个问题，需要通过 X-Real-IP 头将真实的远程地址转发给 bitwarden\_rs。如何操作呢？根据你使用的代理服务器不同而不同。例如，在 Caddy 2.x 中，当你定义反向代理时，同时定义 `header_up X-Real-IP {remote_host}`。更多信息请参阅[代理示例](../deployment/proxy-examples.md)。

### Jail

\[**译者注**\]：[Jail 是什么](https://www.freebsd.org/doc/zh_CN/books/arch-handbook/jail.html)

创建文件并使用如下内容

```python
# path_f2b/jail.d/bitwarden_rs.local

[bitwarden_rs]
enabled = true
port = 80,443,8081
filter = bitwarden_rs
banaction = %(banaction_allports)s
logpath = /path/to/bitwarden.log
maxretry = 3
bantime = 14400
findtime = 14400
```

请注意：Docker 使用 FORWARD 链而不是默认的 INPUT 链。因此，当使用 Docker 时，请用下面的 `action` 替换 `banaction` 行：

```python
action = iptables-allports[name=bitwarden_rs, chain=FORWARD]
```

**注意**：  
如果在 Docker 容器之前使用了反向代理，请不要做此操作。如果使用了 apache2 或 nginx 之类的代理，请使用代理的端口而不要使用 `chain = FORWARD`。仅当在**无**代理的 Docker 时使用！

**上面注意中的注意**：  
在使用 caddy 作为反向代理的 Docker（CentOS 7）上运行时，上面的说法是不正确的。当用 caddy 作为反向代理时，可以使用 `chain = FORWARD` 。

重新加载 fail2ban 使更改生效：

```text
sudo systemctl reload fail2ban
```

随意更改您认为合适的选项。

## 为管理页面设置 <a id="setup-for-admin-page"></a>

如果您通过设置 `ADMIN_TOKEN` 环境变量启用了管理控制台，则可以使用 Fail2Ban 来阻止攻击者暴力破解您的管理令牌。该过程与网页密码库相同。

### Filter <a id="filter"></a>

创建文件并使用如下内容

```python
# path_f2b/filter.d/bitwarden-admin.local

[INCLUDES]
before = common.conf

[Definition]
failregex = ^.*Invalid admin token\. IP: <ADDR>.*$
ignoreregex =
```

### Jail

创建文件并使用如下内容

```python
# path_f2b/jail.d/bitwarden_rs-admin.local

[bitwarden_rs-admin]
enabled = true
port = 80,443
filter = bitwarden_rs-admin
banaction = %(banaction_allports)s
logpath = /path/to/bitwarden.log
maxretry = 3
bantime = 14400
findtime = 14400
```

注意：Docker 使用 FORWARD 链而不是默认的 INPUT 链。因此，当使用 Docker 时，请用下面的 `action` 替换 `banaction` 行：

```python
action = iptables-allports[name=bitwarden_rs, chain=FORWARD]
```

重新加载 fail2ban 使更改生效：

```text
sudo systemctl reload fail2ban
```

## 测试 Fail2Ban <a id="testing-fail-2-ban"></a>

现在，尝试使用任何电子邮件地址登录 Bitwarden（不必是有效电子邮件，只需是电子邮件格式即可）。如果它可以正常工作，您的 IP 将被阻止。运行以下命令来取消 IP 阻止：

使用 Docker：

```python
sudo docker exec -t fail2ban fail2ban-client set bitwarden_rs unbanip XX.XX.XX.XX
```

未使用 Docker：

```python
sudo fail2ban-client set bitwarden_rs unbanip XX.XX.XX.XX
```

如果 Fail2Ban 无法正常运行，请检查 Bitwarden 日志文件的路径是否正确。对于 Docker：如果指定的日志文件未生成和/或更新，请确保将 `EXTENDED_LOGGING` 变量设置为 true（默认值），并且日志文件的路径是 Docker 内部的路径（当您使用 `/bw-data/:/data/` 时，日志文件应位于 `/data/` 中，而不是容器外部）。

还要确认 Docker 容器的时区与主机的时区是否一致。通过将日志文件中显示的时间与主机操作系统的时间进行比较来进行检查。如果它们不一致，则有多种解决方法。一种是使用 `-e "TZ = <timezone>"` 选项启动 docker 。可用的时区（比如 `-e TZ = "Australia/Melbourne"`）的列表在[这里](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones)查看。

如果您使用的是 podman 而不是 docker，则无法通过 `-e "TZ = <timezone>"` 来设置时区。可以按照以下指南解决此问题（当使用 alpine 镜像时）：[https://wiki.alpinelinux.org/wiki/Setting\_the\_timezone](https://wiki.alpinelinux.org/wiki/Setting_the_timezone)。

## SELinux 中的问题 <a id="selinux-problems"></a>

当使用 SELinux 时，SELinux 可能会阻止 fail2ban 读取日志。如果是这样，请运行此命令： `sudo tail /var/log/audit/audit.log`。您应该会看到如下类似内容（当然，实际的审核 ID 会因您的情况而不一样）：

```python
type=AVC msg=audit(1571777936.719:2193): avc:  denied  { search } for  pid=5853 comm="fail2ban-server" name="containers" dev="dm-0" ino=1144588 scontext=system_u:system_r:fail2ban_t:s0 tcontext=unconfined_u:object_r:container_var_lib_t:s0 tclass=dir permissive=0
```

您可以使用 `grep 'type=AVC msg=audit(1571777936.719:2193)' /var/log/audit/audit.log | audit2why` 来找出真正的原因。`audit2allow -a` 将为您提供有关如何创建模块并允许 fail2ban  访问日志的具体说明。

按照这些步骤操作后就结束了！fail2ban 现在应该可以正常工作了。

