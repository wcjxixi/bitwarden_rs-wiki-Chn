# 2.构建二进制

{% hint style="success" %}
对应的[页面地址](https://github.com/dani-garcia/bitwarden_rs/wiki/Building-binary)
{% endhint %}

如果您不想自己构建二进制文件，则可以查看 bitwarden\_rs 是否[已打包用于你的 Linux 发行版](third-party-packages.md)。

### 依赖关系

* `Rust nightly`（强烈建议使用 [rustup](https://rustup.rs/)）
* `OpenSSL`（应该在路径中，通过系统的包管理器安装，也可以使用[预构建的二进制文件](https://wiki.openssl.org/index.php/Binaries)） 对于 Debian，您需要安装`pkg-config`和`libssl-dev`
* `NodeJS`（仅当编译 web-vault 时使用，通过系统的包管理器安装，使用[预构建的二进制文件](https://nodejs.org/en/download/)）或 [nodesource 二进制发行版](https://github.com/nodesource/distributions)。_备注：web-vault 当前使用程序包库（例如，node-sass &lt;v4.12），这需要 NodeJS v11_
* 对于 Debian（Buster）上的 MySQL 后端，您需要安装`libmariadb-dev-compat`和`libmariadb-dev`

### 运行/编译

#### SQLite 后端

```php
# Compile with sqlite backend and run
cargo run --features sqlite --release
# or just compile with sqlite (binary located in target/release/bitwarden_rs)
cargo build --features sqlite --release
```

#### MySQL 后端

```php
# Compile with mysql backend and run
cargo run --features mysql --release
# or just compile with mysql (binary located in target/release/bitwarden_rs)
cargo build --features mysql --release
```

#### PostgreSQL 后端

```php
# Compile with postgresql backend and run
cargo run --features postgresql --release
# or just compile with postgresql (binary located in target/release/bitwarden_rs)
cargo build --features postgresql --release
```

运行后，可从通过 [http://localhost:8000](http://localhost:8000/) 访问服务器。

~~_**注意**：一个先前的_~~[~~_话题_~~](https://github.com/rust-lang/rust/issues/62896)~~_表明由于Rust编译器和LLVM之间存在不兼容，导致编译可能会因段错误而失败。作为解决方法，可以使用较旧版本的编译器，例如`cargo +nightly-2019-08-27 build --features yourbackend --release`_~~

#### 安装网络密码库

可以从 [dani-garcia/bw\_web\_builds](https://github.com/dani-garcia/bw_web_builds/releases) 下载网页密码库的编译版本。

如果您希望手动编译它，请按照如下步骤操作：

_**注意**：构建密码库需要约 1.5GB 的 RAM。在具有 1GB 或更小容量的 RaspberryPI 之类的系统上，请_[_启用交换功能_](https://www.tecmint.com/create-a-linux-swap-file/)_或在功能更强大的计算机上构建，然后从那里复制目录。仅构建时需要大量内存，而运行带密码库的 bitwarden\_rs 仅需要约 10MB 的 RAM。_

1、通过 [bitwarden/web](https://github.com/bitwarden/web) 克隆 git 库，并查看最新的发行标签（例如 v2.1.1）：

```bash
# 克隆库
git clone https://github.com/bitwarden/web.git web-vault
cd web-vault
# 查看最新的发布标签
git checkout "$(git tag --sort=v:refname | tail -n1)"
```

2、从 [dani-garcia/bw\_web\_builds](https://github.com/dani-garcia/bw_web_builds/tree/master/patches) 下载补丁文件，选择要使用的版本，并将其复制到`web-vault`文件夹。这里假设网页密码库版本为 `vX.Y.Z`：

* 如果有版本为`vX.Y.Z`的补丁，请使用该补丁
* 否则，选择最大版本仍小于`vX.Y.Z`的补丁

3、应用补丁：

```php
# 在'web-vault'目录中运行命令
git apply vX.Y.Z.patch
```

4、然后，构建密码库：

```yaml
npm run sub:init
npm install
npm run dist
```

_**注意**：可能会要求您运行`npm audit fix`以修复漏洞。这将自动尝试将软件包升级到较新的版本，该版本可能不兼容并破坏网页密码库功能。如果知道自己在做什么，请自行承担风险。_

5、最后将`build`文件夹的内容复制到目标文件夹中：

* 如果与`cargo run --release`一起运行，则为`bitwarden_rs/web-vault`。
* 如果您直接运行已编译的二进制文件，则它位于二进制文件旁，`bitwarden_rs/target/release/web-vault`中。

### 配置

可用的配置选项记录在默认的`.env`文件中，可以通过在该文件中取消注释所需选项或设置它们各自的环境变量来对其进行修改。有关可用的主要配置选项，请参见此 Wiki 的[配置](../configuration/)章节。

注意：环境变量将覆盖`.env`文件中设置的值。

### 有关部署的更多信息

* [配置反向代理](roxy-examples.md)
* [通过 systemd 设置自动启动](../configuration/creating-a-systemd-service.md)

### 如何为 SQLite 后端重建数据库模式（面向开发人员）

使用 cargo 安装 diesel\_cli：

```text
cargo install diesel_cli --no-default-features --features sqlite-bundled
```

确保`.env`文件中包含正确的数据库路径。

如果要修改模式，请使用以下命令创建新迁移：

```text
diesel migration generate <name>
```

修改`*.sql`文件，确保在`down.sql`文件中还原所有更改。

应用迁移并保存生成的模式，如下所示：

```php
diesel migration redo

# This step should be done automatically when using diesel-cli > 1.3.0
# diesel print-schema > src/db/sqlite/schema.rs
```

### 如何从 SQLite 后端迁移到 MySQL 后端（面向开发人员）

如果要从 SQLite 迁移，请参考[使用 MySQL 后端](../configuration/using-the-mysql-backend.md)。

