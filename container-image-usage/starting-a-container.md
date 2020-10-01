# 2.启动容器

{% hint style="success" %}
对应的[页面地址](https://github.com/dani-garcia/bitwarden_rs/wiki/Starting-a-Container)
{% endhint %}

请注意，`docker run` 命令的名字略有误导性，因为它不仅会创建一个容器，它还会启动容器。当在仅停止容器而不删除容器后使用 `docker run` 命令时，会导致发生冲突。为了一个简单的开始，请接着往下看。

## 创建容器

持久性数据存储在容器内的 /data 下，因此使用 Docker 进行持久性部署的唯一要求是在路径上挂载持久性卷：

```php
# 使用Docker:
docker run -d --name bitwarden -v /bw-data/:/data/ -p 80:80 bitwardenrs/server:latest
# 使用Podman as non-root:
podman run -d --name bitwarden -v /bw-data/:/data/:Z -e ROCKET_PORT=8080 -p 8080:8080 bitwardenrs/server:latest
# 使用Podman as root:
sudo podman run -d --name bitwarden -v bw-data:/data/:Z -p 80:80 bitwardenrs/server:latest
```

所有持久性数据将保存在 `/bw-data/` 路径下，您可以根据自己的需要调整此路径。

该服务将暴露在主机端口 80 或 8080 上。

对于非 x86 硬件或要运行特定版本，可以[选择其他镜像](which-container-image-to-use.md)。

如果您的 docker/bitwarden\_rs 运行在具有固定 IP 的设备上，则可以将主机端口绑定到该 IP 地址，从而避免将主机端口暴露到网络上。如下所示，将 IP 地址（例如 192.168.0.2）添加到主机端口和容器端口前面：

```php
# using Docker:
docker run -d --name bitwarden -v /bw-data/:/data/ -p 192.168.0.2:80:80 bitwardenrs/server:latest
```

## 启动容器

如果运行了 `docker stop bitwarden` 命令，或重启，亦或任何其他原因，容器停止了，则可以使用以下命令将其再次启动：

```php
docker start bitwarden
```

