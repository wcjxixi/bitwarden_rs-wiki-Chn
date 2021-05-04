# 2.构建二进制

{% hint style="success" %}
对应的[页面地址](https://github.com/dani-garcia/bitwarden_rs/wiki/Building-binary)
{% endhint %}

这个页面主要是给那些对 vaultwarden 开发感兴趣，或者有特殊原因想要构建自己的二进制的人。

普通用户应该使用从基于 Alpine 的 Docker 镜像中[提取的预构建二进制](pre-built-binaries.md)文件，[通过 Docker 部署](../container-image-usage/which-container-image-to-use.md)，或者[寻找第三方软件包](third-party-packages.md)。

## 依赖 <a id="dependencies"></a>

* `Rust nightly`（强烈建议使用 [rustup](https://rustup.rs/)）
* 在基于 Debian 的发行版上，一些通用软件包可确保构建能正常进行，请安装以下软件包：`build-essential`、`git`
* `OpenSSL`（应在路径中是可用的，通过系统的包管理器安装，或使用[预构建的二进制](https://wiki.openssl.org/index.php/Binaries)）。在基于 Debian 的发行版上，需要安装 `pkg-config` 和 `libssl-dev`
* 对于基于 Debian 发行版上的 SQLite3 后端，需要安装  `libsqlite3-dev`
* 对于基于 Debian 发行版上的 MySQL 后端，需要安装  `libmariadb-dev-compat` 和`libmariadb-dev`
* 对于基于 Debian 发行版上的 PostgreSQL 后端，需要安装  `libpq-dev` 和 `pkg-config`
* `NodeJS`（仅当编译网页密码库时使用。使用[预构建的二进制](https://nodejs.org/en/download/)，通过系统的包管理器安装）或 [nodesource 二进制发行版](https://github.com/nodesource/distributions)。_备注：web-vault 当前使用的基本程序包（例如，node-sass &lt; v4.12），要求 NodeJS v11_

## 运行/编译 <a id="run-compile"></a>

### 所有后端 <a id="all-backends"></a>

```python
# 使用所有后端编译并运行
cargo run --features sqlite,mysql,postgresql --release
# 或仅使用所有后端编译(二进制位于 target/release/vaultwarden)
cargo build --features sqlite,mysql,postgresql --release
```

### SQLite 后端 <a id="sqlite-backend"></a>

```python
# 使用 sqlite 后端编译并运行
cargo run --features sqlite --release
# 或仅使用 sqlite 编译(二进制位于 target/release/vaultwarden)
cargo build --features sqlite --release
```

### MySQL 后端 <a id="mysql-backend"></a>

```python
# 使用 mysql 后端编译并运行
cargo run --features mysql --release
# 或仅使用 mysql 编译(二进制位于 target/release/vaultwarden)
cargo build --features mysql --release
```

### PostgreSQL 后端 <a id="postgresql-backend"></a>

```python
# 使用 postgresql 后端编译并运行
cargo run --features postgresql --release
# 或仅使用 postgresql 编译(二进制位于 target/release/vaultwarden)
cargo build --features postgresql --release
```

运行后，通过 [http://localhost:8000](http://localhost:8000/) 访问服务器。

~~_**注意**：一个先前的_~~[~~_话题_~~](https://github.com/rust-lang/rust/issues/62896)~~_表明由于Rust编译器和LLVM之间存在不兼容，导致编译可能会因段错误而失败。作为解决方法，可以使用较旧版本的编译器，例如_~~ __~~_`cargo +nightly-2019-08-27 build --features yourbackend --release`_~~

### 安装网页密码库 <a id="install-the-web-vault"></a>

可以从 [dani-garcia/bw\_web\_builds](https://github.com/dani-garcia/bw_web_builds/releases) 下载网页密码库的编译版本。

如果您希望手动编译它，请遵循如下步骤：

_**注意**：构建密码库需要约 1.5GB 的 RAM。在具有 1GB 或更小容量的 RaspberryPI 之类的系统上，请_[_启用交换功能_](https://www.tecmint.com/create-a-linux-swap-file/)_或在功能更强大的计算机上构建，然后从那里复制目录。仅构建时需要大量内存，而运行带密码库的 vaultwarden 仅需要约 10MB 的 RAM。_

1、克隆 [bitwarden/web](https://github.com/bitwarden/web) git 库，并检查最新的发行标签（例如 v2.1.1）：

```python
# 克隆库
git clone https://github.com/bitwarden/web.git web-vault
cd web-vault
# 切换到最新的标签
git checkout "$(git tag --sort=v:refname | tail -n1)"
```

2、从 [dani-garcia/bw\_web\_builds](https://github.com/dani-garcia/bw_web_builds/tree/master/patches) 下载补丁文件并将其复制到 `web-vault` 文件夹。选择要使用的补丁文件的版本（假设网页密码库版本为 `vX.Y.Z`）：

* 如果有版本为 `vX.Y.Z` 的补丁，则使用该版本
* 否则，选择小于 `vX.Y.Z` 的最大的那一个版本

3、应用补丁：

```python
# 在'web-vault'目录中运行命令
git apply vX.Y.Z.patch
```

4、然后，构建密码库：

```python
npm install
# 阅读下方的备注（我们将其用于我们的 docker 构建）。
# npm 审计修复
npm run dist
```

_**备注**：可能会要求您运行 `npm audit fix` 以修复漏洞。这将自动尝试将软件包升级到较新的版本，该版本可能不兼容并破坏网页密码库功能。如果知道自己在做什么，请自行承担风险。顺便一提，我们会在自己的发行版中使用它！_

5、最后将 `build` 文件夹的内容复制到目标文件夹中：

* 如果与 `cargo run --release` 一起运行，则目标文件夹为 `vaultwarden/web-vault`。
* 如果直接运行已编译的二进制，则它位于二进制旁，为 `vaultwarden/target/release/web-vault`。

## 配置 <a id="configuration"></a>

可用的配置选项记录在默认的 `.env` 文件中，可以通过在该文件中取消注释所需的选项或设置它们各自的环境变量来对其进行修改。有关可用的主要配置选项，请参见本 wiki 的[配置](../configuration/)章节。

注意：环境变量将覆盖 `.env` 文件中设置的值。

## 有关部署的更多信息 <a id="more-information-for-deployment"></a>

* [配置反向代理](proxy-examples.md)
* [通过 systemd 设置自动启动](../configuration/creating-a-systemd-service.md)

## 如何为 SQLite 后端重建数据库模式（面向开发人员） <a id="how-to-recreate-database-schemas-for-the-sqlite-backend-for-developers"></a>

使用 cargo 安装 diesel\_cli：

```python
cargo install diesel_cli --no-default-features --features sqlite-bundled
```

确保在 `.env` 文件中包含正确的数据库路径。

如果要修改模式，请使用以下命令创建新迁移：

```python
diesel migration generate <name>
```

修改 `*.sql` 文件，确保在 `down.sql` 文件中还原了所有更改。

应用迁移并保存生成的模式，如下所示：

```python
diesel migration redo

# 当使用的 diesel-cli > 1.3.0 时，此步骤会自动完成
# diesel print-schema > src/db/sqlite/schema.rs
```

## 如何从 SQLite 后端迁移到 MySQL 后端（面向开发人员） <a id="how-to-migrate-from-sqlite-backend-to-mysql-backend-for-developers"></a>

如果要从 SQLite 迁移，请参考[使用 MySQL 后端](../configuration/database/using-the-mariadb-mysql-backend.md)。

## 如何从 SQLite 后端迁移到 PostgreSQL 后端（面向开发人员） <a id="how-to-migrate-from-sqlite-backend-to-postgresql-backend-for-developers"></a>

如果要从 SQLite 迁移，请参考[使用 PostgreSQL 后端](../configuration/database/using-the-postgresql-backend.md)。

