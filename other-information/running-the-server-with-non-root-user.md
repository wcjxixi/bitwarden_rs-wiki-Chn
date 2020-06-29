# 3.使用非root用户运行服务器

{% hint style="success" %}
对应的[页面地址](https://github.com/dani-garcia/bitwarden_rs/wiki/Running-the-server-with-non-root-user)
{% endhint %}

默认设置应足够安全，因此容器内的root用户可以做的事情就非常有限了。如果您想尽力避免在容器中使用root用户，则可以这样做：

1. 创建非root用户拥有的数据文件夹，以便您可以使用该用户写入持久性数据。获取用户`id`。在Linux中，您可以运行`stat <folder_name>`以获取/验证所有者ID。
2. 运行容器时，需要提供用户ID作为参数之一。请注意，该名称必须为数字形式，而不是用户名，因为docker会尝试在镜像内查找此用户，而此用户可能不存在，或者用户的ID与本地用户的ID不同，导致不能写入持久性数据。使用`--user`参数。
3. 默认情况下，bitwarden\_rs侦听容器内部的`80`端口，这[不适用于非root用户](https://www.w3.org/Daemon/User/Installation/PrivilegedPorts.html)，因为常规用户不允许打开下面的`1024`端口。为了解决这个问题，您需要将服务器配置为侦听其他端口，可以使用它`ROCKET_PORT`来执行此操作。

下面是运行docker的示例，该示例使用具有ID为 `1000`且配置了端口重定向的用户，该服务侦听容器内部的`8080`端口，docker将其转换为外部（主机）的`80`端口：

```text
docker run -d --name bitwarden \
  --user 1000 \
  -e ROCKET_PORT=8080 \
  -v /bw-data/:/data/ \
  -p 80:8080 \
  bitwardenrs/server:latest
```

