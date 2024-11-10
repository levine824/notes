# 介绍

Jenkins  是一款流行的开源持续集成（Continuous Integration）工具，广泛用于项目开发，具有自动 化构建、测试和部署等功能。

Jenkins 的特征： 

- 开源的 Java 语言开发持续集成工具，支持持续集成，持续部署。 
- 易于安装部署配置：可通过 方便 web 界面配置管理。 
- 消息通知及测试报告：集成 yum 安装,或下载 war 包以及通过 docker 容器等快速实现安装部署，可 RSS/E-mail 通过 RSS 发布构建结果或当构建完成时通过 e-mail 通知，生 成 JUnit/TestNG 测试报告。
- 分布式构建：支持 Jenkins 能够让多台计算机一起构建/测试。  
- 文件识别： Jenkins 能够跟踪哪次构建生成哪些 jar，哪次构建使用哪个版本的 jar 等。 
- 丰富的插件支持：支持扩展插件，你可以开发适合自己团队使用的工具，如：docker 等。

# 安装

```shell
# Jenkins 需要依赖 JDK，所以先安装 JDK1.8
yum install java-1.8.0-openjdk* -y
 
# 获取 Jenkins 安装包并安装
wget https://www.jenkins.io/zh/download/jenkins-2.190.3-1.1.noarch.rpm
rpm -ivh jenkins-2.190.3-1.1.noarch.rpm

# 修改 Jenkins 配置文件 /etc/syscofig/jenkins 内容
JENKINS_USER="root"
JENKINS_PORT="8080"

# 启动 Jenkins
systemctl start jenkins

# 访问地址：http://<IP>:8080

# 获取并输入 admin 账户密码
cat /var/lib/jenkins/secrets/initialAdminPassword

# 跳过插件安装
```

> Jenkins 国外官方插件地址下载速度非常慢，所以可以修改为国内插件地址：
>
> 1. Jenkins -> Manage Jenkins -> Manage Plugins，点击 Available，把 Jenkins 官方的插件列表下载到本地；
> 2. 替换 `/var/lib/jenkins/updates`文件里地址：
>
> ```shell
> sed -i 's/http:\/\/updates.jenkins
> ci.org\/download/https:\/\/mirrors.tuna.tsinghua.edu.cn\/jenkins/g' default.json && sed -i
>  's/http:\/\/
>  www.google.com/https:\/\/
>  www.baidu.com/g' default.json
> ```
>
> 3. Manage Plugins 点击 Advanced，把 Update Site 改为国内插件下载地址：`https://mirrors.tuna.tsinghua.edu.cn/jenkins/updates/update-center.json`；
> 4. 重启 Jenkins。

# 插件

1. 中文汉化插件：Jenkins -> Manage Jenkins -> Manage Plugins，点击 Available，搜索 "Chinese"；

2. 用户权限管理：Role-based Authorization Strategy；

3. 凭证管理：Credentials Binding；

   > 可以添加的凭证有 5种：
   >
   > - Username with password ：用户名和密码；
   > - SSH Username with private key Secret file ： 使用 SSH 用户和密钥；
   > - Secret file ： 需要保密的文本文件，使用时 Jenkins 会将文件复制到一个临时目录中，再将文件路径 设置到一个变量中，等构建结束后，所复制的 Secret file 就会被删除；
   > - Secret text ：需要保存的一个加密的文本串，如钉钉机器人或 Github 的 api token；
   > - Certificate ：通过上传证书文件的方式。