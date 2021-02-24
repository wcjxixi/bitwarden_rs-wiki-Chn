# 23.使用 MariaDB（MySQL）后端

{% hint style="success" %}
对应的[页面地址](https://github.com/dani-garcia/bitwarden_rs/wiki/Using-the-MySQL-Backend)
{% endhint %}

{% hint style="warning" %}
我们的构建基于 MariaDB 客户端库，因为 Debian 和 Alpine 都提供这个库。我们不支持最新的 Oracle MySQLv8 版本。如果您想使用 MySQLv8 而不是 MariaDB，那么请使用旧的密码散列方式来创建用户，而不要使用默认的方式。
{% endhint %}

要使用 MySQL 后端，你可以使用[官方的 Docker 镜像](https://hub.docker.com/r/bitwardenrs/server-mysql)，也可以[使用 MySQL](../deployment/building-binary.md#mysql-backend) 构建你自己的二进制。

要运行二进制或容器，请确保已设置 `DATABASE_URL` 环境变量（即 `DATABASE_URL='mysql://<user>:<password>@mysql/bitwarden'`）。

**连接字符串语法：**

```python
DATABASE_URL=mysql://[[user]:[password]@]host[:port][/database]
```

如果密码包含特殊字符，则需要使用百分号编码。

| ! | \# | $ | % | & | ' | \( | \) | \* | + | , | / | : | ; | = | ? | @ | \[ | \] |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| %21 | %23 | %24 | %25 | %26 | %27 | %28 | %29 | %2A | %2B | %2C | %2F | %3A | %3B | %3D | %3F | %40 | %5B | %5D |

完整的代码列表可以在 [Wikipedia 的百分号编码页面](https://zh.wikipedia.org/wiki/%E7%99%BE%E5%88%86%E5%8F%B7%E7%BC%96%E7%A0%81)上找到。

**使用** **Docker** **的示例：**

```python
# 启动 mysql 容器
docker run --name mysql --net <some-docker-network>\
 -e MYSQL_ROOT_PASSWORD=<my-secret-pw>\
 -e MYSQL_DATABASE=bitwarden\
 -e MYSQL_USER=<bitwarden_user>\
 -e MYSQL_PASSWORD=<bitwarden_pw> -d mysql:5.7

# 使用 MySQL 环境变量值启动 bitwarden_rs
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

**使用 docker-compose 的示例：**

```python
version: "3.7"
services:
 mariadb:
  image: "mariadb"
  container_name: "mariadb"
  hostname: "mariadb"
  restart: always
  env_file:
   - ".env"
  volumes:
   - "mariadb_vol:/var/lib/mysql"
   - "/etc/localtime:/etc/localtime:ro"
  environment:
   - "MYSQL_ROOT_PASSWORD=<my-secret-pw>"
   - "MYSQL_PASSWORD=<bitwarden_pw>"
   - "MYSQL_DATABASE=bitwarden_db"
   - "MYSQL_USER=<bitwarden_user>"

 bitwarden:
  image: "bitwardenrs/server-mysql:latest"
  container_name: "bitwarden"
  hostname: "bitwarden"
  restart: always
  env_file:
   - ".env"
  volumes:
   - "bitwarden_vol:/data/"
  environment:
## 当在 mysql URL 周围使用单括号时会出现问题，就像在普通的 docker 例子中的一样
   - "DATABASE_URL=mysql://<bitwarden_user>:<bitwarden_pw>@mariadb/bitwarden_db"
   - "ADMIN_TOKEN=<some_random_token_as_per_above_explanation>"
   - "RUST_BACKTRACE=1"
  ports:
   - "80:80"

volumes:
 bitwarden_vol:
 mariadb_vol:
```

**从** **SQLite** **迁移到** **MySQL**

此[话题评论](https://github.com/dani-garcia/bitwarden_rs/issues/497#issuecomment-511827057)中描述了一种从 SQLite 迁移到 MySQL 的简单方法。下面重复这些步骤。请注意，使用此方法风险自负，强烈建议备份您的安装和数据！

1、为 bitwarden\_rs 创建一个新的（空）数据库： 

```python
CREATE DATABASE bitwarden_rs;
```

2、创建一个新的数据库用户并授予数据库权限：

```python
CREATE USER 'bitwarden_rs'@'localhost' IDENTIFIED BY 'yourpassword';
GRANT ALL ON `bitwarden_rs`.* TO 'bitwarden_rs'@'localhost';
FLUSH PRIVILEGES;
```

如果要使用一组受限的授权：

```python
CREATE USER 'bitwarden_rs'@'localhost' IDENTIFIED BY 'yourpassword';
GRANT ALTER, CREATE, DELETE, DROP, INDEX, INSERT, SELECT, UPDATE ON `bitwarden_rs`.* TO 'bitwarden_rs'@'localhost';
FLUSH PRIVILEGES;
```

3、配置 bitwarden\_rs 并启动它，以便 [diesel](http://diesel.rs/) 可以运行迁移并正确设置模式。除此之外不要做别的。

4、停止 bitwarden\_rs。

5、使用下面的命令转储你现有的 SQLite 数据库。仔细检查你的 sqlite 数据库的名称，默认应该是 db.sqlite。

**注意：**在您的 Linux 系统上需要已经安装了 sqlite3 命令。

我们需要从 sqlite 转储的输出中删除一些查询，如创建表等，我们将在这里进行。

你可以使用以下单行命令：

```python
sqlite3 db.sqlite3 .dump | grep "^INSERT INTO" | grep -v "__diesel_schema_migrations" > sqlitedump.sql ; echo -ne "SET FOREIGN_KEY_CHECKS=0;\n$(cat sqlitedump.sql)" > mysqldump.sql
```

或者逐行运行下列命令：

```python
sqlite3 db.sqlite3 .dump | grep "^INSERT INTO" | grep -v "__diesel_schema_migrations" > sqlitedump.sql
echo "SET FOREIGN_KEY_CHECKS=0;" > mysqldump.sql
cat sqlitedump.sql >> mysqldump.sql
```

6、加载 MySQL 转储：

```python
mysql --force --password --user=bitwarden_rs --database=bitwarden_rs < mysqldump.sql
```

7、重新启动 bitwarden\_rs。

_注意：使用_ _`--show-warnings`_ _加载_ _MySQL_ _转储时，会突出显示 datetime_ _字段在导入期间被截断了，这**似乎**还可以。_

```python
Note (Code 1265): Data truncated for column 'created_at' at row 1
Note (Code 1265): Data truncated for column 'updated_at' at row 1
```

_注意：加载 mysqldump.sql 数据过程中出现加载错误_

```python
error (1064): Syntax error near '"users" VALUES('9b5c2d13-8c4f-47e9-bd94-f0d7036ff581'*********)
```

修复：

```python
sed -i s#\"#\#g mysqldump.sql
```

```python
mysql --password --user=bitwarden_rs
use bitwarden_rs
source /bw-data/mysqldump.sql
exit
```
