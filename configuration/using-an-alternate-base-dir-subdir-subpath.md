# 19.使用备用基本目录（子目录/子路径）

{% hint style="success" %}
对应的[页面地址](https://github.com/dani-garcia/vaultwarden/wiki/Using-an-alternate-base-dir)
{% endhint %}

通常，Bitwarden 被限制驻留在子域的根目录中，比如`https://bitwarden.example.com`。

此限制源自后端和网页密码库，他们尚未被设计为容纳备用基本目录（请参阅 [bitwarden/server\#277](https://github.com/bitwarden/server/issues/277)）。实际上，移动端/桌面端应用程序和浏览器扩展都可以使用带路径的基本 URL。

随着对 vaultwarden 的更改（[PR\#868](https://github.com/dani-garcia/vaultwarden/pull/868)（后端）和 [PR\#11](https://github.com/dani-garcia/bw_web_builds/pull/11)（网页密码库）），您现在已经可以使用备用基本目录配置功能齐全的实例了。

## 配置 <a id="configuration"></a>

只需将您的域名 URL 简单配置为包括基本目录即可。例如，假设您想使用 `https://vaultwarden.example.com/base-dir` 访问您的实例。（提示，您也可以根据需要使用多级目录，例如 `https://vaultwarden.example.com/multi/level/base/dir`）

1、停止 vaultwarden。

2、如果您通常使用管理页面来配置 vaultwarden，则将 `config.json` 编辑为如下所示：

```python
{
  "domain": "https://vaultwarden.example.com/base-dir",
  // ... other values ...
}
```

3、如果您通常通过环境变量来配置 vaultwarden，请将 `DOMAIN` 环境变量设置为基本 URL 以更新您的配置文件/脚本。例如：

```python
docker run -e DOMAIN="https://vaultwarden.example.com/base-dir" ...
```

4、重新启动 vaultwarden。

5、现在，您应该可以使用 `https://vaultwarden.example.com/base-dir/`（请注意最后面的斜杠）访问网页密码库了。出于尚不完全清楚的原因，如果您使用 `https://vaultwarden.example.com/base-dir`（最后面不带斜线），可能会遇到问题。

6、使用 `https://vaultwarden.example.com/base-dir` 配置您的应用程序或浏览器扩展。注意：这里如果添加斜杠，则应用程序和浏览器扩展会在保存前自动将其删除。

7、注意对于步骤 **5**。尾部的斜杠 / 问题可以通过在路由位置字符串后添加 / 来解决。比如，在nginx中：

```text
location /my-base-path {
  # 此配置将导致`/`问题
}

location /my-base-path-2/ {
  # 此配置完美运行
}
```

## 反向代理 <a id="reverse-proxying"></a>

既然 vaultwarden API 路由已设置为期望的基本目录，如果您将 vaultwarden 放置在反向代理后面，请确保将您的代理配置为将请求路径传递到 vaultwarden。假如反向代理在 `localhost:8080` 上监听您的 vaultwarden，来自 `https://vaultwarden.example.com/base-dir/api/sync` 的请求到达了您的反向代理，则该请求必须转发到 `http://localhost:8080/base-dir/api/sync`，而不是 `http://localhost:8080/api/sync`。

