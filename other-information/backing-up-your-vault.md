# 2.备份密码库

{% hint style="success" %}
对应的[页面地址](https://github.com/dani-garcia/bitwarden_rs/wiki/Backing-up-your-vault)
{% endhint %}

## 1. sqlite3 数据库 <a id="1-the-sqlite-3-database"></a>

使用合适的 sqlite3 备份命令来备份 sqlite3 数据库。如果在数据库写操作期间进行备份，请确保数据库不会损坏。

```python
mkdir $DATA_FOLDER/db-backup
sqlite3 /$DATA_FOLDER/db.sqlite3 ".backup '/$DATA_FOLDER/db-backup/backup.sqlite3'"
```

可以通过 CRON 计划任务每天运行此命令，但是请注意，这会每次覆盖相同的 `backup.sqlite3` 文件。因此，该备份文件应通过使用附加时间戳的 CRON 计划任务命令或其他备份应用程序（如 Duplicati）通过增量备份保存。要恢复，简单地将 `backup.sqlite3` 覆盖为 `db.sqlite3` 即可（当 bitwarden\_rs 停止时）。

运行以上命令要求在 docker 主机系统上安装 sqlite3。您可以使用以下命令用 sqlite3 docker 容器实现相同的结果。

```python
docker run --rm --volumes-from=bitwarden bruceforce/bw_backup /backup.sh
```

您还可以运行带有集成 cron 守护程序的容器以自动备份数据库。有关示例，请参阅 [https://gitlab.com/1O/bitwarden\_rs-backup](https://gitlab.com/1O/bitwarden_rs-backup) 或 [https://github.com/shivpatel/bitwarden\_rs\_dropbox\_backup](https://github.com/shivpatel/bitwarden_rs_dropbox_backup)。

## 2.附件文件夹 <a id="2-the-attachments-folder"></a>

默认情况下，它位于 `$DATA_FOLDER/attachments`。

## 3.密钥文件 <a id="3-the-key-files"></a>

可选。它们仅用于存储当前登录用户的令牌，删除令牌只会将每个用户注销，从而迫使他们再次登录。默认情况下，它们位于 `$DATA_FOLDER`（默认为 docker 中的 /data）。有 3 个文件：rsa\_key.der、rsa\_key.pem 和 rsa\_key.pub.der。

## 4.图标缓存 <a id="4-icon-cache"></a>

可选。图标缓存可以自动重新下载，但是如果缓存很大，则可能需要很长时间。默认情况下它位于 `$DATA_FOLDER/icon_cache`。

