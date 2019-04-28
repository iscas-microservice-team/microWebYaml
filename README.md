# 自研发微服务 MicroApp 的 Istio 部署 yaml

该微服务应用的各项组件和 yaml 均经过 Kubernetes v1.13.1 版本 和 Istio 1.0.6 版本的测试。


## 研发说明及相关项目

本 repository 中的 yaml 均为自主书写和研发, 配合相应的微服务 Image 进行使用。

相关微服务的源码仓库或说明如下所示:

1. Postgres 数据库模块(基于 Docker 镜像进行启动)

    
    # 基于 Docker 的 Postgres 数据库部署
    https://zhuanlan.zhihu.com/p/62775096

- 注意事项

    数据库应用于其他主机单独启动，而不部署在 Kubernetes/Istio 集群中。
    这样做的目的是模拟微服务应用场景：独立的数据库集群不应与微服务共同使用 Kubernetes/Istio 运维。
    而是单独由 DBA 或数据库团队负责。
    
    
2. 数据资源层模块 postgresManager

       
    # Github 源码仓库
    https://github.com/XinyaoTian/postgresManager
    # DockerHub 镜像仓库
    https://cloud.docker.com/u/xinyaotian/repository/docker/xinyaotian/psqlmanager
    
- 简介
    
    该模块用于对数据库进行操作、为 UI 表现层提供相应的数据结构和格式、
    以及使用 REST API 供其他微服务使用。
    
- 主要功能

    由于该模块计划部署于 Kubernetes 及 Istio 的集群中通过内网进行访问，
    不对集群暴露任何接口，故 API 部分没有检验身份的认证功能。
    对于有安全性需求的用户来说，这点需要特别注意。
    
- 源码编程语言

    采用 Golang 进行研发。
    
- 注意事项

    在 Istio 中，除 Deployment 和 Service 外，
    还需配合 Istio 中的 ServiceEntry 对集群出口流量的配置，否则集群内部节点将不能访问到
    位于集群外部的数据库服务。
    
    
3. UI 表现层 / 用户交互层 WebUI


    # Github 源码仓库
    https://github.com/XinyaoTian/microWebApp
    # DockerHub 镜像仓库
    https://cloud.docker.com/u/xinyaotian/repository/docker/xinyaotian/microweb-ui

- 简介

    直接与用户进行交互，最直观的 Web 表示层。
    
- 主要功能

    通过 postgresManager 提供的 API 向数据库提交或从数据库获取相应数据并展现。
    是直接与用户交互的微服务模块，亦是整个微服务架构中唯一的外部流量能够直接接触到的模块。
    
- 源码编程语言

    采用 Python3 配合 Flask 框架和多种工具库进行研发。
    
- 微服务版本

    为探究 Istio 的微服务流量管理的格式功能，本人特地开发了两套交互模块。
    版本 v1 使用原始 html 编写，而版本 v2 由于采用了 Bootstrap 资源而变得更加美观。
    可以用于探究 Istio 的熔断器、蓝绿部署、A/B测试等诸多流量管控功能。
    
- 注意事项

    在 Istio 中，除 Deployment 和 Service 外，
    还需配合 Istio 中的 Gateway 和 VituralService 
    对集群进入流量进行路由配置，否则外部流量将无法找到该服务。
    

## yaml 的使用方法

这些 yaml 总体分为 2 大类: "基于 Kubernetes 部署" 和 "基于 Kubernetes&Istio 部署"。

研发过程中 yaml 标签的各种功能和对应关系都在各个 yaml 中由详细的阐述和注释。

- 基于 Kubernetes 部署

    以 microweb- 开头的 yaml 都是基于 Kubernetes 进行部署的。
    它们可以帮助你在没有安装 Istio 时就直接体验这个微服务应用
    (当然你要先在集群外部署个数据库)
    
- 基于 Kubernetes&Istio 部署

    其他文件则是基于 Kubernetes&Istio 进行部署。
    
    以 dbmanager- 开头的文件用于部署数据资源层模块；
    而以 webui- 为开头的文件则用于部署 UI 表现层。
    
    欲启动完整的微服务应用，需要至少同时启动如下几个 yaml:
    
    1. dbmanager-svc-and-deploy.yaml 
    2. dbmanager-egress.yaml
    3. webui-svc-and-deploy.yaml
    4. webui-gateway-and-vs.yaml
    
    
## 作者信息

Author: XinyaoTian | leontian1024@gmail.com

Thanks for you reading...
    

