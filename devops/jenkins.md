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

2. 用户权限管理：Role-based Authorization Strategy，RBAC；

3. 凭证管理：Credentials Binding；

   > 可以添加的凭证有 5种：
   >
   > - Username with password ：用户名和密码；
   > - SSH Username with private key Secret file ： 使用 SSH 用户和密钥；
   > - Secret file ： 需要保密的文本文件，使用时 Jenkins 会将文件复制到一个临时目录中，再将文件路径 设置到一个变量中，等构建结束后，所复制的 Secret file 就会被删除；
   > - Secret text ：需要保存的一个加密的文本串，如钉钉机器人或 Github 的 api token；
   > - Certificate ：通过上传证书文件的方式。

4. 流水线插件：Pipeline；

5. 钩子插件：Gitlab Hook；

6. 邮箱插件：Email Extension Template；

7. Kubernetes 插件：Kubernetes；

# 构建

Jenkins 中自动构建项目的类型有很多，常用的有以下三种： 

- 自由风格软件项目（FreeStyle Project） 

- Maven 项目（Maven Project） 

- 流水线项目（Pipeline Project）

每种类型的构建其实都可以完成一样的构建过程与结果，只是在操作方式、灵活度等方面有所区别，在实际开发中可以根据自己的需求和习惯来选择。一般推荐使用流水线类型。

## Pipeline

Pipeline，简单来说，就是一套运行在 Jenkins 上的工作流框架，将原来独立运行于单个或者多个节点 的任务连接起来，实现单个任务难以完成的复杂流程编排和可视化的工作。

Pipeline 脚本是由 Groovy 语言实现的，支持两种语法：Declarative(声明式)和 Scripted Pipeline(脚本式)语法。

Pipeline 有两种创建方法：可以直接在 Jenkins 的 Web UI 界面中输入脚本；也可以通过创建一个 Jenkinsfile 脚本文件放入项目源码库中（一般我们都推荐在 Jenkins 中直接从源代码控制(SCM) 中直接载入 Jenkinsfile Pipeline 这种方法）。

