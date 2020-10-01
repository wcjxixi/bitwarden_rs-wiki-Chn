# 1.容器镜像的选择

{% hint style="success" %}
对应的[页面地址](https://github.com/dani-garcia/bitwarden_rs/wiki/Which-container-image-to-use)
{% endhint %}

对于 Docker 和 Podman 用户，`bitwarden_rs` 提供了三种镜像，每种都支持数据库后端。每种镜像也有一系列可用的标签。这些镜像和标签描述如下。

## 镜像 <a id="images"></a>

* `bitwardenrs/server` -- 基于 Debian 的镜像，支持 SQLite。这个镜像是最被广泛使用/测试的，推荐给大多数用户使用，除非有特殊需求需要使用不同的数据库后端。这个镜像支持 `amd64` \(x86-64\)、`arm32v6`、`arm32v7` 和 `arm64v8` 架构。
* `bitwardenrs/server-mysql` -- 基于 Debian 的镜像，支持 MySQL。由于交叉编译问题（见 [\#530](https://github.com/dani-garcia/bitwarden_rs/issues/530)），这个镜像当前仅支持 `amd64`  架构。
* `bitwardenrs/server-postgresql` -- 基于 Debian 的镜像，支持 PostgreSQL。由于交叉编译问题（见 [\#530](https://github.com/dani-garcia/bitwarden_rs/issues/530)），这个镜像当前仅支持 `amd64`  架构。

`bitwardenrs/server`（即 SQLite 镜像）是一个[多架构](https://www.docker.com/blog/multi-arch-all-the-things/)镜像，这意味着它在一个镜像名下支持多种 CPU 架构。假设你运行的是支持的架构之一，简单地拉取 `bitwardenrs/server` 会自动产生适合你的环境的特定架构的镜像，但 Armv6 板卡除外，比如 Raspberry Pi 1 和 Zero（见 [moby/moby\#41017](https://github.com/moby/moby/issues/41017)）。Armv6 用户必须在镜像标签中指定 `arm32v6`，例如 `latest- arm32v6`。

请注意，MySQL 和 PostgreSQL 镜像目前只支持 `amd64`，所以它们不是多架构的。如果有人出面解决交叉编译的问题，这些镜像也可以成为多架构的。

## 镜像标签

每个 Docker 镜像都有不同的标签，每个标签都代表了镜像的一些变体或属性（例如，特定版本）。

### 通用标签

三钟镜像（SQLite、MySQL和PostgreSQL）都有以下标签：

* `latest` --  跟踪最新发布的版本（即带有版本号的标签）。大多数用户推荐使用这个标签，因为它通常是最稳定的。
* `testing` -- 跟踪源码库的最新提交的版本。这个标签推荐给想要提前获取最新功能或增强功能的用户。测试版一般都很稳定，但不可避免，它偶尔也会出现一些问题。
* `x.y.z` \(例如 `1.16.0`\) -- 代表一个特定的发布版本。

### SQLite 特有标签 <a id="sqlite-te-you-de-biao-qian"></a>

SQLite 镜像（`bitwardenrs/server`）有以下附加标签：

* `alpine` -- 功能上与 `latest` 相同，但它是基于 Alpine 而非 Debian，因此镜像更小。 `latest` 与 `alpine` 主要是一个偏好问题，但请注意 `alpine` 标签目前只支持 `amd64` 架构。
* `x.y.z-alpine` \(e.g., `1.16.0-alpine`\) -- 与 `alpine` 类似，但它代表一个特定的发布版本。
* `latest-arm32v6` -- 与 `latest` 相同，但明确表示为 `arm32v6` 镜像。目前，对于使用 Armv6 板卡（如 Raspberry Pi 1 和 Zero）的用户来说，需要这样做。否则，Docker 将尝试拉取 `arm32v7` 镜像，这将无法工作（见 [moby/moby\#41017](https://github.com/moby/moby/issues/41017)）。
* `testing-arm32v6` -- 与 `testing` 相同，但明确表示为 `arm32v6` 镜像。
* `x.y.z-arm32v6` \(例如 `1.16.0-arm32v6`\) -- 与 `latest-arm32v6` 类似，但它代表一个特定的发布版本。

## 镜像更新

偶尔，上游的 Bitwarden 项目（即 Bitwarden 公司）会对客户端做一些向后不兼容的改动，这就需要对服务器的实现做相应的改动。bitwarden\_rs 一般会及时推送新的版本来适应这些改动。

然而，由于上游控制着客户端的发布，而移动应用和浏览器扩展通常会自己自动更新，因此，对于 bitwarden\_rs 用户来说，保持更新为最新的 bitwarden\_rs 版本非常重要。否则，不兼容的客户端和服务器版本可能会导致突然中断或异常。

web vault 是唯一的例外：由于它与 bitwarden\_rs 镜像捆绑在一起，web vault 的版本总是与 bitwarden\_rs 服务器的版本相匹配。如果你只把 web vault 用作客户端（可能性不大），那么你就不需要担心这些兼容性问题。

## 历史标签

在增加对多架构镜像支持之前，所有特定镜像都有其自己的特定标签。你仍然可以在 Docker Hub 中找到以下这些旧版本的 bitwarden\_rs，但不应再使用它们：

* `raspberry` - Armv7hf 镜像可以运行在 Raspberry Pi 2 或更新版本上，也可以运行在任何其他兼容的板子上。这个镜像不能在 Raspberry Pi 1 或 Raspberry Pi Zero 上运行，因为他们使用 armv6 CPU。
* `armv6` - 运行在 Raspberry Pi 1 和 Raspberry Pi Zero 上的 Armv6 镜像。
* `aarch64` - Aarch64 镜像，可以运行在 ARMv8 设备上，如 Raspberry Pi 3 或其他基于 ARMv8 的设备。

需要**注意**的是，你的设备上要求安装 arch64 发行版，例如，如果你在Raspberry Pi 3 上使用 Raspbian，因为 Raspbian 是一个 `armv7hf` 发行版，你仍然需要使用 `raspberry` 标签。

## 已报告的兼容性表

如果您有在下表中尚未存在的硬件上运行镜像，请在此处添加您的详细信息。

| 使用的硬件 | OS | 报告的 Docker  架构 | 使用的镜像 | 状态 | 备注 |
| :--- | :--- | :--- | :--- | :--- | :--- |
| 常规 64bit 服务器 | Ubuntu 18.04 | x86\_64 | `bitwardenrs/server` | OK |  |
| O-Droid HC2 | Armbian | arm7l \(arm32\) | `registry.lollipopcloud.solutions/arm32v7/bitwarden` \(see notes\) | OK | 从上游资源建立的非官方镜像；`bitwardenrs/server:raspberry` 是官方的等效镜像 |
| Raspberry Pi Zero W | Raspbian \(4.14.98+\) | linux/arm \(armv6l\) | `bitwardenrs/server:armv6` | OK |  |
| Raspberry Pi Zero W | Raspbian \(4.19.66+\) | linux/arm \(armv6l\) | `bitwardenrs/server:latest` \(Multiarch\) | OK | 只有在使用 docker 实验性功能 "docker pull --platform=linux/arm/v6"时，才能使用。否则会选择错误的镜像\([https://github.com/dani-garcia/bitwarden\_rs/issues/1064](https://github.com/dani-garcia/bitwarden_rs/issues/1064)\) |
| Raspberry Pi 1 B | Raspbian \(4.19.97+\) | linux/arm \(armv6l\) | `bitwardenrs/server:armv6` | OK |  |
| Raspberry Pi 3 B | Raspbian \(4.14.98-v7+\) | linux/arm \(armv7l\) | `bitwardenrs/server:raspberry` | OK |  |
| Raspberry Pi 4 | Raspbian \(4.19.118-v7l+\) | linux/arm \(armv7l\) | `bitwardenrs/server:raspberry` | OK | 4go 版本, rev 1.1 |
| Synology | DSM \(DSM 6.2.1-23824 Update 6\) | Docker-x64-17.05.0-0367 | `bitwardenrs/server:latest` | OK |  |
| Synology | DSM \(DSM 6.2.2-24922 Update 4\) | Docker-x64-18.09.0-0506 | `bitwardenrs/server:1.13.0-alpine` | OK |  |
| 常规 64位 服务器 | Unraid 6.8.0 | 19.03.5 | `bitwardenrs/server:latest` | OK |  |

