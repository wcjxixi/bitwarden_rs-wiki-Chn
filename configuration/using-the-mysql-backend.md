# 23.使用 MySQL 后端

{% hint style="success" %}
对应的[页面地址](https://github.com/dani-garcia/bitwarden_rs/wiki/Using-the-MySQL-Backend)
{% endhint %}

要使用 MySQL 后端，请首先确保您在[启用了](../deployment/building-binary.md#mysql-hou-duan) [MySQL](../deployment/building-binary.md#mysql-hou-duan) [功能](../deployment/building-binary.md#mysql-hou-duan)情况下构建二进制文件。

要运行二进制文件或容器，请确保已设置 `DATABASE_URL` 环境变量（即 `DATABASE_URL='mysql://<user>:<password>@mysql/bitwarden'`），并将 `ENABLE_DB_WAL` 设置为 false（即  `ENABLE_DB_WAL='false'`）。

**连接字符串语法：**

```python
DATABASE_URL=mysql://[[user]:[password]@]host[:port][/database]
```

如果密码包含特殊字符，则需要使用百分比编码。

| ! | \# | $ | % | & | ' | \( | \) | \* | + | , | / | : | ; | = | ? | @ | \[ | \] |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| %21 | %23 | %24 | %25 | %26 | %27 | %28 | %29 | %2A | %2B | %2C | %2F | %3A | %3B | %3D | %3F | %40 | %5B | %5D |

完整的百分比编码代码列表可以在 [Wikipedia](https://en.wikipedia.org/wiki/Percent-encoding#Percent-encoding_reserved_characters) [页面](https://en.wikipedia.org/wiki/Percent-encoding#Percent-encoding_reserved_characters)上找到。

**使用** **Docker** **的示例：**

```python
# Start a mysql container
docker run --name mysql --net <some-docker-network>\
 -e MYSQL_ROOT_PASSWORD=<my-secret-pw>\
 -e MYSQL_DATABASE=bitwarden\
 -e MYSQL_USER=<bitwarden_user>\
 -e MYSQL_PASSWORD=<bitwarden_pw> -d mysql:5.7

# Start bitwarden_rs with MySQL Env Vars set.
docker run -d --name bitwarden --net <some-docker-network>\
 -v $(pwd)/bw-data/:/data/ -v <Path to ssl certs>:/ssl/\
 -p 443:80 -e ROCKET_TLS='{certs="/ssl/<your ssl cert>",key="/ssl/<your ssl key>"}'\
 -e RUST_BACKTRACE=1 -e DATABASE_URL='mysql://<bitwarden_user>:<bitwarden_pw>@mysql/bitwarden'\
 -e ADMIN_TOKEN=<some_random_token_as_per_above_explanation>\
 -e ENABLE_DB_WAL='false' <you bitwarden_rs image name>
```

**使用非** **Docker MySQL** **服务器的示例：**

```python
Server IP/Port 192.168.1.10:3306 UN: dbuser / PW: yourpassword / DB: bitwarden
mysql://dbuser:yourpassword@192.168.1.10:3306/bitwarden
```

**从** **SQLite** **迁移到** **MySQL**

此[话题评论](https://github.com/dani-garcia/bitwarden_rs/issues/497#issuecomment-511827057)中描述了一种从 SQLite 迁移到 MySQL 的简单方法。下面重复这些步骤。请注意，使用此工具风险自负，强烈建议备份您的安装和数据！

1、为 bitwarden\_rs 创建一个新的（空）数据库： `CREATE DATABASE bitwarden_rs;`

2、创建一个新的数据库用户并授予数据库权限：

```python
CREATE USER 'bitwarden_rs'@'localhost' IDENTIFIED BY 'yourpassword';
GRANT ALL ON `bitwarden_rs`.* TO 'bitwarden_rs'@'localhost';
FLUSH PRIVILEGES;
```

您如果想使用一组受限的授权：

```python
CREATE USER 'bitwarden_rs'@'localhost' IDENTIFIED BY 'yourpassword';
GRANT ALTER, CREATE, DELETE, DROP, INDEX, INSERT, SELECT, UPDATE ON `bitwarden_rs`.* TO 'bitwarden_rs'@'localhost';
FLUSH PRIVILEGES;
```

3、配置 bitwarden\_rs 并启动它，以便 diesel 可以运行迁移并正确设置架构。不要做别的。

4、停止 bitwarden\_rs。

5、转储您现有的 SQLite 数据库：`sqlite3 db.sqlite3 .dump > sqlitedump.sql`。注意：在 Debian（Buster） 上，您需要为此安装 sqlite3。

6、从转储中删除架构创建和 diesel 元数据，仅保留实际数据： `grep "INSERT INTO" sqlitedump.sql | grep -v "__diesel_schema_migrations" > mysqldump.sql`

7、加载 MySQL 转储： `mysql -ubitwarden_rs -pyourpassword < mysqldump.sql`

8、启动 bitwarden\_rs。

_注意：使用_ _`--show-warnings`_ _加载_ _MySQL_ _转储，会突出显示导入期间_ _datetime_ _字段被截断了，这**似乎**还可以。_

```python
Note (Code 1265): Data truncated for column 'created_at' at row 1
Note (Code 1265): Data truncated for column 'updated_at' at row 1
```

