### 什么是Istio？
云平台为使用它的组织人员提供了巨大的福利。然而，不可否认的是，云平台也为开发运维团队带来了压力。开发者必须用微服务架构来满足可移植，同时，运维也管理着大量的不同平台上服务的部署。Istio让你连接、保护、控制、监控你的服务。

更高层面上来说，Istio帮助你减少了这些部署的复杂性，也减少了开发团队的压力。它是一个完全开源的、而且层对现有分部署应用透明的服务网格。它也是一个包含日志、遥感、策略系统的API的平台。Istio多样化的功能集让你成功、高效地运行一个分布式微服务架构，同时提供统一的方法来保护、连接、监控微服务。

### 什么是服务网格？
Istio解决了开发跟运维人员在单体应用程序向分布式微服务架构过度中遇到的挑战。来看看是如何实现的，它能让你对服务网格有个更好的了解。
服务网格，过去经常用来描述组成应用以及应用内部之间的微服务之间的网络。因为规模和复杂程度剧增，服务网格变得越来越难以理解和管控。它需要可以包含服务发现、负载均衡、故障恢复、采集指标、监控。服务网格也有更多复杂的可选需求，比如A/B 测试，金丝雀发布，访问速率限制、访问控制、段对端的认证。

Istio提供了整个服务网格的行为追踪和操作控制，提供了一个完整的解决方案，以满足微服务应用的各种需求。

### 为什么使用Istio？
Istio 可以轻松创建具有负载均衡、服务到服务身份验证、监视等功能的已部署服务网络, 而服务代码中很少或根本没有代码更改。通过在整个环境中部署一个特殊的 sidecar 代理来拦截微服务之间的所有网络通信, 然后使用其控制面板功能能，包括：

1、自动负载均衡
2、通过路由规则、重试、故障转移和故障注入对通信行为进行细粒度控制
3、支持访问控制、速率限制和配额的可插入策略层和配置 API
4、流量的自动指标、日志和跟踪, 包括群集入口和出口
5、通过强大的基于身份的身份验证和授权，确保服务到服务的安全

Istio是为扩展和满足不同部署需求而设计的。

### Istio安装过程

- 在阿里云上准备三台以上机器。

- 每台机器上安装centos7。

- 每台机器上安装docker。

- 在其中一台上安装Rancher。

参考官网，https://rancher.com/docs/rancher/v2.x/en/quick-start-guide/deployment/quickstart-manual-setup/

- 使用Rancher创建Kubernetes集群。

- 在本地安装kubectl。

- 在本地安装helm客户端。

下载地址，https://github.com/helm/helm/releases

解压到 /usr/local/bin

查看并记住版本 helm version

手动登录目标机器，执行

docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/tiller:<helm version>

docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/tiller:<helm version> gcr.io/kubernetes-helm/tiller:<helm version>

helm install install/kubernetes/helm/istio --name istio --namespace istio-system \

​    --values install/kubernetes/helm/istio/values-istio-demo.yaml --set global.hub=registry.cn-hangzhou.aliyuncs.com/aliacs-app-catalog

#### Rancher:

开源的企业级容器管理平台，提供了在生产环境中使用的管理Docker和Kubernetes的全栈化容器部署与管理平台。

包含四个部分：

1、基础设施编排，基础设施服务包括[网络](https://rancher.com/docs/rancher/v1.6/zh/rancher-services/networking)， [存储](https://rancher.com/docs/rancher/v1.6/zh/rancher-services/storage-service/)， [负载均衡](https://rancher.com/docs/rancher/v1.6/zh/rancher-services/load-balancer/)， [DNS](https://rancher.com/docs/rancher/v1.6/zh/rancher-services/dns-service/)和安全模块

2、容器编排与调度

3、应用商店

4、企业级权限管理