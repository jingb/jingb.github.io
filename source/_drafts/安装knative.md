---
title: 安装knative
tags:
---

后面会去做serverless，在自己的电脑装knative，记录下坑
[infoQ的hello world有安装过程，没有直接用，但适合先扫一眼有个感性的认识](https://www.infoq.cn/article/YiTsx91bMVgoETvc*BVz)
<!-- more -->

本文可以提供的信息包括，都是在本机试过可行
1. 装minikube拉镜像失败怎么搞
2. 装istio拉镜像失败怎么搞
3. 装knative-serving相关组件拉镜像失败怎么搞

本机是15年的mac pro，已经有virtual box了

先装minikube，按官网的装法不行，因为拉镜像都是长城外。官网有个走proxy的 https://minikube.sigs.k8s.io/docs/handbook/vpn_and_proxy  推荐
试了没什么卵用，还是没搞定。还得基于你有代理或者找公共代理的情况

后来随便看了下 minikube -h   有设置 --image-mirror-country --registry-mirror 的选项，亲测有效  送一篇阿里云的  https://developer.aliyun.com/article/221687  
minikube start \--image-mirror-country cn \--registry-mirror=XXXX 
至此minikube搞定

在装好的minikube k8s集群上装istio，看的knative官网的这个 [Installing Istio without sidecar injection(Recommended default installation)](https://knative.dev/docs/install/installing-istio)
kubectl describe 看到是拉镜像失败，用minikube ssh 上机器去看，虚拟机上的 docker并没有用minikube start时带的镜像参数。又是一通找官网的解决方案  
官网有个registry的方案，看了下有点复杂 https://minikube.sigs.k8s.io/docs/handbook/registry/  大体是要你的本机电脑有docker，然后设置端口转发，拉镜像再转发进你的虚拟机（安装kubernetes的那台机器）
没有用上面的方案，尝试直接修改docker的配置  先minikube ssh 可以直接登到虚拟机上 
虚拟机上docker的配置文件手动修改加上 docker.mirrors.ustc.edu.cn 的mirror，记得重启，再尝试装istio，成功了 

最蛋疼的部分，安装knative相关组件，因为 ustc的源没有 grc.io，搜了一堆的资料 包括去阿里云别人的镜像找，dockerhub官方的镜像找，都没有找到我要的v0.16.0版本，有v0.10.0的，太老了
看的[官方文档](https://knative.dev/docs/install/any-kubernetes-cluster/#serving_networking-0)，照着一步步操作 
我本机装的v0.16.0   https://github.com/knative/serving/releases/download/v0.16.0/serving-core.yaml  里面的镜像都在 grc.io，拉不下来
参考了这个思路 https://blog.didiyun.com/index.php/2018/11/23/minikube-knative/#3.1%20%E5%BA%94%E7%94%A8%E8%AE%BF%E9%97%AE%E6%BC%94%E7%A4%BA  

**因此得想其它办法，一个比较简单的办法是利用 Docker Hub ，可在国内 pull，但它同时能拉取国外镜像的特点，选择在 Docker Hub 上构建一个以目标镜像为 base 镜像的方式，然后将上面 release-lite.yaml 上的目标镜像替换为在 Docker HUb 上建立的镜像地址即可。如下为一个 Dockerfile 的示例：**
```
FROM gcr.io/knative-releases/github.com/knative/build/cmd/webhook@sha256:58775663a5bc0d782c8505a28cc88616a5e08115959dc62fa07af5ad76c54a97
MAINTAINER doop
```
思路是很好，尝试在阿里云的镜像服务找了下，并没有Dockerfile构建这种功能。实在没办法，买了1个月hostwinds 阿姆斯特丹一台虚拟机，上去手动装docker，拉grc.io需要的镜像，改成自己的tag，push到了dockerhub，再自己的mac从dockerhub上拉下来，真是绕，但总算搞定了……
**镜像都是public的，需要v0.16.0的可自取** 

{% asset_img knative镜像.png %}

cat /lib/systemd/system/docker.service | grep Env   看启动传进去的docker env生效没有 
minikube start --image-mirror-country cn --registry-mirror=https://2gtshzci.mirror.aliyuncs.com --cpus=6 --docker-env HTTP_PROXY=socks5://host.minikube.internal:1086 --docker-env HTTPS_PROXY=socks5://host.minikube.internal:1086 --docker-env

**这不是一篇从头到尾每个细节每条指令都展现的安装指导文，因为我自己也尝试了很多个解决办法搜了很多资料，但相信有些地方还是能帮到在搜解决方案的人**
 

