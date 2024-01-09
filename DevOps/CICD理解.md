## 简单介绍一下 DevOps

DevOps 是一种思想，整个思想是涵盖了开发、测试、分发的整个过程，主要是通过一些自动化工具来实现的，目的就是可以高效的交付

一个项目从开始新建文件夹到分发到用户使用，要懂的不应该只有技术开发，如何将开发的产品提交到用户手里的过程也是很重要的

DevOps 主要是利用一些自动化工具，当项目开发完了后通过这些自动化工具和流程就可以实现更快、更稳定的软件交付

### 自动化工具

#### 1、Docker

Docker 是个开源的应用容器引擎，可以创建、部署和运行应用程序的容器

它可以在任何支持 Docker 的系统上运行，无论该系统的配置和安装的操作系统和如何，它提供了一种在隔离的环境中打包应用程序及其依赖项的方式，使得应用可以在不同的环境中一致的运行

#### 2、Teamcity

teamcity 是由 JetBrains 开发的一个持续集成和持续部署（CI/CD）服务器，做持续集成工作。当开发人员在 github 上提交代码更改，teamcity 会拉取仓库代码，执行构建和测试

teamcity 的架构图

![image](https://github.com/hotdongdong/StudyNotes/assets/88584214/63aad5a9-d926-4715-8d03-16d6605ef665)

#### 3、Octopus Deploy

Octopus Deploy 为自动化部署提供了一种安全、可重复、可靠的方式，主要是为了接管 CI 服务器结束的位置，是一个自动化部署和发布管理工具，自动化在多个环境（如开发、测试和生产环境）中部署软件的过程

#### 4、对 DevOps 流程的理解

![image](https://github.com/hotdongdong/StudyNotes/assets/88584214/853857b6-4e15-48ec-8ef0-92098331a541)

a、在项目根目录下添加一个`Dockerfile`文件，内容主要是将打包出来的包放在 docker 里

- `Dockerfile`

  简单理解：将 Docker 当作是一个远程的虚拟的服务器来使用

  - 先在本地进行测试，本地安装并启动 Docker

  - 终端执行打包命令（例如 yarn build）生成打包文件（例如 build 文件）

  - 编辑 Dockerfile 文件，类似于写个脚本：将 build 文件放在 Docker 里（就像不使用 DevOps 这个自动化构建部署流的时候，也需要将打包出来的文件放在服务器一样的道理）

  - 执行 `docker build -f Dockerfile -t XXX-web .`,这个命令就是使用 Dockerfile 来创建一个 docker 镜像

  - 执行 `docker run -p 8188:80 -t XXX-web`，启动 docker 去 run 这个名字为`XXX-web`的包，80 端口就是在 Dockerfile 文件里面自定义的端口

  - 在浏览器打开 `http://localhost:8188`进行测试

  ```ts
  // Dockerfile文件内容

  // 指定基础镜像，使用`nginx`镜像
  FROM nginx:stable-alpine

  // 将打包后的build文件夹复制到容器中的usr/share/nginx/html目录
  COPY /build /usr/share/nginx/html

  // 当有404错误都会返回`index.html`文件
  RUN sed -i '12a error_page 404 /index.html;' /etc/nginx/conf.d/default.conf

  // 修改`nginx`主配置文件，开启gzip压缩，提高传输效率和加快页面加载时间
  RUN sed -i '/^http {/a \
  gzip on;\n\
  gzip_static on;' /etc/nginx/nginx.conf

  // 暴露80端口，是`nginx`默认都HTTP端口，也可以使用其他的
  EXPOSE 80

  // 启动`nginx`服务器
  CMD ["nginx", "-g", "daemon off;"]
  ```

b、开发者提交代码至 Github 仓库，Github 通过 Webhook 通知 Teamcity 由代码更改

c、Teamcity 执行创建，包括编译代码和运行测试

d、Teamcity 创建 Docker 镜像，并将其推送到 Docker 仓库

- Teamcity build step

  teamcity 的构建步骤配置基本跟本地测试的流程一样，先打包出 build 文件，再把文件放到 docker 镜像里面，最后将这个 docker 镜像给推送到 docker 仓库里

  ![image](https://github.com/hotdongdong/StudyNotes/assets/88584214/00ecc9bc-1eb9-4ab4-ba72-1f4ad81d5d2f)

e、Octopus Deploy 从 Docker 仓库中检索提取 Docker 镜像并部署到不同的环境中

- 关键步骤：在 process 的第一步配置（Deploy web）中的 Container 中

这两个值看 teamcity 中 Parameters 的 docker.registry 和 Build step 中构建 docker 镜像的名字

![image](https://github.com/hotdongdong/StudyNotes/assets/88584214/ae1cb4df-bf2f-46e4-81b2-47c81e22f4c3)

#### 5、Kubernetes (K8s) 部署

- K8s 用于自动化容器化应用程序的部署、扩展和操作

  - 容器化：K8s 基于容器技术，主要支持 Docker 容器，允许开发人员将应用程序及其依赖项打包成一个可移植的容器，使得在不同环境中的部署变得更加一致和可靠

  - 集群：K8s 集群是由多个物理或虚拟机器组成的，被称为节点

  - Pod：是 K8s 中最小的可部署和可扩展的单元，包含一个或多个容器，它们共享相同的网络命名空间、IP 地址和存储卷，通常用于组织、调度和管理容器

  - 服务（Service）：定义一组 Pod 并提供一个稳定的访问点，使集群内部和外部能够访问应用程序

- 步骤：

  - step1：DEPLOY KUBERNETES CONTAINERS

    主要是将容器化的应用程序给部署到 K8s 集群中，在这个步骤中主要提供：

    - 配置 Kubernetes 目标集群

    - 镜像名称和标签（Containers）

      ![image](https://github.com/hotdongdong/StudyNotes/assets/88584214/03d15069-80c9-458d-aeb0-1432b174f39f)

    - Service 用于定义一组 Pod，提供一个稳定的网络访问点的资源，将端口暴露供外部访问

      ![image](https://github.com/hotdongdong/StudyNotes/assets/88584214/bd7fa8be-295b-4e7a-b724-c5aca6490eda)

  - step2：DEPLOY KUBERNETES INGRESS RESOURCE

    部署资源，允许外部流量进入到集群中并访问服务

    通过配置 Ingress Host Rules 和 Ingress TLS 可以实现基于主机名的流量路由，并提供加密的安全连接。对于将外部流量正确的路由到不同的服务以及通过 HTTPS 加密流量非常有用

- 总结

  - "Deploy Kubernetes Containers" 步骤部署容器化的应用程序和 Nginx 到 Kubernetes 中

  - Nginx 以容器形式运行在 Pod 中，作为应用程序的一部分

  - "Deploy Kubernetes Ingress Resource" 步骤配置 Ingress 资源，定义了外部流量如何在集群内部进行路由

  - 外部流量通过 Ingress 控制器被路由到相应的服务
