# 21.以非root用户运行Docker容器

{% hint style="success" %}
对应的[页面地址](https://github.com/dani-garcia/bitwarden_rs/wiki/Running-docker-container-with-non-root-user)
{% endhint %}

默认情况下`bitwardenrs/server`使用root用户在容器内运行服务。如果需要，只需设置几项即可以非root用户身份运行：

1、确保挂载在容器内部的目录对用户是可写的。例如，如果您决定以`nobody`身份运行，则该目录必须可以由ID为65534的用户写入。有关在容器内指定用户的其他方式，请参阅[docker文档](https://docs.docker.com/engine/reference/run/#user)，在这里的示例中，我们使用`nobody`。

```text
# Make the directory on the host, change this to you preferred path
sudo mkdir /bw-data

# Set the owner using user id. 
# Note that the ownership must match user in /etc/passwd *inside* the container, not on your host
sudo chown 65534 /bw-data

# Give the owner full rights to the folder
sudo chmod u+rwx /bw-data
```

2、使用适当的参数启动容器。定义用户并确保使用`1024`或更高的端口启动。

```text
docker run -d \
  --name bitwarden \
  --user nobody \
  -e ROCKET_PORT=1024 \
  -v /bw-data/:/data/ \
  -p 80:1024 \
  bitwardenrs/server:latest
```

请注意，端口映射（`-p 80:1024`）与`ROCKET_PORT`的设置要对应。

