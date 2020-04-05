# 更新Bitwarden镜像

{% hint style="success" %}
对应的[页面地址](https://github.com/dani-garcia/bitwarden_rs/wiki/Updating-the-bitwarden-image)
{% endhint %}

更新非常简单，你只需确保保留了已装载的卷。如果您使用[此处](starting-a-container.md)示例中的挂载卷的方式，则只需使用`pull`拉取最新版的映像，使用`stop`和`rm`停止和移除当前容器，然后与之前相同的方式启动一个新容器：

```php
# 拉取最新版本的镜像
docker pull bitwardenrs/server:latest

# 停止并移除旧版本容器
docker stop bitwarden
docker rm bitwarden

# 使用已挂载的数据启动容器
docker run -d --name bitwarden -v /bw-data/:/data/ -p 80:80 bitwardenrs/server:latest
```

然后访问[http://localhost:80](http://localhost/)

如果您没有为持久性数据绑定挂载卷，则多一个中间步骤，就是使用中间容器来保留数据：

```php
# 拉取最新版本的镜像
docker pull bitwardenrs/server:latest

# 创建中间容器以保留数据
docker run --volumes-from bitwarden --name bitwarden_data busybox true

# 停止并移除旧版本容器
docker stop bitwarden
docker rm bitwarden

# 使用已挂载的数据启动容器
docker run -d --volumes-from bitwarden_data --name bitwarden -p 80:80 bitwardenrs/server:latest

# 移除中间容器（可选）
docker rm bitwarden_data

# 您可以保留数据容器以用于将来的更新，这样的话，可以跳过最后一步。
```

### 使用系统服务进行更新（在本例中为Debian/Raspbian）

```php
sudo systemctl restart bitwarden.service
sudo docker system prune -f
# 警告！这将删除已停止或未使用的容器，例如与bitwarden_rs不关联的容器
# 请仔细查看哪个容器是你需要的

docker ps -a
# 查看已停止的容器

# WARNING! This will remove:
#        - all stopped #containers
#        - all networks not used by at least one container
#        - all dangling images
#        - all dangling build cache
# 使用以下命令列出所有Docker镜像
docker images
# 这里你将看到所有未使用的镜像
#
```

`restart`命令将会依次停止容器、提取最新镜像、然后再次运行容器。`prune`命令将会移除目前较旧的容器（-f 表示不需要确认）。

如果需要，可以将它们放入cronjob中以计划任务自动运行（根据您的需要修改时间）：

```scheme
$ sudo crontab -e
0 2 * * * sudo systemctl restart bitwarden.service

0 3 * * * sudo /usr/bin/docker system prune -f
```

如果`/usr/bin/docker`不是docker的正确路径，可以使用命令`which docker`查看它的实际路径。

