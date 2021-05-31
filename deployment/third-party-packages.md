# 4.第三方软件包

{% hint style="success" %}
对应的[页面地址](https://github.com/dani-garcia/vaultwarden/wiki/Third-party-packages)
{% endhint %}

本页面是一个第三方 vaultwarden 软件包的索引。由于这些软件包不是由 vaultwarden 维护或控制的，因此它们可能会比官方的发行版本滞后，有时甚至会很明显。如果你依赖这些软件包，你可能需要[启用监视](https://docs.github.com/en/github/managing-subscriptions-and-notifications-on-github/viewing-your-subscriptions#configuring-your-watch-settings-for-an-individual-repository)新的 vaultwarden 发行版本，并让维护者知道该软件包未保持更新。

## Arch Linux

可在[官方库](https://www.archlinux.org/packages/community/x86_64/bitwarden_rs/)中获取，同时包含了[网页密码库](https://www.archlinux.org/packages/community/any/bitwarden_rs-web/)。

## Debian

基于工具链的 docker 可用于构建 debian 软件包：[https://github.com/greizgh/bitwarden\_rs-debian](https://github.com/greizgh/bitwarden_rs-debian)。它将服务器和网页密码库捆绑在一起。

## CentOS7 / RHEL7

RPM 库由 @MrMEEE 打包在这里：[https://copr.fedorainfracloud.org/coprs/mrmeee/bitwarden\_rs/](https://copr.fedorainfracloud.org/coprs/mrmeee/bitwarden_rs/)...在附加包里同样包含有网页界面。

安装说明：[https://github.com/MrMEEE/bitwarden\_rs\_rpm/blob/master/README.md](https://github.com/MrMEEE/bitwarden_rs_rpm/blob/master/README.md)

任何 RPM 话题可以提交到这里：[https://github.com/MrMEEE/bitwarden\_rs\_rpm/issues](https://github.com/MrMEEE/bitwarden_rs_rpm/issues)

## CentOS 8 / RHEL 8

克隆的一个分支，它构建一个 RPM 并使用 Docker 将其推送到 COPR。

[https://github.com/alexpdp7/bitwarden\_rs/tree/rpm/packages/centos8](https://github.com/alexpdp7/bitwarden_rs/tree/rpm/packages/centos8) [https://copr.fedorainfracloud.org/coprs/koalillo/bitwarden\_rs/](https://copr.fedorainfracloud.org/coprs/koalillo/bitwarden_rs/)

## Nix \(OS\)

在 Nix 中，Bitwarden\_rs 被打包为 4 个包（一个用于 mysql、sqlite 以及 postgresql，一个用于 vault）。对于 NixOS 来说，由于有一个 services.bitwarden\_rs 模块，因此 bitwarden\_rs 也可以用 NixOS 的声明方式来配置。

## Cloudron

[Cloudron](https://cloudron.io/) 是一个帮助你在服务器上运行 Web 应用的平台。使用 Cloudron，你可以从 [App Library](https://cloudron.io/store/com.github.bitwardenrs.html) 中轻松地安装自定义域名的 Bitwarden\_rs。该应用包与上游网页密码库捆绑在一起，安装后不需要任何进一步的配置即可开始使用。Cloudron 团队会保持发行版跟踪并提供自动更新。

软件包代码和话题跟踪器可以在这里找到：[https://git.cloudron.io/cloudron/bitwardenrs-app](https://git.cloudron.io/cloudron/bitwardenrs-app)。

## Home Assistant <a id="home-assistant"></a>

[Home Assistant](https://www.home-assistant.io/) 是一个开源的家庭自动化平台。在这里可找到 bitwarden\_rs 社区插件：[https://github.com/hassio-addons/addon-bitwarden](https://github.com/hassio-addons/addon-bitwarden)。

## 用于 Ubuntu 20.04 的编译脚本 <a id="build-script-for-ubuntu-20-04"></a>

Dinger1986 创建了一个在 Ubuntu 20.04 上从源代码安装 bitwarden\_rs 的脚本，参见：[https://github.com/dinger1986/bitwardenrs\_install\_script](https://github.com/dinger1986/bitwardenrs_install_script)

## FreeBSD

在 FreeBSD 端口树中可用，并在 FreeBSD pkg 存储库中作为二进制包提供：`pkg install bitwarden_rs`

