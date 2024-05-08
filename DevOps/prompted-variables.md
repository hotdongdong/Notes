## Prompted variables 使用

- 当在 Octopus 上部署一些项目的时候，如果需要一些全局的变量（只针对环境 prd、dev、test）改变的，按照以前就是提前在 Variables 中去添加，比如 IngressBaseDomainName 或者 serverUrl 之类的；然后在每一次 create release 的时候会将当前整个提前定义好的 Project Variables 包成一个 Variable Snapshot，然后给 process 里面定义的步骤使用或者其他地方使用，这种更适合一些定义后就不需要怎么改动的变量；

  ![image](https://github.com/mindy0-0/StudyNotes/assets/88584214/4f08c414-7ee5-48ce-a21a-4389a5688353)

  create release 后可以在 Variable Snapshot 那里查看到当前这个包拿到的 Variables

  ![image](https://github.com/mindy0-0/StudyNotes/assets/88584214/7e0a4a27-df93-435b-9509-a6a612b48f9c)

- 目前 EA 有一个功能点：在每一次发布新版本的时候需要顺便确定好即将发布的版本是否需要强制更新；按照以前的写法，也就是在 Variables 的写法去提前定义好 UpdateForce 这个字段；

  ![image](https://github.com/mindy0-0/StudyNotes/assets/88584214/83d08f67-98c5-4bc7-8720-bc858acd5672)

  但是这种会有一个不太稳定的因素：每一次在 create release 之前需要到 Variables 页面去改变这个 ForceUpdate，涉及到顺序的问题；如果先 create release 后改字段值的话，在 deploy 前需要做个 update variables 的操作，否则不会触发到更新，原因就是 create release 的时候就已经将当前的 Variables 包成了一个 Variable Snapshot

  ![image](https://github.com/mindy0-0/StudyNotes/assets/88584214/23eaa45d-4c8d-434c-a1b9-a4eb99e883a7)

- 因此之前在 Variables 里面就提前定义好 ForceUpdate 为 true 或者 false 就不太稳定，因此换了一种方法：在 deploy 的时候顺便选择 ForceUpdate 字段为 true 或者 false 就可以了，以下是尝试的方法：

#### 尝试 一 、Process 中多加一个步骤

- 先在 Variables 中新增一个 ForceUpdate 的字段，值为空，env 这一些也为空

- 多加一个 RUN A SCRIPT 的 setp，取名为 Set variables；在里面编写 Script 是希望在 deploy 的过程中，可以手动去输入或者选择一个值（true 或者 false）然后将输入的这个值再赋值给 Variables 中提前已经设置好值为空的 ForceUpdate

  ```PowerShell
    # 在 Octopus Deploy 中运行的 Powershell 脚本
    # 通过用户选择设置 Octopus Deploy 变量

    # 定义选项
    $options = @("true", "false")

    # 提示用户选择 ForceUpdate 的值
    $selectedOption = $options | Select-String -Title "Select value for ForceUpdate" -Unique

    # 设置 Octopus 变量
    Set-OctopusVariable -name "ForceUpdate" -value $selectedOption

  ```

  其他步骤不变，在 run 的时候发现报错了，大概就是在 octopus 执行过程中是处于非交互模式的，一些交互提示功能是不可以用的，这个应该是因为 windows octopus 限制了，感觉如果是个客户端的话应该是可以实现到的

  ```
  Creating kubectl context to https://8.218.48.31:6443 (namespace default) using a Token
  May 8th 2024 10:21:48Error
  InvalidArgument: 找不到与参数名称“Title”匹配的参数。
  May 8th 2024 10:21:48Error
  所在位置 C:\Octopus\Work\20240508022144-681684-65642\Script.ps1:8 字符: 44
  May 8th 2024 10:21:48Error
  $selectedOption = $options | Select-String -Title "Select value for F ...
  May 8th 2024 10:21:48Error
  May 8th 2024 10:21:48Error
  在 <ScriptBlock>、C:\Octopus\Work\20240508022144-681684-65642\Script.ps1 中: 第 8 行
  May 8th 2024 10:21:48Error
  在 <ScriptBlock>、<无文件> 中: 第 1 行
  May 8th 2024 10:21:48Error
  在 <ScriptBlock>、C:\Octopus\Work\20240508022144-681684-65642\Octopus.FunctionAppenderContext.ps1 中: 第 211 行
  May 8th 2024 10:21:48Error
  在 <ScriptBlock>、C:\Octopus\Work\20240508022144-681684-65642\Bootstrap.Octopus.FunctionAppenderContext.ps1 中: 第 1876 行
  May 8th 2024 10:21:48Error
  在 <ScriptBlock>、<无文件> 中: 第 1 行
  May 8th 2024 10:21:48Error
  在 <ScriptBlock>、<无文件> 中: 第 1 行
  May 8th 2024 10:21:49Fatal
  The remote script failed with exit code 1
  May 8th 2024 10:21:49Fatal
  The action Set variables on SJ-HK-AKS failed
  ```

#### 尝试 二 ：使用 Prompted variables（最终实现方案）

- 发现在 deploy 的过程是做不到通过交互操作去选择改变字段后，重新看了 Octopus 的文档，学到了 Variables 中一个叫 Prompted variables 的东西

  1、不需要在 process 中新增 step 去通过脚本改变字段

  2、在 Variables 中设置

  - 步骤一：将普通变量设置为提示变量（prompted variables），在创建或者编辑变量的时候点击 OPEN DEITOR

    ![image](https://github.com/mindy0-0/StudyNotes/assets/88584214/e47bf0b4-b8fc-420b-ae79-1e8b48965766)

  - 步骤二：输入名称和描述（可选），并指定是否需要该值，勾选后在创建部署时必须提供必需的变量，并且不得为空或者空格

    ![image](https://github.com/mindy0-0/StudyNotes/assets/88584214/61761f0f-1b17-473a-a047-f45aac9d5ceb)

  - 步骤三：由于字段 ForceUpdate 的值是 true 或者 false，所以在 Control type 可以选用 Checkbox 或者 Drop down（避免是 text type 手动输入 true 或 false 会拼错的情况），我这里选择的是 Drop down，给了两个 options

    ![image](https://github.com/mindy0-0/StudyNotes/assets/88584214/fb726145-0ea9-453d-afde-8e5698235e6d)

  - 步骤四：配置好了以上后就可以测试一下

    ![image](https://github.com/mindy0-0/StudyNotes/assets/88584214/b67540ff-e23f-4f39-a1cb-2415436993f5)

    ![image](https://github.com/mindy0-0/StudyNotes/assets/88584214/0c7d68a8-b477-4037-ba0a-10ff33f2aeca)

    ![image](https://github.com/mindy0-0/StudyNotes/assets/88584214/ce1a4ff6-ec19-42e8-a70b-4aca7cbc092c)

    ![image](https://github.com/mindy0-0/StudyNotes/assets/88584214/0e1efcbe-d3bb-40f6-8155-d0dba127435f)
