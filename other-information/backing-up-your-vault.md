# 备份密码库

{% hint style="success" %}
对应的[页面地址](https://github.com/dani-garcia/bitwarden_rs/wiki/Backing-up-your-vault)
{% endhint %}

### 1. sqlite3数据库

使用合适的sqlite3备份命令来备份sqlite3数据库。如果在数据库写操作期间进行备份，请确保数据库不会损坏。

```text
mkdir $DATA_FOLDER/db-backup
sqlite3 /$DATA_FOLDER/db.sqlite3 ".backup '/$DATA_FOLDER/db-backup/backup.sqlite3'"
```

可以通过CRON计划任务每天运行此命令，但是请注意，每次都会覆盖相同的`backup.sqlite3`文件。因此，该备份文件应通过使用附加时间戳的CRON计划任务命令或其他备份应用程序（如Duplicati）通过增量备份保存。要恢复，只需用`backup.sqlite3`覆盖为`db.sqlite3`即可（当bitwarden\_rs停止时）。

运行以上命令要求在docker主机系统上安装sqlite3。您可以通过以下命令使用sqlite3 docker容器实现相同的结果。

```text
docker run --rm --volumes-from=bitwarden bruceforce/bw_backup /backup.sh
```

您还可以运行带有集成cron守护程序的容器以自动备份数据库。有关示例，请参见[https://gitlab.com/1O/bitwarden\_rs-backup](https://gitlab.com/1O/bitwarden_rs-backup)或[https://github.com/shivpatel/bitwarden\_rs\_dropbox\_backup](https://github.com/shivpatel/bitwarden_rs_dropbox_backup)。

### 2.附件文件夹

默认情况下，它位于 `$DATA_FOLDER/attachments`

### 3.密钥文件

可选。它们仅用于存储当前登录用户的令牌，删除令牌只会将每个用户注销，从而迫使他们再次登录。默认情况下，它们位于`$DATA_FOLDER`（默认为docker中的/data）。有3个文件：rsa\_key.der、rsa\_key.pem和rsa\_key.pub.der。

### 4.图标缓存

可选。图标缓存可以自动重新下载，但是如果缓存很大，则可能需要很长时间。默认情况下它位于`$DATA_FOLDER/icon_cache`。

