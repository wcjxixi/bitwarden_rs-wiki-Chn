# 24.使用备用基本目录（子目录/子路径）

{% hint style="success" %}
对应的[页面地址](https://github.com/dani-garcia/bitwarden_rs/wiki/Using-an-alternate-base-dir)
{% endhint %}

通常，Bitwarden 被限于驻留在子域的根中，比如`https://bitwarden.example.com`。

此限制源自后端和网页密码库，他们尚未被设计为容纳备用基本目录（请参阅 [bitwarden/server\#277](https://github.com/bitwarden/server/issues/277)）。实际上，移动端/桌面端应用程序和浏览器扩展都可以使用带路径的基础 URL。

随着对 bitwarden\_rs 的更改（[PR\#868](https://github.com/dani-garcia/bitwarden_rs/pull/868)（后端）和 [PR\#11](https://github.com/dani-garcia/bw_web_builds/pull/11)（网页密码库）），您已经可以在备用基本目录中配置功能齐全的实例了。

### 配置

只需将您的域 URL 配置为包括基本目录即可。例如，假设您想使用 `https://bitwarden.example.com/base-dir` 访问您的实例。（提示，您也可以根据需要使用多级目录，例如 `https://bitwarden.example.com/multi/level/base/dir`）

1、停止 bitwarden\_rs。

2、如果您通常使用管理页面来配置 bitwarden\_rs，则将 `config.json` 编辑为如下所示：

```python
{
  "domain": "https://bitwarden.example.com/base-dir",
  // ... other values ...
}
```

3、如果您通常通过环境变量来配置 bitwarden\_rs，请将 `DOMAIN` 环境变量设置为基本 URL 以更新您的配置文件/脚本。例如：

```python
docker run -e DOMAIN="https://bitwarden.example.com/base-dir" ...
```

4、重新启动 bitwarden\_rs。

5、现在，您应该可以使用 `https://bitwarden.example.com/base-dir/`（请注意最后面的斜杠）访问网页密码库了。出于尚不完全清楚的原因，如果您使用 `https://bitwarden.example.com/base-dir`（最后面不带斜线），可能会遇到问题。

6、使用 `https://bitwarden.example.com/base-dir` 配置您的应用程序或浏览器扩展。注意：这里如果添加斜杠，则应用程序和浏览器扩展会在保存前自动将其删除。

### 反向代理

如果将 bitwarden\_rs 放在反向代理后面，请确保将您的代理配置为将请求路径传递到 bitwarden\_rs，因为 bitwarden\_rs API 路由已设置为使用基本目录。这样，假如反向代理在 `localhost:8080` 上监听您的 bitwarden\_rs，来自 `https://bitwarden.example.com/base-dir/api/sync` 的请求击中了您的反向代理，则该请求将转到 `http://localhost:8080/base-dir/api/sync`，而不是 `http://localhost:8080/api/sync`。

