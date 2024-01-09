CD 的配置与之前的项目没有其他的变化

主要还是通过两个步骤 DEPLOY KUBERNETES CONTAINERS 和 DEPLOY KUBERNETES INGRESS RESOURCE 结合使用进行部署

### step1：DEPLOY KUBERNETES CONTAINERS

- 首先要确定好要将 docker 镜像包部署到哪个集群，这个应该跟服务器域名相关

- Deployment

  - Volumes

    volumes 主要用处是跟 config map 结合使用

    ConfigMap 作为配置存储，用于存储项目的配置信息，而不必将配置硬编码到容器镜像中。这使得在不同环境中部署相同的镜像时，可以通过修改 ConfigMap 中的配置来适应不同的环境

    而 Volumes 用于共享数据，它可以将 ConfigMap 中的配置文件挂载到容器中，使得项目能够读取到这些配置文件，这种方式就可以确保项目可以动态地获取配置而无需重新构建镜像

    ![image](https://github.com/hotdongdong/StudyNotes/assets/88584214/793dc272-88ea-4692-84f4-62d6cdf23602)

- Containers

  - Image Details 添加要部署的镜像包（镜像包名称和 docker 库在 ci 那边作为 params 设置的）

    ![image](https://github.com/hotdongdong/StudyNotes/assets/88584214/8ca1c802-3040-48c0-aa2c-a613246f7cfc)

  - Volume Mounts

    将配置的 Volumes 挂载到 nginx 代理服务器上（映射到容器资源），每一个 Volume 都有一个唯一的名字

    ![image](https://github.com/hotdongdong/StudyNotes/assets/88584214/ef0ff812-8509-4654-9115-eb02a8a9987f)

- Namespace：命名空间

  不同项目可以在同一个集群中共存，而 namespace 可以隔离开，每个 namespace 的资源（pod、service、configmap 等）都会彼此隔离，属于不同的虚拟集群，可以视为不同环境中运行的应用程序和服务

- Service

  网络暴露，允许将应用程序的 IP 和端口公开给其他服务或外部网络，这样可以通过 DNS 名称或集群内部的虚拟 IP 地址来访问应用程序

- Config Map Items

  用于存储配置数据，如键值对、配置文件等，它可以用于将配置信息从应用程序的镜像中解耦，使得配置可以独立于应用程序的部署进行管理。可以在不重新构建镜像的情况下修改应用程序的配置

  EA 这里的话是挂载了一个 release-notes.md 文件，让应用程序去访问到这个 release-notes.md 获取到更新日志

  ![image](https://github.com/hotdongdong/StudyNotes/assets/88584214/087ec6b1-e318-45b9-b6a3-807085ef6097)

### step2：DEPLOY KUBERNETES INGRESS RESOURCE

- Ingress Annotations

  允许在 Ingress 资源上设置自定义注释，以便影响 Ingress 的行为和配置

  ![image](https://github.com/hotdongdong/StudyNotes/assets/88584214/bc89cfa0-6741-4b7f-8e52-7a452ead8f82)

  - nginx.ingress.kubernetes.io/ssl-redirect: 启用此注释后，Ingress 将会将 HTTP 请求重定向到 HTTPS

  - prometheus.io/probe: 用于启用 Prometheus 探针支持，允许 Prometheus 监控该 Ingress 资源的健康状态

  - kubernetes.io/ingress.class: 指定使用的 Ingress 控制器的类别，这里是使用了 nginx Ingress 控制器

  - nginx.ingress.kubernetes.io/proxy-body-size: 设置请求主体的大小限制为 0，即无限制

  - kubernetes.io/tls-acme: 启用 TLS 证书自动获取，通常与 cert-manager 一起使用

  - cert-manager.io/cluster-issuer: 指定 Cert Manager 的 ClusterIssuer 为 zerossl，用于管理 TLS 证书的颁发

- Ingress Host Rules

  定义域名，将域名映射到服务器

  ![image](https://github.com/hotdongdong/StudyNotes/assets/88584214/20061452-ea87-4acf-8b3c-3846159e7455)

- Ingress TLS

  为特定的域名配置 TLS（HTTPS）证书，实现加密通信

  ![image](https://github.com/hotdongdong/StudyNotes/assets/88584214/343c0d7b-f9ff-4018-b731-d3ba91667b0b)
