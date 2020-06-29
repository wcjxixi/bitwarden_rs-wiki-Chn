# 1.容器镜像的选择

{% hint style="success" %}
对应的[页面地址](https://github.com/dani-garcia/bitwarden_rs/wiki/Which-container-image-to-use)
{% endhint %}

如果您打算使用Docker或Podman来运行`bitwarden_rs`，您会发现有好几种可用的镜像版本。通常，您是在amd64（x86）硬件上运行此服务，并且您还想使用最新的代码，在这种情况下，`bitwardenrs/server`就是你要使用的镜像版本。

一些用户可能希望在不同的硬件上运行`bitwarden_rs`，或者他们更愿意运行此服务的特定版本。在这种情况下，就有多个选项了。

### 在不同的硬件架构上运行

最常见的架构是amd64。您的PC或服务器通常使用此架构，如果你不太确定的话，这是最有可能的情况。如果您使用像Raspberry Pi这样的单片计算机，这会有些棘手，因为它取决于CPU，也取决于所使用的OS。一般来说，您可以通过运行`docker version`命令来检查您的架构（请参阅`OS/Arch`信息）。或者尝试`uname -a`，这个命令也提供有关架构信息。

根据您的架构，您可以选用以下镜像之一：

#### `bitwardenrs/server:latest`

这是“默认镜像”，运行在amd64上（x86，64位）。

#### `bitwardenrs/server:alpine`

基于Alpine的amd64镜像，与上面相同，但体积稍小。

#### `bitwardenrs/server:raspberry`

适用于Raspberry Pi 2或更高版本，以及任何其他兼容主板的ARMv7hf镜像。此镜像无法在那些使用ARMv6 CPU的Raspberry Pi 1或Raspberry Pi Zero上运行。

#### `bitwardenrs/server:armv6`

适用于Raspberry Pi 1和Raspberry Pi Zero的ARMv6镜像。

#### `bitwardenrs/server:aarch64`

Aarch64镜像，该镜像运行在ARMv8设备上，比如Raspberry Pi 3或其他基于ARMv8的设备。

**请注意**，比如您的Raspberry Pi 3上使用的是Raspbian，由于Raspbian是ARMv7hf的分发版，则需要使用`bitwardenrs/server:raspberry`镜像。可能还需要在设备上安装aarch64分发版。

### 运行特定版本

通常情况下，使用我们这里提供的稳定版镜像的最新版本是比较好的。但是，如果您希望运行该服务的特定版本并按照自己的进度更新到下一个版本，则可以使用我们带版本控制的发布版本。

您可以通过运行带有版本号标记的镜像来运行特定版本，例如`bitwardenrs/server:1.7.0`。如果您在不同架构（请参见上文）上运行你的服务，则可以使用为你的架构提供的版本，例如`bitwardenrs/server:1.7.0-raspberry`

**请注意**，我们无法控制Bitwarden客户端应用程序的发布，它们通常支持最新的API，因此，运行较低版本的`bitwarden_rs`可能会导致客户端应用程序使用异常（例如，功能性缺失或损坏），但密码库是一个例外，因为它与镜像捆绑在一起。这就是为什么我们通常建议运行最新的镜像，因为它通常是与最新的官方应用程序最兼容的版本。

### 已报告的兼容性表

如果您要在下表中尚未存在的硬件上运行镜像，请在此处添加您的详细信息。

| 所用硬件 | 操作系统 | Docker架构报告 | 镜像选择 | 状态 | 注释 |
| :--- | :--- | :--- | :--- | :--- | :--- |
| 常规的64位服务器 | Ubuntu 18.04 | x86\_64 | `bitwardenrs/server` | OK |  |
| O-Droid HC2 | Armbian | arm7l（arm32） | `registry.lollipopcloud.solutions/arm32v7/bitwarden` （请参阅注释） | OK | 从上游来源建立的非官方镜像; `bitwardenrs/server:raspberry`是官方的等效镜像 |
| Raspberry Pi Zero W | Raspbian（4.14.98+） | linux / arm（armv6l） | `bitwardenrs/server:armv6` | OK |  |
| Raspberry Pi 3 B | Raspbian（4.14.98-v7 +） | linux / arm（armv7l） | `bitwardenrs/server:raspberry` | OK |  |
| Synology | DSM \(DSM 6.2.1-23824 Update 6\) | Docker-x64-17.05.0-0367 | `bitwardenrs/server:latest` | OK |  |
| Synology | DSM \(DSM 6.2.2-24922 Update 4\) | Docker-x64-18.09.0-0506 | `bitwardenrs/server:1.13.0-alpine` | OK |  |
| 常规的64位服务器 | Unraid 6.8.0 | 19.03.5 | `bitwardenrs/server:latest` | OK |  |