Pipeline语法可以查看[官网](https://www.jenkins.io/zh/doc/)。

### 声明式

```groovy
pipeline {
    agent any
    stages {
        stage('拉取代码') {
            steps {
                echo '拉取代码'
            }
        }
        stage('编译构建') {
            steps {
                echo '编译构建'
            }
        }
        stage('项目部署') {
            steps {
                echo '项目部署'
            }
        }
    }
}
```

- stages：代表整个流水线的所有执行阶段。通常 stages 只有1个，里面包含多个 stage。

- stage：代表流水线中的某个阶段，可能出现 n 个。一般分为拉取代码，编译构建，部署等阶段。 

- steps：代表一个阶段内需要执行的逻辑。steps 里面是 shell 脚本，git 拉取代码，ssh 远程发布等任意内容。

### 脚本式

```groovy
node {
   def mvnHome
   stage('Preparation') { // for display purposes
   }
   stage('Build') {
   }
   stage('Results') {
   }
 }
```

- Node ：节点，一个 Node 就是一个 Jenkins 节点，Master 或者 Agent，是执行 Step 的具体运行 环境，后续讲到 Jenkins 的Master-Slave 架构的时候用到。

- Stage ：阶段，一个 Pipeline 可以划分为若干个 Stage，每个 Stage 代表一组操作，比如： Build、Test、Deploy，Stage 是一个逻辑分组的概念。

- Step ：步骤，Step 是最基本的操作单元，可以是打印一句话，也可以是构建一个 Docker 镜像， 由各类 Jenkins 插件提供，比如命令：`sh make`，就相当于我们平时 shell 终端中执行 make 命令 一样。

### 样例

```groovy
pipeline {
    agent any
    stages {
        stage('拉取代码') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/master']], 
                doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], 
                userRemoteConfigs: [[credentialsId: '<ID>', url: 
                '<URL>']]])
            }
        }
        stage('编译构建') {
            steps {
                sh label: '', script: 'mvn clean package'
            }
        }
        stage('项目部署') {
            steps {
                deploy adapters: [tomcat8(credentialsId: 'ID',path: '', 
                url: 'http://<IP>:<PORT>')], contextPath: null, war: 'target/*.war'
            }
        }
    }
}
```

## 触发器

Jenkins 内置 4 种构建触发器：

- 触发远程构建

- 其他工程构建后触发（ Build after other projects are build）

- 定时构建（ Build periodically）

- 轮询 SCM（Poll SCM）

## Git hook 自动触发构建

利用 Gitlab 的 webhook 实现代码 push 到仓库，立即触发项目自动构建;

1. 在 Jeknkins 中生成 webhook URL，把生成的 webhook URL 配置到 Gitlab 中;

2. 开启 Gitlab webhook 功能，Admin Area -> Settings -> Network，勾选"Allow requests to the local network from web hooks and services";
3. 在 Gitlab 项目添加 webhook，点击项目-> Settings -> Integrations，填写 URL，勾选 Push events;
4. 在 Jeknkins 中点击 Manage Jenkins -> Configure System，取消勾选”Enable authentication for project end-point"。

## 参数化构建

可以选择字符串、选择框等参数，可以在 Pipeline 里使用 `${parameters}` 引用参数。

## 邮件发送

邮件模板：

```html
<!DOCTYPE html>
 <html>
 <head>
    <meta charset="UTF-8">
    <title>${ENV, var="JOB_NAME"}-第${BUILD_NUMBER}次构建日志</title>
 </head>
 <body leftmargin="8" marginwidth="0" topmargin="8" marginheight="4" offset="0">
 <table width="95%" cellpadding="0" cellspacing="0"
       style="font-size: 11pt; font-family: Tahoma, Arial, Helvetica, sans-serif">
    <tr>
        <td>(本邮件是程序自动下发的，请勿回复！)</td>
    </tr>
    <tr>
        <td><h2>
            <font color="#0000FF">构建结果 - ${BUILD_STATUS}</font>
        </h2></td>
    </tr>
    <tr>
        <td><br />
            <b><font color="#0B610B">构建信息</font></b>
            <hr size="2" width="100%" align="center" /></td>
    </tr>
    <tr>
        <td>
            <ul>
                <li>项目名称&nbsp;：&nbsp;${PROJECT_NAME}</li>
                <li>构建编号&nbsp;：&nbsp;第${BUILD_NUMBER}次构建</li>
                <li>触发原因：&nbsp;${CAUSE}</li>
                <li>构建日志：&nbsp;<a href="${BUILD_URL}console">${BUILD_URL}console</a></li>
                <li>构建&nbsp;&nbsp;Url&nbsp;：&nbsp;<a href="${BUILD_URL}">${BUILD_URL}</a></li>
                <li>工作目录&nbsp;：&nbsp;<a href="${PROJECT_URL}ws">${PROJECT_URL}ws</a></li>
                <li>项目&nbsp;&nbsp;Url&nbsp;：&nbsp;<a href="${PROJECT_URL}">${PROJECT_URL}</a></li>
            </ul>
        </td>
    </tr>
    <tr>
        <td><b><font color="#0B610B">Changes Since Last
            Successful Build:</font></b>
            <hr size="2" width="100%" align="center" /></td>
    </tr>
    <tr>
        <td>
            <ul>
                <li>历史变更记录 : <a href="${PROJECT_URL}changes">${PROJECT_URL}changes</a></li>
            </ul> ${CHANGES_SINCE_LAST_SUCCESS,reverse=true, format="Changes for Build #%n:<br />%c<br />",showPaths=true,changesFormat="<pre>[%a]<br/>%m</pre>",pathFormat="&nbsp;&nbsp;&nbsp;&nbsp;%p"}
        </td>
    </tr>
    <tr>
        <td><b>Failed Test Results</b>
            <hr size="2" width="100%" align="center" /></td>
    </tr>
    <tr>
        <td><pre
                style="font-size: 11pt; font-family: Tahoma, Arial, Helvetica, sans-serif">$FAILED_TESTS</pre>
            <br /></td>
    </tr>
    <tr>
        <td><b><font color="#0B610B">构建日志 (最后 100行):</font></b>
            <hr size="2" width="100%" align="center" /></td>
    </tr>
    <tr>
        <td><textarea cols="80" rows="30" readonly="readonly"
                      style="font-family: Courier New">${BUILD_LOG, maxLines=100}</textarea>
        </td>
    </tr>
 </table>
 </body>
 </html>
```

添加构建后邮件发送：

```groovy
 post {
     always {
         emailext(
             subject: '构建通知：${PROJECT_NAME} - Build # ${BUILD_NUMBER} - ${BUILD_STATUS}!',
             body: '${FILE,path="email.html"}',
             to: 'xxx@qq.com'
         )
}
```

# 基于 Kubernetes 实现持续集成

## Master-Slave 分布式构建

Jenkins 的 Master-Slave 分布式构建，就是通过将构建过程分配到从属 Slave 节点上，从而减轻 Master 节点的压力，而且可以同时构建多个，有点类似负载均衡的概念。

开启代理程序的 TCP 端口：Manage Jenkins -> Configure Global Security，开启代理。

使用 jnlp 协议在 Slave节点启动 java 进程连接 Master 节点。

## 持续集成方案

### 工作流程

手动 /自动构建 -> Jenkins 调度 K8S API -＞动态生成 Jenkins Slave pod -＞ Slave pod 拉取 Git 代码／编译／打包镜像 -＞推送到镜像仓库 Harbor -＞ Slave 工作完成，Pod 自动销毁 -＞部署 到测试或生产 Kubernetes平台。

### 先决条件

1. Kubernetes 集群；
2. 存储动态供应；

### 整合

1. 安装 Kubernetes 和 Kubernetes Continuous Deploy 插件；

2. Jenkins与Kubernetes整合：系统管理 -> 系统配置 -> 云 -> 新建云 -> Kubernetes

   > kubernetes 地址采用了kube的服务器发现： https://kubernetes.default.svc.cluster.local。
   >
   > namespace 填 kube-ops，然后点击 Test Connection，如果出现 Connection test successful 的提 示信息证明 Jenkins 已经可以和 Kubernetes 系统正常通信。
   >
   > Jenkins URL  地址： http://jenkins.kube-ops.svc.cluster.local:8080。

3. 构建 Jenkins-Slave 自定义镜像，官方推荐镜像：jenkins/jnlp-slave:latest；

4. 编写 Pipeline：

   ```grovvy
   def git_address = ""
   def git_auth = ""
    //构建版本的名称
   def tag = "latest"
    //Harbor私服地址
   def harbor_url = ""
    //Harbor的项目名称
   def harbor_project_name = ""
    //Harbor的凭证
   def harbor_auth = ""
   def deploy_image_name = "${harbor_url}/${harbor_project_name}/${imageName}"
   
   podTemplate(label: 'jenkins-slave', cloud: 'kubernetes', containers: [
       containerTemplate(
           name: 'jnlp', 
           image: "192.168.66.102:85/library/jenkins-slave-maven:latest"
       ),
       containerTemplate(
           name: 'docker', 
           image: "docker:stable",
           ttyEnabled: true,
           command: 'cat'
       ),
     ],
     volumes: [
       hostPathVolume(mountPath: '/var/run/docker.sock', hostPath: '/var/run/docker.sock'),
       nfsVolume(mountPath: '/usr/local/apache-maven/repo', serverAddress: '192.168.66.101' , serverPath: '/opt/nfs/maven'),
     ],
    ) 
   {
       node("jenkins-slave"){
           stage('拉取代码'){
               checkout([$class: 'GitSCM', branches: [[name: '${branch}']], 
               userRemoteConfigs: [[credentialsId: "${git_auth}", url: "${git_address}"]]])
         }
         
           stage('代码编译'){
               //编译并安装公共工程
               sh "mvn -f tensquare_common clean install" 
           }
         
           stage('构建镜像，部署项目'){
               //把选择的项目信息转为数组
               def selectedProjects = "${project_name}".split(',')
               
               for(int i=0;i<selectedProjects.size();i++){
                   //取出每个项目的名称和端口
                   def currentProject = selectedProjects[i];
                   //项目名称
                   def currentProjectName = currentProject.split('@')[0]
                   //项目启动端口
                   def currentProjectPort = currentProject.split('@')[1]
                    //定义镜像名称
                   def imageName = "${currentProjectName}:${tag}"
                    
                   //编译，构建本地镜像
                   sh "mvn -f ${currentProjectName} clean package dockerfile:build"
                   container('docker') {
                       //给镜像打标签
                       sh "docker tag ${imageName} ${harbor_url}/${harbor_project_name}/${imageName}"
                       //登录Harbor，并上传镜像
                       withCredentials([
                           usernamePassword(credentialsId: "${harbor_auth}", 
                           passwordVariable: 'password', 
                           usernameVariable: 'username')]) 
                       {
                           //登录
                           sh "docker login -u ${username} -p ${password} ${harbor_url}"
                           //上传镜像
                           sh "docker push ${harbor_url}/${harbor_project_name}/${imageName}"
                        }
                        //删除本地镜像
                        sh "docker rmi -f ${imageName}"
                        sh "docker rmi -f ${harbor_url}/${harbor_project_name}/${imageName}"
                   }
                   //部署到K8S
                   sh """
                   sed -i 's#\$IMAGE_NAME#${deploy_image_name}#' ${currentProjectName}/deploy.yml
                   sed -i 's#\$SECRET_NAME#${secret_name}#' ${currentProjectName}/deploy.yml
                   """
                   kubernetesDeploy configs: "${currentProjectName}/deploy.yml", kubeconfigId: "${k8s_auth}"
               }
           }
       }
   }
   ```

   5. 建立 Kubernetes 认证凭证，将 kubeconfig 复制到 Jenkins；
   6. Kubernetes 集群生成 Docker 凭证;
   7. 测试。