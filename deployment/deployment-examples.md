# 4.部署示例

{% hint style="success" %}
对应的[页面地址](https://github.com/dani-garcia/bitwarden_rs/wiki/Deployment-examples)
{% endhint %}

本页是单机部署示例的索引。如果添加新的示例，请酌情新建一个类别，并保持一般情况下的有序性。

## Google Cloud

* [https://github.com/dadatuputi/bitwarden\_gcloud](https://github.com/dadatuputi/bitwarden_gcloud)

Bitwarden 的安装针对谷歌云的"永远免费"的 f1-micro 计算实例进行了优化。

## Kubernetes

* [https://github.com/icicimov/kubernetes-bitwarden\_rs](https://github.com/icicimov/kubernetes-bitwarden_rs)

它将在以 [nginx-ingress-controller](https://github.com/kubernetes/ingress-nginx) 和 AWS [ELBv1](https://aws.amazon.com/elasticloadbalancing/features/#Details_for_Elastic_Load_Balancing_Products) 作为后端的 Kubernetes 上设置一个功能齐全且安全的 `bitwarden_rs` 应用程序。它提供的不仅仅是简单的部署，还可以根据您的需要和设置使用全部或部分功能。

* [https://github.com/Skeen/helm-bitwarden\_rs](https://github.com/Skeen/helm-bitwarden_rs)

它将在以您选择的 nginx 控制器作为后端的 Kubernetes 上设置一个功能齐全且安全的 `bitwarden_rs` 应用程序。它运行良好，并已使用 [microk8s](https://microk8s.io/) 设置进行了测试。也支持通过 [cert-manager](https://github.com/jetstack/cert-manager) 生成 SSL 证书。

## Raspberry Pi

* [https://github.com/martient/bitwardenrs-ansible](https://github.com/martient/bitwardenrs-ansible)

在 raspberry pi 上为 bitwarden\_rs 进行 Ansible 部署。

## 共享主机 <a id="shared-hosting"></a>

* [https://github.com/jjlin/bitwardenrs-shared-hosting](https://github.com/jjlin/bitwardenrs-shared-hosting)

在 [DreamHost](https://www.dreamhost.com/) 上运行 bitwarden\_rs 的配置示例，但应该同样适用于许多其他共享主机服务。

## NixOS \(by tklitschi\)

这里有一个 NixOS 上的 bitwarden 配置示例。它不是很复杂，有您想使用的数据库类型的后端选项、用于系统服务专用备份的备份目录、启用它的选项以及配置选项。对于配置选项，你只需以 nix 语法[从 .env模板中](https://github.com/dani-garcia/bitwarden_rs/blob/1.13.1/.env.template)传递 .env 变量。请参阅[代理示例](roxy-examples.md)以了解 nixos-nginx 的示例配置。

配置示例：

```python
{pkgs,...}:
{
  services.bitwarden_rs = {
  enable = true;
  backupDir = "/mnt/bitwarden";
  

  config = {
      WEB_VAULT_FOLDER = "${pkgs.bitwarden_rs-vault}/share/bitwarden_rs/vault";
      WEB_VAULT_ENABLED = true;
      LOG_FILE = "/var/log/bitwarden";
      WEBSOCKET_ENABLED= true;
      WEBSOCKET_ADDRESS = "0.0.0.0";
      WEBSOCKET_PORT = 3012;
      SIGNUPS_VERIFY = true;
      ADMIN_TOKEN = (import /etc/nixos/secret/bitwarden.nix).ADMIN_TOKEN;
      DOMAIN = "https://exmaple.com";
      YUBICO_CLIENT_ID = (import /etc/nixos/secret/bitwarden.nix).YUBICO_CLIENT_ID;
      YUBICO_SECRET_KEY = (import /etc/nixos/secret/bitwarden.nix).YUBICO_SECRET_KEY;
      YUBICO_SERVER = "https://api.yubico.com/wsapi/2.0/verify";
      SMTP_HOST = "mx.example.com";
      SMTP_FROM = "bitwarden@example.com";
      SMTP_FROM_NAME = "Bitwarden_RS";
      SMTP_PORT = 587;
      SMTP_SSL = true;
      SMTP_USERNAME= (import /etc/nixos/secret/bitwarden.nix).SMTP_USERNAME;
      SMTP_PASSWORD = (import /etc/nixos/secret/bitwarden.nix).SMTP_PASSWORD;
      SMTP_TIMEOUT = 15;
      ROCKET_PORT = 8812;
    };
  };

  environment.systemPackages = with pkgs; [
    bitwarden_rs-vault
  ];

}
```

如果你有任何关于这部分的问题，请随时联系我。我在 matrix 上的  @litschi:litschi.xyz 、以及 IRC（hackint 和 freenode）上的 litschi 或简单地在 matrix.org 的 bitwarden\_rs 频道中询咨询我。

