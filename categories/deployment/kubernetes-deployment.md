# 在kubernetes上部署

{% hint style="success" %}
对应的[页面地址](https://github.com/dani-garcia/bitwarden_rs/wiki/Kubernetes-deployment)
{% endhint %}

在Kubernetes上部署有两种选择：

* 本机
* 通过[Helm](https://helm.sh/)

#### 本机

请查看[kubernetes-bitwarden\_rs](https://github.com/icicimov/kubernetes-bitwarden_rs)库以获取有关在Kubernetes中部署的示例。

它将在以[nginx-ingress-controller](https://github.com/kubernetes/ingress-nginx)和AWS [ELBv1](https://aws.amazon.com/elasticloadbalancing/features/#Details_for_Elastic_Load_Balancing_Products)作为后端的Kubernetes上设置一个功能齐全且安全的`bitwarden_rs`应用程序。它提供的不仅仅是简单的部署，还可以根据需要和设置使用全部或部分功能。

#### 通过Helm

请查看[helm-bitwarden\_rs](https://github.com/Skeen/helm-bitwarden_rs)库以获取有关在Kubernetes中部署的示例。

它将在以您选择的nginx控制器作为后端的Kubernetes上设置一个功能齐全且安全的`bitwarden_rs`应用程序。它运行良好，并已使用[microk8s](https://microk8s.io/)设置进行了测试。也支持通过[cert-manager](https://github.com/jetstack/cert-manager)生成SSL证书。

