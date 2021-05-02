# 1.构建你自己的镜像

{% hint style="success" %}
对应的[页面地址](https://github.com/dani-garcia/bitwarden_rs/wiki/Building-your-own-docker-image)
{% endhint %}

克隆库，然后从库的根目录运行以使用默认的 SQLite 后端进行构建：

```python
# 构建 docker 镜像:
docker build -t vaultwarden .
```

要使用 MySQL 后端构建，运行：

```python
# 构建 docker 镜像:
docker build -t vaultwarden --build-arg DB=mysql .
```

要使用 Postgresql 后端构建，运行：

```python
# 构建 docker 镜像:
docker build -t vaultwarden --build-arg DB=postgresql .
```

在 docker-compose.yml 中它看起来像这样：

```python
  vaultwarden:
    # image: vaultwarden/server-postgresql:latest
    image: vaultwarden
    build: 
      context: vaultwarden
      args: 
        DB: postgresql
```

