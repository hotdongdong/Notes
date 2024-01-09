### 一、目前 EA 是在 teamcity 中分了三个 build configurations，分别有不同的作用：

- Build for test：构建出两个操作系统(windows、mac)docker 镜像包用于部署 test 环境

- Build for prd：构建出两个操作系统(windows、mac)docker 镜像包用于部署 prd 环境

- Build for out：开发时测试，可直接在 ci 打包出安装包（本地打包可能会有开发环境依赖等影响）

配置步骤基本一致，除了 Build for out 无推送 docker 步骤，并且多配置了 Artifact paths 用于打包出安装包

![image](https://github.com/hotdongdong/StudyNotes/assets/88584214/ccf40399-9dfa-421e-9871-01393bd98d28)

### 二、 VCS 与其他前端项目配置基本一致，主要是去关联 github 仓库

### 三、 Parameters

- 首先在 project（外层）先设置了三个 configurations 都需要使用到的参数

  docker 相关的 params

  csc.key.password：用于解开应用签名证书时的密码

  ![image](https://github.com/hotdongdong/StudyNotes/assets/88584214/03d31b24-4cfa-4c7b-aa17-6da9f4173325)

- 接着在相应的 configuration 中设置对应的 params

  例如：appsetting，用于更改项目配置

  ![image](https://github.com/hotdongdong/StudyNotes/assets/88584214/826fb996-ad04-4a52-a71a-c4e7efd9c1ff)

### 四、Build Steps

- GitVersion：主要是为了生成版本号或版本信息，用于后续构建步骤或其他

  ```ts
  gitversion /output buildserver
  ```

- Install Dependencies：安装项目所需要的依赖

  ```ts
  yarn install
  brew install --cask --no-quarantine wine-stable
  ```

- Update Version

  由于 EA 涉及到自动更新功能，在每次构建的时候版本号都需要有一个递增的规律，因此考虑用 build counter 来作为 beta 版的版本号

  考虑会需要 git tag 发布正式版本，这里也结合了 build number，也就是构建时的名字，通过去判断构建名字来判断是否是 git tag，若是则直接使用 build number 作为版本号，若不是则使用 git version 再结合 beta.build_counter 来作为 beta 版本号

  重构建完版本号后将新 version 替换 package.json 中的旧 version

  ```ts
  BUILD_COUNTER="%build.counter%"
  BUILD_NUMBER="%build.number%"

  if [[ $BUILD_NUMBER =~ ^[0-9]+\.[0-9]+\.[0-9]+$ ]]; then
      newVersion=$BUILD_NUMBER
  else
      baseVersion=$(echo $BUILD_NUMBER | sed 's/[-+].*//')
      newVersion="${baseVersion}-beta.$BUILD_COUNTER"
  fi

  sed -i.bak "s/\"version\": \"[^\"]*\"/\"version\": \"$newVersion\"/" package.json

  cat package.json
  ```

- Change config

  修改配置文件，以往项目的配置文件在打包后是独立开来在打包文件下如 dist 文件下的，因此不必在 ci 中处理，只要在 cd 中多配置一个 configmap 然后将原有的配置文件替换掉即可

  但是 electron 比较特殊，打包后不是将静态代码放置服务器，而是将应用程序安装包放置在服务器，所以无法在 cd 阶段进行配置文件的替换，只能在 ci 构建应用程序包的时候就将配置文件更改

  这也是为什么在每个 configuration 下都要去加一个 params，为各个环境不同的配置文件

  这里的话是将配置文件作为一个 params，然后将其设置为环境变量，之后在项目工程中添加脚本，去将这个环境变量替换掉工程里原本的项目配置文件

  ```ts
  // ci  build step
  export APPSETTING=%appsetting%    // 将配置文件内容设置为环境变量
  node cicd/build-for-env.js    // 执行脚本文件实现替换
  ```

  ```ts
  // 项目根目录下建cicd文件夹，添加脚本文件 build-for-env.js
  const fs = require("fs");
  const path = require("path");

  // 提出已设置成环境变量的配置文件，去替换原本的appsetting.json
  const appsetting = JSON.parse(process.env.APPSETTING);

  fs.writeFile(
    path.join(__dirname, "../public/appsetting.json"),
    JSON.stringify(appsetting),
    "utf8",
    (err) => {
      if (err) throw err;
    }
  );
  ```

- Build：执行打包命令

  ```ts
  // 将应用签名证书密码设置为环境变量，用于打包时使用
  export CSC_KEY_PASSWORD=%csc.key.password%
  yarn build
  yarn build --w
  ```

- Change File：修改构建打包出的文件，需要更改两处

  - 由于构建 mac 和 windows 安装包的时候会生成许多自动更新不需要的文件，例如 mac 打包会生成以下：

    而真正需要用到的只有红框里面的文件，为了减少最后 docker build 的包，因此需要去掉，包括 windows 的一些不必要文件

    ![image](https://github.com/hotdongdong/StudyNotes/assets/88584214/7f3289b3-0531-4b9b-adbd-122cff9ebc3c)

    ```ts
    rm -r out/win-unpacked
    rm -r out/mac-universal
    rm -r out/EchoAvatar-mac.dmg.blockmap
    rm -r out/EchoAvatar-mac.zip.blockmap
    rm -r out/EchoAvatar-win.exe.blockmap
    ```

  - 由于在前面 update version 的时候会有生成 beta 版本的应用程序包，如果是 beta 版本的话，更新源文件名字会从 latest-mac.yml 变为 beta-mac.yml 或者 latest.yml 变为 beta.yml，这样会导致在自动更新检测更新源文件的时候找不到正确的名字导致 404，因此也需要更改

    ```ts
    // 检查 macOS 更新文件
    if [ -f out/beta-mac.yml ]; then
        mv out/beta-mac.yml out/latest-mac.yml
    fi

    // 检查 Windows 更新文件
    if [ -f out/beta.yml ]; then
        mv out/beta.yml out/latest.yml
    fi
    ```

- Docker Build Image：构建 docker 镜像包

  ```ts
  repository=`echo %build.number%|sed 's^+^-^g'`
  docker build -t %docker.tag%:$repository .
  ```

- Docker Push Image：推送镜像包至 docker 库

  ```ts
  repository=`echo %build.number%|sed 's^+^-^g'`
  docker push %docker.tag%:$repository
  ```

- Docker Remove：删除本地的 docker 镜像包

  ```ts
  repository=`echo %build.number%|sed 's^+^-^g'`
  docker rmi %docker.tag%:$repository
  ```

### 五、总结

三个 configuration 的基本配置是一样的，只不过 build for prd 和另外两个的 params 中的 appsetting 不一样（配置文件不同）

而 build for out 只是为了开发时的测试，因此不需要构建 docker 镜像包和推送 docker

需要在 ci 生成安装包则可以配置`Artifact paths`：

![image](https://github.com/hotdongdong/StudyNotes/assets/88584214/73f36c0e-77ec-400f-b5fc-316c6f11cf99)

![image](https://github.com/hotdongdong/StudyNotes/assets/88584214/ac4148a1-ca25-45c4-a2f4-5efe25b3562a)
