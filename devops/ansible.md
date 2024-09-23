# 架构

# ![ansible_1](../assets/ansible_1.png)

- Ansible：Ansible 核心程序；
- HostInventory：记录由 Ansible 管理的主机信息，包括端口、密码、ip 等；
- Playbooks：“剧本”YAML 格式文件，多个任务定义在一个文件中，定义主机需要调用哪些模块来完成的功能；
- CoreModules：核心模块，主要操作是通过调用核心模块来完成管理任务；
- CustomModules：自定义模块，完成核心模块无法完成的功能，支持多种语言；
- ConnectionPlugins：连接插件，Ansible 和 Host 通信使用。

# 任务执行

Ansible 系统由控制主机对被管节点的操作方式可分为两类，即 adhoc 和 playbook ：

- ad-hoc 模式（点对点模式）
  使用单个模块，支持批量执行单条命令。ad-hoc 命令是一种可以快速输入的命令，而且不需要保存起来的命令。就相当于 bash 中的一句话 shell。
- playbook 模式（剧本模式）
  是 Ansible 主要管理方式，也是 Ansible 功能强大的关键所在。playbook 通过多个 task 集合完成一类功能，如 Web 服务的安装部署、数据库服务器的批量备份等。可以简单地把 playbook 理解为通过组合多条ad-hoc 操作的配置文件。

![img](https://cdn.nlark.com/yuque/0/2023/png/25552087/1679486717683-0f868ae5-070b-4dda-8215-296fb9c05e32.png)

1. 加载自己的配置文件，默认`/etc/ansible/ansible.cfg`；
2. 查找对应的主机配置文件，找到要执行的主机或者组；
3. 加载自己对应的模块文件，如：command；
4. 通过 ansible 将模块或命令生成对应的临时 py 文件( python 脚本)， 并将该文件传输至远程服务器；
5. 对应执行用户的家目录的`.ansible/tmp/XXX/XXX.PY`文件；
6. 给文件 +x 执行权限；
7. 执行并返回结果；
8. 删除临时 py 文件，`sleep 0`退出；

# 配置文件

ansible 与我们其他的服务在这一点上有很大不同，这里的配置文件查找是从多个地方找的，顺序如下：

1. 检查环境变量 ANSIBLE_CONFIG 指向的路径文件(`export ANSIBLE_CONFIG=/etc/ansible.cfg`)；
2. `~/.ansible.cfg`，检查当前目录下的 ansible.cfg 配置文件；
3. `/etc/ansible.cfg`检查 etc 目录的配置文件。

ansible 的配置文件为`/etc/ansible/ansible.cfg`，ansible 有许多参数，下面我们列出一些常见的参数：

```shell
inventory = /etc/ansible/hosts		#这个参数表示资源清单inventory文件的位置
library = /usr/share/ansible		#指向存放Ansible模块的目录，支持多个目录方式，只要用冒号（：）隔开就可以
forks = 5		#并发连接数，默认为5
sudo_user = root		#设置默认执行命令的用户
remote_port = 22		#指定连接被管节点的管理端口，默认为22端口，建议修改，能够更加安全
host_key_checking = False		#设置是否检查SSH主机的密钥，值为True/False。关闭后第一次连接不会提示配置实例
timeout = 60		#设置SSH连接的超时时间，单位为秒
log_path = /var/log/ansible.log		#指定一个存储ansible日志的文件（默认不记录日志）
```

# Inventory 文件

Ansible 可同时操作属于一个组的多台主机,组和主机之间的关系通过 inventory 文件配置。默认的文件路径为 ：/etc/ansible/hosts 。除默认文件外，你还可以同时使用多个 inventory 文件，也可以从动态源，或云上拉取 inventory 配置信息。

## 主机与组

/etc/ansible/hosts 文件的格式：

```shell
mail.example.com

[webservers]
foo.example.com
bar.example.com

[dbservers]
one.example.com
two.example.com
three.example.com
```

方括号 [] 中是组名，用于对系统进行分类,便于对不同系统进行个别的管理。

一个系统可以属于不同的组，比如一台服务器可以同时属于 webserver 组 和 dbserver 组.这时属于两个组的变量都可以为这台主机所用。

如果有主机的 SSH 端口不是标准的22端口，可在主机名之后加上端口号，用冒号分隔。

```shell
badwolf.example.com:5309
```

一组相似的 hostname , 可简写如下:

```shell
[webservers]
www[01:50].example.com
```

对于每一个 host ，你还可以选择连接类型和连接用户名：

```shell
[targets]

localhost              ansible_connection=local
other1.example.com     ansible_connection=ssh        ansible_ssh_user=mpdehaan
other2.example.com     ansible_connection=ssh        ansible_ssh_user=mdehaan
```

## 主机变量

前面已经提到过，分配变量给主机很容易做到，这些变量定义后可在 playbooks 中使用：

```shell
[atlanta]
host1 http_port=80 maxRequestsPerChild=808
host2 http_port=303 maxRequestsPerChild=909
```

## 组的变量

```shell
[atlanta]
host1
host2

[atlanta:vars]
ntp_server=ntp.atlanta.example.com
proxy=proxy.atlanta.example.com
```

## 把一个组作为另一个组的子成员

```shell
[atlanta]
host1
host2

[raleigh]
host2
host3

[southeast:children]
atlanta
raleigh

[southeast:vars]
some_server=foo.southeast.example.com
halon_system_timeout=30
self_destruct_countdown=60
escape_pods=2

[usa:children]
southeast
northeast
southwest
northwest
```

## 分文件定义 Host 和 Group 变量

在 inventory 主文件中保存所有的变量并不是最佳的方式。还可以保存在独立的文件中，这些独立文件与 inventory 文件保持关联。

假设 inventory 文件的路径为：/etc/ansible/hosts

假设有一个主机名为  'foosball'， 主机同时属于两个组,一个是 'raleigh'， 另一个是 'webservers'。 那么以下配置文件( YAML 格式)中的变量可以为  'foosball' 主机所用.依次为 'raleigh' 的组变量，'webservers' 的组变量，'foosball'的主机变量:

```shell
/etc/ansible/group_vars/raleigh
/etc/ansible/group_vars/webservers
/etc/ansible/host_vars/foosball
```

## Inventory 参数的说明

```shell
ansible_ssh_host
      将要连接的远程主机名.与你想要设定的主机的别名不同的话,可通过此变量设置.

ansible_ssh_port
      ssh端口号.如果不是默认的端口号,通过此变量设置.

ansible_ssh_user
      默认的 ssh 用户名

ansible_ssh_pass
      ssh 密码(这种方式并不安全,我们强烈建议使用 --ask-pass 或 SSH 密钥)

ansible_sudo_pass
      sudo 密码(这种方式并不安全,我们强烈建议使用 --ask-sudo-pass)

ansible_sudo_exe (new in version 1.8)
      sudo 命令路径(适用于1.8及以上版本)

ansible_connection
      与主机的连接类型.比如:local, ssh 或者 paramiko. Ansible 1.2 以前默认使用 paramiko.1.2 以后默认使用 'smart','smart' 方式会根据是否支持 ControlPersist, 来判断'ssh' 方式是否可行.

ansible_ssh_private_key_file
      ssh 使用的私钥文件.适用于有多个密钥,而你不想使用 SSH 代理的情况.

ansible_shell_type
      目标系统的shell类型.默认情况下,命令的执行使用 'sh' 语法,可设置为 'csh' 或 'fish'.

ansible_python_interpreter
      目标主机的 python 路径.适用于的情况: 系统中有多个 Python, 或者命令路径不是"/usr/bin/python",比如  \*BSD, 或者 /usr/bin/python
      不是 2.X 版本的 Python.我们不使用 "/usr/bin/env" 机制,因为这要求远程用户的路径设置正确,且要求 "python" 可执行程序名不可为 python以外的名字(实际有可能名为python26).

      与 ansible_python_interpreter 的工作方式相同,可设定如 ruby 或 perl 的路径....
```

# 常用命令

`/usr/bin/ansible`：Ansibe AD-Hoc 临时命令执行工具，常用于临时命令的执行；
`/usr/bin/ansible-doc`：Ansible 模块功能查看工具；
`/usr/bin/ansible-galaxy`：下载/上传优秀代码或 Roles 模块 的官网平台，基于网络的；
`/usr/bin/ansible-playbook`：Ansible 定制自动化的任务集编排工具；
`/usr/bin/ansible-pull`：Ansible 远程执行命令的工具，拉取配置而非推送配置（使用较少，海量机器时使用，对运维的架构能力要求较高）；
`/usr/bin/ansible-vault`：Ansible 文件加密工具；
`/usr/bin/ansible-console`：Ansible 基于 Linux Consoble 界面可与用户交互的命令执行工具。

ansible 命令语法：`ansible <host-pattern> [-f forks][-m module_name][-a args]`

# 常用模块

<details class="lake-collapse"><summary id="ucac01ea4"><span class="ne-text">ping</span></summary><pre data-language="shell" id="HMJrT" class="ne-codeblock language-shell" style="border: 1px solid #e8e8e8; border-radius: 2px; background: #f9f9f9; padding: 16px; font-size: 13px; color: #595959"><code>[root@server ~]# ansible web -m ping
192.168.37.122 | SUCCESS =&gt; {
    "changed": false, 
    "ping": "pong"
}
192.168.37.133 | SUCCESS =&gt; {
    "changed": false, 
    "ping": "pong"
}</code></pre></details>

<details class="lake-collapse"><summary id="ud95c778c"><span class="ne-text">command</span></summary><pre data-language="shell" id="DDrZe" class="ne-codeblock language-shell" style="border: 1px solid #e8e8e8; border-radius: 2px; background: #f9f9f9; padding: 16px; font-size: 13px; color: #595959"><code>[root@server ~]# ansible web -m command -a 'ss -ntl'
192.168.37.122 | SUCCESS | rc=0 &gt;&gt;
State      Recv-Q Send-Q Local Address:Port               Peer Address:Port              
LISTEN     0      128          *:111                      *:*                  
LISTEN     0      5      192.168.122.1:53                       *:*                  
LISTEN     0      128          *:22                       *:*                  
LISTEN     0      128    127.0.0.1:631                      *:*                  
LISTEN     0      128          *:23000                    *:*                  
LISTEN     0      100    127.0.0.1:25                       *:*                  
LISTEN     0      128         :::111                     :::*                  
LISTEN     0      128         :::22                      :::*                  
LISTEN     0      128        ::1:631                     :::*                  
LISTEN     0      100        ::1:25                      :::*                  

192.168.37.133 | SUCCESS | rc=0 &gt;&gt;
State      Recv-Q Send-Q Local Address:Port               Peer Address:Port              
LISTEN     0      128          *:111                      *:*                  
LISTEN     0      128          *:22                       *:*                  
LISTEN     0      128    127.0.0.1:631                      *:*                  
LISTEN     0      128          *:23000                    *:*                  
LISTEN     0      100    127.0.0.1:25                       *:*                  
LISTEN     0      128         :::111                     :::*                  
LISTEN     0      128         :::22                      :::*                  
LISTEN     0      128        ::1:631                     :::*                  
LISTEN     0      100        ::1:25                      :::*  </code></pre><p id="u3eb89011" class="ne-p" style="margin: 0; padding: 0; min-height: 24px"><span class="ne-text" style="color: rgb(0, 0, 0); font-size: 14px">命</span><span class="ne-text" style="color: rgb(0, 0, 0)">令模块接受命令名称，后面是空格分隔的列表参数。给定的命令将在所有选定的节点上执行。它不会通过 shell 进行处理，比如 $HOME 和操作如"&lt;"，"&gt;"，"|"，";"，"&amp;" 工作（需要使用（shell）模块实现这些功能）。</span><span class="ne-text">注意，该命令不支持 </span><span class="ne-text" style="color: rgb(0, 0, 0)">"|"</span><span class="ne-text">。</span></p></details>

<details class="lake-collapse"><summary id="u23021e84"><span class="ne-text">shell</span></summary><pre data-language="shell" id="zfwBq" class="ne-codeblock language-shell" style="border: 1px solid #e8e8e8; border-radius: 2px; background: #f9f9f9; padding: 16px; font-size: 13px; color: #595959"><code>[root@server ~]# ansible web -m shell -a 'cat /etc/passwd |grep "keer"'
192.168.37.122 | SUCCESS | rc=0 &gt;&gt;
keer:x:10001:1000:keer:/home/keer:/bin/sh

192.168.37.133 | SUCCESS | rc=0 &gt;&gt;
keer:x:10001:10001::/home/keer:/bin/sh</code></pre></details>

<details class="lake-collapse"><summary id="uc677a933"><span class="ne-text">copy</span></summary><p id="ua0193d58" class="ne-p" style="margin: 0; padding: 0; min-height: 24px"><span class="ne-text" style="color: rgb(0, 0, 0)">这个模块用于将文件复制到远程主机，同时支持给定内容生成文件和修改权限等。<br></span><span class="ne-text" style="color: rgb(0, 0, 0)">其相关选项如下：</span></p><p id="u3086e125" class="ne-p" style="margin: 0; padding: 0; min-height: 24px"><code class="ne-code" style="font-family: SFMono-Regular, Consolas, Liberation Mono, Menlo, Courier, monospace; background-color: rgba(0, 0, 0, 0.06); border: 1px solid rgba(0, 0, 0, 0.08); border-radius: 2px; padding: 0px 2px"><span class="ne-text">src</span></code><span class="ne-text" style="color: rgb(85, 85, 85)">：被复制到远程主机的本地文件。可以是绝对路径，也可以是相对路径。如果路径是一个目录，则会递归复制，用法类似于 "rsync"；<br></span><code class="ne-code" style="font-family: SFMono-Regular, Consolas, Liberation Mono, Menlo, Courier, monospace; background-color: rgba(0, 0, 0, 0.06); border: 1px solid rgba(0, 0, 0, 0.08); border-radius: 2px; padding: 0px 2px"><span class="ne-text">content</span></code><span class="ne-text" style="color: rgb(85, 85, 85)">：用于替换 "src"，可以直接指定文件的值；<br></span><code class="ne-code" style="font-family: SFMono-Regular, Consolas, Liberation Mono, Menlo, Courier, monospace; background-color: rgba(0, 0, 0, 0.06); border: 1px solid rgba(0, 0, 0, 0.08); border-radius: 2px; padding: 0px 2px"><span class="ne-text" style="background-color: rgb(246, 246, 246)">dest</span></code><span class="ne-text" style="color: rgb(85, 85, 85)">：必选项，将源文件复制到的远程主机的绝对路径；<br></span><code class="ne-code" style="font-family: SFMono-Regular, Consolas, Liberation Mono, Menlo, Courier, monospace; background-color: rgba(0, 0, 0, 0.06); border: 1px solid rgba(0, 0, 0, 0.08); border-radius: 2px; padding: 0px 2px"><span class="ne-text" style="background-color: rgb(246, 246, 246)">backup</span></code><span class="ne-text" style="color: rgb(85, 85, 85)">：当文件内容发生改变后，在覆盖之前把源文件备份，备份文件包含时间信息；<br></span><code class="ne-code" style="font-family: SFMono-Regular, Consolas, Liberation Mono, Menlo, Courier, monospace; background-color: rgba(0, 0, 0, 0.06); border: 1px solid rgba(0, 0, 0, 0.08); border-radius: 2px; padding: 0px 2px"><span class="ne-text" style="background-color: rgb(246, 246, 246)">directory_mode</span></code><span class="ne-text" style="color: rgb(85, 85, 85)">：递归设定目录的权限，默认为系统默认权限；<br></span><code class="ne-code" style="font-family: SFMono-Regular, Consolas, Liberation Mono, Menlo, Courier, monospace; background-color: rgba(0, 0, 0, 0.06); border: 1px solid rgba(0, 0, 0, 0.08); border-radius: 2px; padding: 0px 2px"><span class="ne-text" style="background-color: rgb(246, 246, 246)">force</span></code><span class="ne-text" style="color: rgb(85, 85, 85)">：当目标主机包含该文件，但内容不同时，设为 "yes"，表示强制覆盖；设为 "no"，表示目标主机的目标位置不存在该文件才复制，默认为 "yes"；<br></span><code class="ne-code" style="font-family: SFMono-Regular, Consolas, Liberation Mono, Menlo, Courier, monospace; background-color: rgba(0, 0, 0, 0.06); border: 1px solid rgba(0, 0, 0, 0.08); border-radius: 2px; padding: 0px 2px"><span class="ne-text" style="background-color: rgb(246, 246, 246)">others</span></code><span class="ne-text" style="color: rgb(85, 85, 85)">：所有的 file 模块中的选项可以在这里使用。</span></p><pre data-language="shell" id="dB1bp" class="ne-codeblock language-shell" style="border: 1px solid #e8e8e8; border-radius: 2px; background: #f9f9f9; padding: 16px; font-size: 13px; color: #595959"><code>[root@server ~]# ansible web -m copy -a 'src=~/hello dest=/data/hello' 
192.168.37.122 | SUCCESS =&gt; {
    "changed": true, 
    "checksum": "22596363b3de40b06f981fb85d82312e8c0ed511", 
    "dest": "/data/hello", 
    "gid": 0, 
    "group": "root", 
    "md5sum": "6f5902ac237024bdd0c176cb93063dc4", 
    "mode": "0644", 
    "owner": "root", 
    "size": 12, 
    "src": "/root/.ansible/tmp/ansible-tmp-1512437093.55-228281064292921/source", 
    "state": "file", 
    "uid": 0
}
192.168.37.133 | SUCCESS =&gt; {
    "changed": true, 
    "checksum": "22596363b3de40b06f981fb85d82312e8c0ed511", 
    "dest": "/data/hello", 
    "gid": 0, 
    "group": "root", 
    "md5sum": "6f5902ac237024bdd0c176cb93063dc4", 
    "mode": "0644", 
    "owner": "root", 
    "size": 12, 
    "src": "/root/.ansible/tmp/ansible-tmp-1512437093.74-44694985235189/source", 
    "state": "file", 
    "uid": 0
}</code></pre></details>

<details class="lake-collapse"><summary id="u69368f90"><span class="ne-text">file</span></summary><p id="u6f0d2e85" class="ne-p" style="margin: 0; padding: 0; min-height: 24px"><span class="ne-text">该模块主要用于设置文件的属性，比如创建文件、创建链接文件、删除文件等。<br></span><span class="ne-text">下面是一些常见的命令：</span></p><p id="u7fd4a6a2" class="ne-p" style="margin: 0; padding: 0; min-height: 24px"><code class="ne-code" style="font-family: SFMono-Regular, Consolas, Liberation Mono, Menlo, Courier, monospace; background-color: rgba(0, 0, 0, 0.06); border: 1px solid rgba(0, 0, 0, 0.08); border-radius: 2px; padding: 0px 2px"><span class="ne-text">force</span></code><span class="ne-text"> ：需要在两种情况下强制创建软链接，一种是源文件不存在，但之后会建立的情况下；另一种是目标软链接已存在，需要先取消之前的软链，然后创建新的软链，有两个选项：yes|no；<br></span><code class="ne-code" style="font-family: SFMono-Regular, Consolas, Liberation Mono, Menlo, Courier, monospace; background-color: rgba(0, 0, 0, 0.06); border: 1px solid rgba(0, 0, 0, 0.08); border-radius: 2px; padding: 0px 2px"><span class="ne-text">group</span></code><span class="ne-text">：定义文件/目录的属组。后面可以加上 mode：定义文件/目录的权限；<br></span><code class="ne-code" style="font-family: SFMono-Regular, Consolas, Liberation Mono, Menlo, Courier, monospace; background-color: rgba(0, 0, 0, 0.06); border: 1px solid rgba(0, 0, 0, 0.08); border-radius: 2px; padding: 0px 2px"><span class="ne-text">owner</span></code><span class="ne-text">：定义文件/目录的属主。后面必须跟上 path：定义文件/目录的路径；<br></span><code class="ne-code" style="font-family: SFMono-Regular, Consolas, Liberation Mono, Menlo, Courier, monospace; background-color: rgba(0, 0, 0, 0.06); border: 1px solid rgba(0, 0, 0, 0.08); border-radius: 2px; padding: 0px 2px"><span class="ne-text">recurse</span></code><span class="ne-text">：递归设置文件的属性，只对目录有效，后面跟上 src：被链接的源文件路径，只应用于state=link 的情况；<br></span><code class="ne-code" style="font-family: SFMono-Regular, Consolas, Liberation Mono, Menlo, Courier, monospace; background-color: rgba(0, 0, 0, 0.06); border: 1px solid rgba(0, 0, 0, 0.08); border-radius: 2px; padding: 0px 2px"><span class="ne-text">dest</span></code><span class="ne-text">：被链接到的路径，只应用于 state=link 的情况；<br></span><code class="ne-code" style="font-family: SFMono-Regular, Consolas, Liberation Mono, Menlo, Courier, monospace; background-color: rgba(0, 0, 0, 0.06); border: 1px solid rgba(0, 0, 0, 0.08); border-radius: 2px; padding: 0px 2px"><span class="ne-text">state</span></code><span class="ne-text">：状态，有以下选项：</span></p><ul class="ne-ul" style="margin: 0; padding-left: 23px"><li id="u9d0e6b21" data-lake-index-type="0"><code class="ne-code" style="font-family: SFMono-Regular, Consolas, Liberation Mono, Menlo, Courier, monospace; background-color: rgba(0, 0, 0, 0.06); border: 1px solid rgba(0, 0, 0, 0.08); border-radius: 2px; padding: 0px 2px"><span class="ne-text">directory</span></code><span class="ne-text">：如果目录不存在，就创建目录</span></li><li id="ub4e2f979" data-lake-index-type="0"><code class="ne-code" style="font-family: SFMono-Regular, Consolas, Liberation Mono, Menlo, Courier, monospace; background-color: rgba(0, 0, 0, 0.06); border: 1px solid rgba(0, 0, 0, 0.08); border-radius: 2px; padding: 0px 2px"><span class="ne-text">file</span></code><span class="ne-text">：即使文件不存在，也不会被创建</span></li><li id="u944f751e" data-lake-index-type="0"><code class="ne-code" style="font-family: SFMono-Regular, Consolas, Liberation Mono, Menlo, Courier, monospace; background-color: rgba(0, 0, 0, 0.06); border: 1px solid rgba(0, 0, 0, 0.08); border-radius: 2px; padding: 0px 2px"><span class="ne-text">link</span></code><span class="ne-text">：创建软链接</span></li><li id="ucf7442e1" data-lake-index-type="0"><code class="ne-code" style="font-family: SFMono-Regular, Consolas, Liberation Mono, Menlo, Courier, monospace; background-color: rgba(0, 0, 0, 0.06); border: 1px solid rgba(0, 0, 0, 0.08); border-radius: 2px; padding: 0px 2px"><span class="ne-text">hard</span></code><span class="ne-text">：创建硬链接</span></li><li id="u7c386dbe" data-lake-index-type="0"><code class="ne-code" style="font-family: SFMono-Regular, Consolas, Liberation Mono, Menlo, Courier, monospace; background-color: rgba(0, 0, 0, 0.06); border: 1px solid rgba(0, 0, 0, 0.08); border-radius: 2px; padding: 0px 2px"><span class="ne-text">touch</span></code><span class="ne-text">：如果文件不存在，则会创建一个新的文件，如果文件或目录已存在，则更新其最后修改时间</span></li><li id="u360302c7" data-lake-index-type="0"><code class="ne-code" style="font-family: SFMono-Regular, Consolas, Liberation Mono, Menlo, Courier, monospace; background-color: rgba(0, 0, 0, 0.06); border: 1px solid rgba(0, 0, 0, 0.08); border-radius: 2px; padding: 0px 2px"><span class="ne-text">absent</span></code><span class="ne-text">：删除目录、文件或者取消链接文件</span></li></ul><pre data-language="shell" id="Jp1mH" class="ne-codeblock language-shell" style="border: 1px solid #e8e8e8; border-radius: 2px; background: #f9f9f9; padding: 16px; font-size: 13px; color: #595959"><code>[root@server ~]# ansible web -m file -a 'path=/data/app state=directory'
192.168.37.122 | SUCCESS =&gt; {
    "changed": true, 
    "gid": 0, 
    "group": "root", 
    "mode": "0755", 
    "owner": "root", 
    "path": "/data/app", 
    "size": 6, 
    "state": "directory", 
    "uid": 0
}
192.168.37.133 | SUCCESS =&gt; {
    "changed": true, 
    "gid": 0, 
    "group": "root", 
    "mode": "0755", 
    "owner": "root", 
    "path": "/data/app", 
    "size": 4096, 
    "state": "directory", 
    "uid": 0
}</code></pre></details>

<details class="lake-collapse"><summary id="u46b0f3d3"><span class="ne-text">fetch</span></summary><p id="ubee50ac1" class="ne-p" style="margin: 0; padding: 0; min-height: 24px"><span class="ne-text">该模块用于从远程某主机获取（复制）文件到本地。</span></p><pre data-language="shell" id="RXePA" class="ne-codeblock language-shell" style="border: 1px solid #e8e8e8; border-radius: 2px; background: #f9f9f9; padding: 16px; font-size: 13px; color: #595959"><code>[root@server ~]# ansible web -m fetch -a 'src=/data/hello dest=/data'  
192.168.37.122 | SUCCESS =&gt; {
    "changed": true, 
    "checksum": "22596363b3de40b06f981fb85d82312e8c0ed511", 
    "dest": "/data/192.168.37.122/data/hello", 
    "md5sum": "6f5902ac237024bdd0c176cb93063dc4", 
    "remote_checksum": "22596363b3de40b06f981fb85d82312e8c0ed511", 
    "remote_md5sum": null
}
192.168.37.133 | SUCCESS =&gt; {
    "changed": true, 
    "checksum": "22596363b3de40b06f981fb85d82312e8c0ed511", 
    "dest": "/data/192.168.37.133/data/hello", 
    "md5sum": "6f5902ac237024bdd0c176cb93063dc4", 
    "remote_checksum": "22596363b3de40b06f981fb85d82312e8c0ed511", 
    "remote_md5sum": null
}</code></pre></details>

<details class="lake-collapse"><summary id="u52d13589"><span class="ne-text">cron</span></summary><p id="u5aaac693" class="ne-p" style="margin: 0; padding: 0; min-height: 24px"><span class="ne-text">该模块适用于管理 cron 计划任务的，其使用的语法跟我们的 crontab 文件中的语法一致。</span></p><p id="uae1c9bdd" class="ne-p" style="margin: 0; padding: 0; min-height: 24px"><code class="ne-code" style="font-family: SFMono-Regular, Consolas, Liberation Mono, Menlo, Courier, monospace; background-color: rgba(0, 0, 0, 0.06); border: 1px solid rgba(0, 0, 0, 0.08); border-radius: 2px; padding: 0px 2px"><span class="ne-text">day</span></code><span class="ne-text">：日应该运行的工作( 1-31, *, */2, )；<br></span><code class="ne-code" style="font-family: SFMono-Regular, Consolas, Liberation Mono, Menlo, Courier, monospace; background-color: rgba(0, 0, 0, 0.06); border: 1px solid rgba(0, 0, 0, 0.08); border-radius: 2px; padding: 0px 2px"><span class="ne-text">hour</span></code><span class="ne-text">：小时 ( 0-23, *, */2, )；<br></span><code class="ne-code" style="font-family: SFMono-Regular, Consolas, Liberation Mono, Menlo, Courier, monospace; background-color: rgba(0, 0, 0, 0.06); border: 1px solid rgba(0, 0, 0, 0.08); border-radius: 2px; padding: 0px 2px"><span class="ne-text">minute</span></code><span class="ne-text">：分钟( 0-59, *, */2, )；<br></span><code class="ne-code" style="font-family: SFMono-Regular, Consolas, Liberation Mono, Menlo, Courier, monospace; background-color: rgba(0, 0, 0, 0.06); border: 1px solid rgba(0, 0, 0, 0.08); border-radius: 2px; padding: 0px 2px"><span class="ne-text">month</span></code><span class="ne-text">：月( 1-12, *, /2, )；<br></span><code class="ne-code" style="font-family: SFMono-Regular, Consolas, Liberation Mono, Menlo, Courier, monospace; background-color: rgba(0, 0, 0, 0.06); border: 1px solid rgba(0, 0, 0, 0.08); border-radius: 2px; padding: 0px 2px"><span class="ne-text">weekday</span></code><span class="ne-text">：周 ( 0-6 for Sunday-Saturday,, )；<br></span><code class="ne-code" style="font-family: SFMono-Regular, Consolas, Liberation Mono, Menlo, Courier, monospace; background-color: rgba(0, 0, 0, 0.06); border: 1px solid rgba(0, 0, 0, 0.08); border-radius: 2px; padding: 0px 2px"><span class="ne-text">job</span></code><span class="ne-text">：指明运行的命令是什么；<br></span><code class="ne-code" style="font-family: SFMono-Regular, Consolas, Liberation Mono, Menlo, Courier, monospace; background-color: rgba(0, 0, 0, 0.06); border: 1px solid rgba(0, 0, 0, 0.08); border-radius: 2px; padding: 0px 2px"><span class="ne-text">name</span></code><span class="ne-text">：定时任务描述；<br></span><code class="ne-code" style="font-family: SFMono-Regular, Consolas, Liberation Mono, Menlo, Courier, monospace; background-color: rgba(0, 0, 0, 0.06); border: 1px solid rgba(0, 0, 0, 0.08); border-radius: 2px; padding: 0px 2px"><span class="ne-text">reboot</span></code><span class="ne-text">：任务在重启时运行，不建议使用，建议使用special_time；<br></span><code class="ne-code" style="font-family: SFMono-Regular, Consolas, Liberation Mono, Menlo, Courier, monospace; background-color: rgba(0, 0, 0, 0.06); border: 1px solid rgba(0, 0, 0, 0.08); border-radius: 2px; padding: 0px 2px"><span class="ne-text">special_time</span></code><span class="ne-text">：特殊的时间范围，参数：reboot（重启时），annually（每年），monthly（每月），weekly（每周），daily（每天），hourly（每小时）；<br></span><code class="ne-code" style="font-family: SFMono-Regular, Consolas, Liberation Mono, Menlo, Courier, monospace; background-color: rgba(0, 0, 0, 0.06); border: 1px solid rgba(0, 0, 0, 0.08); border-radius: 2px; padding: 0px 2px"><span class="ne-text">state</span></code><span class="ne-text">：指定状态，present表示添加定时任务，也是默认设置，absent表示删除定时任务；<br></span><code class="ne-code" style="font-family: SFMono-Regular, Consolas, Liberation Mono, Menlo, Courier, monospace; background-color: rgba(0, 0, 0, 0.06); border: 1px solid rgba(0, 0, 0, 0.08); border-radius: 2px; padding: 0px 2px"><span class="ne-text">user</span></code><span class="ne-text">：以哪个用户的身份执行。</span></p><pre data-language="shell" id="GDjzI" class="ne-codeblock language-shell" style="border: 1px solid #e8e8e8; border-radius: 2px; background: #f9f9f9; padding: 16px; font-size: 13px; color: #595959"><code>[root@server ~]# ansible web -m cron -a 'name="ntp update every 5 min" minute=*/5 job="/sbin/ntpdate 172.17.0.1 &amp;&gt; /dev/null"'
192.168.37.122 | SUCCESS =&gt; {
    "changed": true, 
    "envs": [], 
    "jobs": [
        "ntp update every 5 min"
    ]
}
192.168.37.133 | SUCCESS =&gt; {
    "changed": true, 
    "envs": [], 
    "jobs": [
        "ntp update every 5 min"
    ]
}</code></pre></details>

<details class="lake-collapse"><summary id="u1491d61b"><span class="ne-text">yum</span></summary><p id="uffa60202" class="ne-p" style="margin: 0; padding: 0; min-height: 24px"><span class="ne-text" style="color: rgb(0, 0, 0); font-size: 14px">该模块主要用于软件的安装。</span></p><p id="ufb75fa01" class="ne-p" style="margin: 0; padding: 0; min-height: 24px"><code class="ne-code" style="font-family: SFMono-Regular, Consolas, Liberation Mono, Menlo, Courier, monospace; background-color: rgba(0, 0, 0, 0.06); border: 1px solid rgba(0, 0, 0, 0.08); border-radius: 2px; padding: 0px 2px"><span class="ne-text">name</span></code><span class="ne-text">：所安装的包的名称；<br></span><code class="ne-code" style="font-family: SFMono-Regular, Consolas, Liberation Mono, Menlo, Courier, monospace; background-color: rgba(0, 0, 0, 0.06); border: 1px solid rgba(0, 0, 0, 0.08); border-radius: 2px; padding: 0px 2px"><span class="ne-text">state</span></code><span class="ne-text">：present---&gt;安装， latest---&gt;安装最新的, absent---&gt; 卸载软件；<br></span><code class="ne-code" style="font-family: SFMono-Regular, Consolas, Liberation Mono, Menlo, Courier, monospace; background-color: rgba(0, 0, 0, 0.06); border: 1px solid rgba(0, 0, 0, 0.08); border-radius: 2px; padding: 0px 2px"><span class="ne-text">update_cache</span></code><span class="ne-text">：强制更新 yum 的缓存；<br></span><code class="ne-code" style="font-family: SFMono-Regular, Consolas, Liberation Mono, Menlo, Courier, monospace; background-color: rgba(0, 0, 0, 0.06); border: 1px solid rgba(0, 0, 0, 0.08); border-radius: 2px; padding: 0px 2px"><span class="ne-text">conf_file</span></code><span class="ne-text">：指定远程 yum 安装时所依赖的配置文件（安装本地已有的包）；<br></span><code class="ne-code" style="font-family: SFMono-Regular, Consolas, Liberation Mono, Menlo, Courier, monospace; background-color: rgba(0, 0, 0, 0.06); border: 1px solid rgba(0, 0, 0, 0.08); border-radius: 2px; padding: 0px 2px"><span class="ne-text">disable_pgp_check</span></code><span class="ne-text">：是否禁止 GPG checking，只用于 presentor latest；<br></span><code class="ne-code" style="font-family: SFMono-Regular, Consolas, Liberation Mono, Menlo, Courier, monospace; background-color: rgba(0, 0, 0, 0.06); border: 1px solid rgba(0, 0, 0, 0.08); border-radius: 2px; padding: 0px 2px"><span class="ne-text">disablerepo</span></code><span class="ne-text">：临时禁止使用 yum 库。 只用于安装或更新时；<br></span><code class="ne-code" style="font-family: SFMono-Regular, Consolas, Liberation Mono, Menlo, Courier, monospace; background-color: rgba(0, 0, 0, 0.06); border: 1px solid rgba(0, 0, 0, 0.08); border-radius: 2px; padding: 0px 2px"><span class="ne-text">enablerepo</span></code><span class="ne-text">：临时使用的 yum 库。只用于安装或更新时。</span></p><pre data-language="shell" id="b6Svn" class="ne-codeblock language-shell" style="border: 1px solid #e8e8e8; border-radius: 2px; background: #f9f9f9; padding: 16px; font-size: 13px; color: #595959"><code>[root@server ~]# ansible web -m yum -a 'name=htop state=present'
192.168.37.122 | SUCCESS =&gt; {
    "changed": true, 
    "msg": "", 
    "rc": 0, 
    "results": [
        "Loaded plugins: fastestmirror, langpacks\nLoading mirror speeds from cached hostfile\nResolving Dependencies\n--&gt; Running transaction check\n---&gt; Package htop.x86_64 0:2.0.2-1.el7 will be installed\n--&gt; Finished Dependency Resolution\n\nDependencies Resolved\n\n================================================================================\n Package         Arch              Version                Repository       Size\n================================================================================\nInstalling:\n htop            x86_64            2.0.2-1.el7            epel             98 k\n\nTransaction Summary\n================================================================================\nInstall  1 Package\n\nTotal download size: 98 k\nInstalled size: 207 k\nDownloading packages:\nRunning transaction check\nRunning transaction test\nTransaction test succeeded\nRunning transaction\n  Installing : htop-2.0.2-1.el7.x86_64                                      1/1 \n  Verifying  : htop-2.0.2-1.el7.x86_64                                      1/1 \n\nInstalled:\n  htop.x86_64 0:2.0.2-1.el7                                                     \n\nComplete!\n"
    ]
}
192.168.37.133 | SUCCESS =&gt; {
    "changed": true, 
    "msg": "Warning: RPMDB altered outside of yum.\n** Found 3 pre-existing rpmdb problem(s), 'yum check' output follows:\nipa-client-4.4.0-12.el7.centos.x86_64 has installed conflicts freeipa-client: ipa-client-4.4.0-12.el7.centos.x86_64\nipa-client-common-4.4.0-12.el7.centos.noarch has installed conflicts freeipa-client-common: ipa-client-common-4.4.0-12.el7.centos.noarch\nipa-common-4.4.0-12.el7.centos.noarch has installed conflicts freeipa-common: ipa-common-4.4.0-12.el7.centos.noarch\n", 
    "rc": 0, 
    "results": [
        "Loaded plugins: fastestmirror, langpacks\nLoading mirror speeds from cached hostfile\nResolving Dependencies\n--&gt; Running transaction check\n---&gt; Package htop.x86_64 0:2.0.2-1.el7 will be installed\n--&gt; Finished Dependency Resolution\n\nDependencies Resolved\n\n================================================================================\n Package         Arch              Version                Repository       Size\n================================================================================\nInstalling:\n htop            x86_64            2.0.2-1.el7            epel             98 k\n\nTransaction Summary\n================================================================================\nInstall  1 Package\n\nTotal download size: 98 k\nInstalled size: 207 k\nDownloading packages:\nRunning transaction check\nRunning transaction test\nTransaction test succeeded\nRunning transaction\n  Installing : htop-2.0.2-1.el7.x86_64                                      1/1 \n  Verifying  : htop-2.0.2-1.el7.x86_64                                      1/1 \n\nInstalled:\n  htop.x86_64 0:2.0.2-1.el7                                                     \n\nComplete!\n"
    ]
}</code></pre></details>

<details class="lake-collapse"><summary id="u8ebb8609"><span class="ne-text">service</span></summary><p id="u4f74b5a8" class="ne-p" style="margin: 0; padding: 0; min-height: 24px"><span class="ne-text">该模块用于服务程序的管理。</span></p><p id="uf175657b" class="ne-p" style="margin: 0; padding: 0; min-height: 24px"><code class="ne-code" style="font-family: SFMono-Regular, Consolas, Liberation Mono, Menlo, Courier, monospace; background-color: rgba(0, 0, 0, 0.06); border: 1px solid rgba(0, 0, 0, 0.08); border-radius: 2px; padding: 0px 2px"><span class="ne-text">arguments</span></code><span class="ne-text">：命令行提供额外的参数；<br></span><code class="ne-code" style="font-family: SFMono-Regular, Consolas, Liberation Mono, Menlo, Courier, monospace; background-color: rgba(0, 0, 0, 0.06); border: 1px solid rgba(0, 0, 0, 0.08); border-radius: 2px; padding: 0px 2px"><span class="ne-text">enabled</span></code><span class="ne-text">：设置开机启动；<br></span><code class="ne-code" style="font-family: SFMono-Regular, Consolas, Liberation Mono, Menlo, Courier, monospace; background-color: rgba(0, 0, 0, 0.06); border: 1px solid rgba(0, 0, 0, 0.08); border-radius: 2px; padding: 0px 2px"><span class="ne-text">name</span></code><span class="ne-text">：服务名称；<br></span><code class="ne-code" style="font-family: SFMono-Regular, Consolas, Liberation Mono, Menlo, Courier, monospace; background-color: rgba(0, 0, 0, 0.06); border: 1px solid rgba(0, 0, 0, 0.08); border-radius: 2px; padding: 0px 2px"><span class="ne-text">runlevel</span></code><span class="ne-text">：开机启动的级别，一般不用指定；<br></span><code class="ne-code" style="font-family: SFMono-Regular, Consolas, Liberation Mono, Menlo, Courier, monospace; background-color: rgba(0, 0, 0, 0.06); border: 1px solid rgba(0, 0, 0, 0.08); border-radius: 2px; padding: 0px 2px"><span class="ne-text">sleep</span></code><span class="ne-text">：在重启服务的过程中，是否等待。如在服务关闭以后等待2秒再启动(定义在剧本中)；<br></span><code class="ne-code" style="font-family: SFMono-Regular, Consolas, Liberation Mono, Menlo, Courier, monospace; background-color: rgba(0, 0, 0, 0.06); border: 1px solid rgba(0, 0, 0, 0.08); border-radius: 2px; padding: 0px 2px"><span class="ne-text">state</span></code><span class="ne-text"> ：有四种状态，分别为：started---&gt;启动服务， stopped---&gt;停止服务， restarted---&gt;重启服务， reloaded---&gt;重载配置。</span></p><pre data-language="shell" id="x51i8" class="ne-codeblock language-shell" style="border: 1px solid #e8e8e8; border-radius: 2px; background: #f9f9f9; padding: 16px; font-size: 13px; color: #595959"><code>[root@server ~]# ansible web -m service -a 'name=nginx state=started enabled=true' 
192.168.37.122 | SUCCESS =&gt; {
    "changed": true, 
    "enabled": true, 
    "name": "nginx", 
    "state": "started", 
    ……
}
192.168.37.133 | SUCCESS =&gt; {
    "changed": true, 
    "enabled": true, 
    "name": "nginx", 
    "state": "started", 
    ……
}</code></pre></details>

<details class="lake-collapse"><summary id="u0022c491"><span class="ne-text">user</span></summary><p id="u506fe57b" class="ne-p" style="margin: 0; padding: 0; min-height: 24px"><span class="ne-text">该模块主要是用来管理用户账号。</span></p><p id="u56675e4a" class="ne-p" style="margin: 0; padding: 0; min-height: 24px"><code class="ne-code" style="font-family: SFMono-Regular, Consolas, Liberation Mono, Menlo, Courier, monospace; background-color: rgba(0, 0, 0, 0.06); border: 1px solid rgba(0, 0, 0, 0.08); border-radius: 2px; padding: 0px 2px"><span class="ne-text">comment</span></code><span class="ne-text">： 用户的描述信息；<br></span><code class="ne-code" style="font-family: SFMono-Regular, Consolas, Liberation Mono, Menlo, Courier, monospace; background-color: rgba(0, 0, 0, 0.06); border: 1px solid rgba(0, 0, 0, 0.08); border-radius: 2px; padding: 0px 2px"><span class="ne-text">createhome</span></code><span class="ne-text">：是否创建家目录；<br></span><code class="ne-code" style="font-family: SFMono-Regular, Consolas, Liberation Mono, Menlo, Courier, monospace; background-color: rgba(0, 0, 0, 0.06); border: 1px solid rgba(0, 0, 0, 0.08); border-radius: 2px; padding: 0px 2px"><span class="ne-text">force</span></code><span class="ne-text">：在使用 state=absent 时, 行为与 userdel –force 一致；<br></span><code class="ne-code" style="font-family: SFMono-Regular, Consolas, Liberation Mono, Menlo, Courier, monospace; background-color: rgba(0, 0, 0, 0.06); border: 1px solid rgba(0, 0, 0, 0.08); border-radius: 2px; padding: 0px 2px"><span class="ne-text">group</span></code><span class="ne-text">：指定基本组；<br></span><code class="ne-code" style="font-family: SFMono-Regular, Consolas, Liberation Mono, Menlo, Courier, monospace; background-color: rgba(0, 0, 0, 0.06); border: 1px solid rgba(0, 0, 0, 0.08); border-radius: 2px; padding: 0px 2px"><span class="ne-text">groups</span></code><span class="ne-text">：指定附加组，如果指定为(groups=)表示删除所有组；<br></span><code class="ne-code" style="font-family: SFMono-Regular, Consolas, Liberation Mono, Menlo, Courier, monospace; background-color: rgba(0, 0, 0, 0.06); border: 1px solid rgba(0, 0, 0, 0.08); border-radius: 2px; padding: 0px 2px"><span class="ne-text">home</span></code><span class="ne-text">：指定用户家目录；<br></span><code class="ne-code" style="font-family: SFMono-Regular, Consolas, Liberation Mono, Menlo, Courier, monospace; background-color: rgba(0, 0, 0, 0.06); border: 1px solid rgba(0, 0, 0, 0.08); border-radius: 2px; padding: 0px 2px"><span class="ne-text">move_home</span></code><span class="ne-text">：如果设置为 home= 时, 试图将用户主目录移动到指定的目录；<br></span><code class="ne-code" style="font-family: SFMono-Regular, Consolas, Liberation Mono, Menlo, Courier, monospace; background-color: rgba(0, 0, 0, 0.06); border: 1px solid rgba(0, 0, 0, 0.08); border-radius: 2px; padding: 0px 2px"><span class="ne-text">name</span></code><span class="ne-text">：指定用户名；<br></span><code class="ne-code" style="font-family: SFMono-Regular, Consolas, Liberation Mono, Menlo, Courier, monospace; background-color: rgba(0, 0, 0, 0.06); border: 1px solid rgba(0, 0, 0, 0.08); border-radius: 2px; padding: 0px 2px"><span class="ne-text">non_unique</span></code><span class="ne-text">：该选项允许改变非唯一的用户 ID 值；<br></span><code class="ne-code" style="font-family: SFMono-Regular, Consolas, Liberation Mono, Menlo, Courier, monospace; background-color: rgba(0, 0, 0, 0.06); border: 1px solid rgba(0, 0, 0, 0.08); border-radius: 2px; padding: 0px 2px"><span class="ne-text">password</span></code><span class="ne-text">：指定用户密码；<br></span><code class="ne-code" style="font-family: SFMono-Regular, Consolas, Liberation Mono, Menlo, Courier, monospace; background-color: rgba(0, 0, 0, 0.06); border: 1px solid rgba(0, 0, 0, 0.08); border-radius: 2px; padding: 0px 2px"><span class="ne-text">remove</span></code><span class="ne-text">： 在使用 state=absent 时, 行为是与 userdel –remove 一致；<br></span><code class="ne-code" style="font-family: SFMono-Regular, Consolas, Liberation Mono, Menlo, Courier, monospace; background-color: rgba(0, 0, 0, 0.06); border: 1px solid rgba(0, 0, 0, 0.08); border-radius: 2px; padding: 0px 2px"><span class="ne-text">shell</span></code><span class="ne-text">： 指定默认 shell；<br></span><code class="ne-code" style="font-family: SFMono-Regular, Consolas, Liberation Mono, Menlo, Courier, monospace; background-color: rgba(0, 0, 0, 0.06); border: 1px solid rgba(0, 0, 0, 0.08); border-radius: 2px; padding: 0px 2px"><span class="ne-text">state</span></code><span class="ne-text">：设置帐号状态，不指定为创建，指定值为 absent 表示删除；<br></span><code class="ne-code" style="font-family: SFMono-Regular, Consolas, Liberation Mono, Menlo, Courier, monospace; background-color: rgba(0, 0, 0, 0.06); border: 1px solid rgba(0, 0, 0, 0.08); border-radius: 2px; padding: 0px 2px"><span class="ne-text">system</span></code><span class="ne-text">：当创建一个用户，设置这个用户是系统用户。这个设置不能更改现有用户；<br></span><code class="ne-code" style="font-family: SFMono-Regular, Consolas, Liberation Mono, Menlo, Courier, monospace; background-color: rgba(0, 0, 0, 0.06); border: 1px solid rgba(0, 0, 0, 0.08); border-radius: 2px; padding: 0px 2px"><span class="ne-text">uid</span></code><span class="ne-text">：指定用户的 uid。</span></p><pre data-language="shell" id="vLIjE" class="ne-codeblock language-shell" style="border: 1px solid #e8e8e8; border-radius: 2px; background: #f9f9f9; padding: 16px; font-size: 13px; color: #595959"><code>[root@server ~]# ansible web -m user -a 'name=keer uid=11111'
192.168.37.122 | SUCCESS =&gt; {
    "changed": true, 
    "comment": "", 
    "createhome": true, 
    "group": 11111, 
    "home": "/home/keer", 
    "name": "keer", 
    "shell": "/bin/bash", 
    "state": "present", 
    "stderr": "useradd: warning: the home directory already exists.\nNot copying any file from skel directory into it.\nCreating mailbox file: File exists\n", 
    "system": false, 
    "uid": 11111
}
192.168.37.133 | SUCCESS =&gt; {
    "changed": true, 
    "comment": "", 
    "createhome": true, 
    "group": 11111, 
    "home": "/home/keer", 
    "name": "keer", 
    "shell": "/bin/bash", 
    "state": "present", 
    "stderr": "useradd: warning: the home directory already exists.\nNot copying any file from skel directory into it.\nCreating mailbox file: File exists\n", 
    "system": false, 
    "uid": 11111
}</code></pre></details>

<details class="lake-collapse"><summary id="u54acd456"><span class="ne-text">group</span></summary><p id="u17080f02" class="ne-p" style="margin: 0; padding: 0; min-height: 24px"><span class="ne-text">该模块主要用于添加或删除组。</span></p><pre data-language="shell" id="gyRpD" class="ne-codeblock language-shell" style="border: 1px solid #e8e8e8; border-radius: 2px; background: #f9f9f9; padding: 16px; font-size: 13px; color: #595959"><code>[root@server ~]# ansible web -m group -a 'name=sanguo gid=12222'
192.168.37.122 | SUCCESS =&gt; {
    "changed": true, 
    "gid": 12222, 
    "name": "sanguo", 
    "state": "present", 
    "system": false
}
192.168.37.133 | SUCCESS =&gt; {
    "changed": true, 
    "gid": 12222, 
    "name": "sanguo", 
    "state": "present", 
    "system": false
}</code></pre></details>

<details class="lake-collapse"><summary id="u5a46e88b"><span class="ne-text">script</span></summary><p id="u5a121938" class="ne-p" style="margin: 0; padding: 0; min-height: 24px"><span class="ne-text">该模块用于将本机的脚本在被管理端的机器上运行。</span></p><pre data-language="shell" id="wpppB" class="ne-codeblock language-shell" style="border: 1px solid #e8e8e8; border-radius: 2px; background: #f9f9f9; padding: 16px; font-size: 13px; color: #595959"><code>[root@server ~]# vim /tmp/df.sh
	#!/bin/bash

	date &gt;&gt; /tmp/disk_total.log
	df -lh &gt;&gt; /tmp/disk_total.log 
[root@server ~]# chmod +x /tmp/df.sh 

[root@server ~]# ansible web -m script -a '/tmp/df.sh'
192.168.37.122 | SUCCESS =&gt; {
    "changed": true, 
    "rc": 0, 
    "stderr": "Shared connection to 192.168.37.122 closed.\r\n", 
    "stdout": "", 
    "stdout_lines": []
}
192.168.37.133 | SUCCESS =&gt; {
    "changed": true, 
    "rc": 0, 
    "stderr": "Shared connection to 192.168.37.133 closed.\r\n", 
    "stdout": "", 
    "stdout_lines": []
}</code></pre></details>

<details class="lake-collapse"><summary id="u6183489e"><span class="ne-text">setup</span></summary><p id="u87a15ed1" class="ne-p" style="margin: 0; padding: 0; min-height: 24px"><span class="ne-text">该模块主要用于收集信息，是通过调用 facts 组件来实现的。<br></span><span class="ne-text">facts 组件是 Ansible 用于采集被管机器设备信息的一个功能，我们可以使用 setup 模块查机器的所有facts 信息，可以使用 filter 来查看指定信息。整个 facts 信息被包装在一个 JSON 格式的数据结构中，ansible_facts 是最上层的值。<br></span><span class="ne-text">facts 就是变量，内建变量 。每个主机的各种信息，cpu 颗数、内存大小等。会存在 facts 中的某个变量中。调用后返回很多对应主机的信息，在后面的操作中可以根据不同的信息来做不同的操作。</span></p><pre data-language="shell" id="lMZRv" class="ne-codeblock language-shell" style="border: 1px solid #e8e8e8; border-radius: 2px; background: #f9f9f9; padding: 16px; font-size: 13px; color: #595959"><code>[root@server ~]# ansible web -m setup -a 'filter="*mem*"'	#查看内存
192.168.37.122 | SUCCESS =&gt; {
    "ansible_facts": {
        "ansible_memfree_mb": 1116, 
        "ansible_memory_mb": {
            "nocache": {
                "free": 1397, 
                "used": 587
            }, 
            "real": {
                "free": 1116, 
                "total": 1984, 
                "used": 868
            }, 
            "swap": {
                "cached": 0, 
                "free": 3813, 
                "total": 3813, 
                "used": 0
            }
        }, 
        "ansible_memtotal_mb": 1984
    }, 
    "changed": false
}
192.168.37.133 | SUCCESS =&gt; {
    "ansible_facts": {
        "ansible_memfree_mb": 1203, 
        "ansible_memory_mb": {
            "nocache": {
                "free": 1470, 
                "used": 353
            }, 
            "real": {
                "free": 1203, 
                "total": 1823, 
                "used": 620
            }, 
            "swap": {
                "cached": 0, 
                "free": 3813, 
                "total": 3813, 
                "used": 0
            }
        }, 
        "ansible_memtotal_mb": 1823
    }, 
    "changed": false
}</code></pre></details>

# Playbook

Playbook 与 ad-hoc 相比,是一种完全不同的运用 ansible 的方式，类似与 saltstack 的 state 状态文件。ad-hoc无法持久使用，playbook 可以持久使用。
playbook 是由一个或多个 play 组成的列表，play 的主要功能在于将事先归并为一组的主机装扮成事先通过ansible 中的 task 定义好的角色。从根本上来讲，所谓的 task 无非是调用 ansible 的一个 module。将多个 play组织在一个 playbook 中，即可以让它们联合起来按事先编排的机制完成某一任务。

## 核心元素

- Hosts 执行的远程主机列表
- Tasks 任务集
- Variables 内置变量或自定义变量在 playbook 中调用
- Templates 模板，即使用模板语法的文件，比如配置文件等
- Handlers 和 notify 结合使用，由特定条件触发的操作，满足条件方才执行，否则不执行
- tags 标签，指定某条任务执行，用于选择运行 playbook 中的部分代码。

## 语法

playbook 使用 yaml 语法格式，后缀可以是 yaml，也可以是 yml。

- 在单一一个 playbook 文件中，可以连续三个连子号(---)区分多个 play。还有选择性的连续三个点好(...)用来表示 play 的结尾，也可省略；
- 次行开始正常写 playbook 的内容，一般都会写上描述该 playbook 的功能；
- 使用#号注释代码；
- 缩进必须统一，不能空格和 tab 混用；
- 缩进的级别也必须是一致的，同样的缩进代表同样的级别，程序判别配置的级别是通过缩进结合换行实现的。
- YAML 文件内容和 Linux系统大小写判断方式保持一致，是区分大小写的，k/v 的值均需大小写敏感；
- k/v 的值可同行写也可以换行写。同行使用:分隔；
- v 可以是个字符串，也可以是一个列表；
- 一个完整的代码块功能需要最少元素包括 name: task。

```shell
# 创建playbook文件
[root@ansible ~]# cat playbook01.yml
---                       #固定格式
- hosts: 192.168.1.31     #定义需要执行主机
  remote_user: root       #远程用户
  vars:                   #定义变量
    http_port: 8088       #变量

  tasks:                             #定义一个任务的开始
    - name: create new file          #定义任务的名称
      file: name=/tmp/playtest.txt state=touch   #调用模块，具体要做的事情
    - name: create new user
      user: name=test02 system=yes shell=/sbin/nologin
    - name: install package
      yum: name=httpd
    - name: config httpd
      template: src=./httpd.conf dest=/etc/httpd/conf/httpd.conf
      notify:                 #定义执行一个动作(action)让handlers来引用执行，与handlers配合使用
        - restart apache      #notify要执行的动作，这里必须与handlers中的name定义内容一致
    - name: copy index.html
      copy: src=/var/www/html/index.html dest=/var/www/html/index.html
    - name: start httpd
      service: name=httpd state=started
  handlers:                                    #处理器：更加tasks中notify定义的action触发执行相应的处理动作
    - name: restart apache                     #要与notify定义的内容相同
      service: name=httpd state=restarted      #触发要执行的动作

#测试页面准备
[root@ansible ~]# echo "<h1>playbook test file</h1>" >>/var/www/html/index.html
#配置文件准备
[root@ansible ~]# cat httpd.conf |grep ^Listen
Listen {{ http_port }}

#执行playbook， 第一次执行可以加-C选项，检查写的playbook是否ok
[root@ansible ~]# ansible-playbook playbook01.yml
PLAY [192.168.1.31] *********************************************************************************************
TASK [Gathering Facts] ******************************************************************************************
ok: [192.168.1.31]
TASK [create new file] ******************************************************************************************
changed: [192.168.1.31]
TASK [create new user] ******************************************************************************************
changed: [192.168.1.31]
TASK [install package] ******************************************************************************************
changed: [192.168.1.31]
TASK [config httpd] *********************************************************************************************
changed: [192.168.1.31]
TASK [copy index.html] ******************************************************************************************
changed: [192.168.1.31]
TASK [start httpd] **********************************************************************************************
changed: [192.168.1.31]
PLAY RECAP ******************************************************************************************************
192.168.1.31               : ok=7    changed=6    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 


# 验证上面playbook执行的结果
[root@ansible ~]# ansible 192.168.1.31 -m shell -a 'ls /tmp/playtest.txt && id test02'
192.168.1.31 | CHANGED | rc=0 >>
/tmp/playtest.txt
uid=990(test02) gid=985(test02) 组=985(test02)

[root@ansible ~]# curl 192.168.1.31:8088
<h1>playbook test file</h1>
```

## 运行命令

<details class="lake-collapse"><summary id="u8ea28534"><span class="ne-text">ansible-playbook</span></summary><p id="u2d9634c7" class="ne-p" style="margin: 0; padding: 0; min-height: 24px"><span class="ne-text">ansible-playbook &lt;filename.yml&gt; ... [options]</span></p><pre data-language="shell" id="IH82w" class="ne-codeblock language-shell" style="border: 1px solid #e8e8e8; border-radius: 2px; background: #f9f9f9; padding: 16px; font-size: 13px; color: #595959"><code>[root@ansible PlayBook]# ansible-playbook -h
#ansible-playbook常用选项：
--check  or -C    #只检测可能会发生的改变，但不真正执行操作
--list-hosts      #列出运行任务的主机
--list-tags       #列出playbook文件中定义所有的tags
--list-tasks      #列出playbook文件中定义的所以任务集
--limit           #主机列表 只针对主机列表中的某个主机或者某个组执行
-f                #指定并发数，默认为5个
-t                #指定tags运行，运行某一个或者多个tags。（前提playbook中有定义tags）
-v                #显示过程  -vv  -vvv更详细</code></pre></details>

## 元素属性

### 主机与用户

在一个 playbook 开始时，最先定义的是要操作的主机和用户

```shell
---
- hosts: 192.168.1.31
  remote_user: root
```

除了上面的定义外，还可以在某一个 tasks 中定义要执行该任务的远程用户

```shell
tasks: 
  - name: run df -h
    remote_user: test
    shell: name=df -h
```

还可以定义使用 sudo 授权用户执行该任务

```shell
tasks: 
  - name: run df -h
    sudo_user: test
    sudo: yes
    shell: name=df -h
```

### tasks 任务列表

每一个 task 必须有一个名称 name，这样在运行 playbook 时，从其输出的任务执行信息中可以很清楚的辨别是属于哪一个 task 的，如果没有定义 name，action 的值将会用作输出信息中标记特定的 task。
每一个 playbook 中可以包含一个或者多个tasks任务列表，每一个 tasks 完成具体的一件事，（任务模块）比如创建一个用户或者安装一个软件等，在 hosts 中定义的主机或者主机组都将会执行这个被定义的 tasks。

```shell
tasks:
  - name: create new file
    file: path=/tmp/test01.txt state=touch
  - name: create new user
    user: name=test001 state=present
```

### Handlers 与 Notify

很多时候当我们某一个配置发生改变，我们需要重启服务，（比如 httpd 配置文件文件发生改变了）这时候就可以用到 handlers 和 notify 了；
(当发生改动时 )notify actions 会在 playbook 的每一个 task 结束时被触发，而且即使有多个不同 task 通知改动的发生，notify actions 知会被触发一次；比如多个 resources 指出因为一个配置文件被改动，所以 apache 需要重启，但是重新启动的操作知会被执行一次。

```shell
[root@ansible ~]# cat httpd.yml 
#用于安装httpd并配置启动
---
- hosts: 192.168.1.31
  remote_user: root

  tasks:
  - name: install httpd
    yum: name=httpd state=installed
  - name: config httpd
    template: src=/root/httpd.conf dest=/etc/httpd/conf/httpd.conf
    notify:
      - restart httpd
  - name: start httpd
    service: name=httpd state=started

  handlers:
    - name: restart httpd
      service: name=httpd state=restarted

#这里只要对httpd.conf配置文件作出了修改，修改后需要重启生效，在tasks中定义了restart httpd这个action，然后在handlers中引用上面tasks中定义的notify。
```

## Playbook 变量的使用

### 命令行指定变量

执行 playbook 时候通过参数 -e 传入变量，这样传入的变量在整个 playbook 中都可以被调用，属于全局变量。

```shell
[root@ansible PlayBook]# cat variables.yml 
---
- hosts: all
  remote_user: root

  tasks:
    - name: install pkg
      yum: name={{ pkg }}

#执行playbook 指定pkg
[root@ansible PlayBook]# ansible-playbook -e "pkg=httpd" variables.yml
```

### hosts 文件中定义变量

在 /etc/ansible/hosts 文件中定义变量，可以针对每个主机定义不同的变量，也可以定义一个组的变量，然后直接在 playbook 中直接调用。注意，组中定义的变量没有单个主机中的优先级高。

```shell
# 编辑hosts文件定义变量
[root@ansible PlayBook]# vim /etc/ansible/hosts
[apache]
192.168.1.36 webdir=/opt/test     #定义单个主机的变量
192.168.1.33
[apache:vars]      #定义整个组的统一变量
webdir=/web/test

[nginx]
192.168.1.3[1:2]
[nginx:vars]
webdir=/opt/web


# 编辑playbook文件
[root@ansible PlayBook]# cat variables.yml 
---
- hosts: all
  remote_user: root

  tasks:
    - name: create webdir
      file: name={{ webdir }} state=directory   #引用变量


# 执行playbook
[root@ansible PlayBook]# ansible-playbook variables.yml
```

### playbook 文件中定义变量

编写 playbook 时，直接在里面定义变量，然后直接引用，可以定义多个变量；注意：如果在执行 playbook 时，又通过 -e 参数指定变量的值，那么会以 -e 参数指定的为准。
 

```shell
# 编辑playbook
[root@ansible PlayBook]# cat variables.yml 
---
- hosts: all
  remote_user: root
  vars:                #定义变量
    pkg: nginx         #变量1
    dir: /tmp/test1    #变量2

  tasks:
    - name: install pkg
      yum: name={{ pkg }} state=installed    #引用变量
    - name: create new dir
      file: name={{ dir }} state=directory   #引用变量


# 执行playbook
[root@ansible PlayBook]# ansible-playbook variables.yml

# 如果执行时候又重新指定了变量的值，那么会已重新指定的为准
[root@ansible PlayBook]# ansible-playbook -e "dir=/tmp/test2" variables.yml
```

### 调用 setup 模块获取变量

setup 模块默认是获取主机信息的，有时候在 playbook 中需要用到，所以可以直接调用。

```shell
# 编辑playbook文件
[root@ansible PlayBook]# cat variables.yml 
---
- hosts: all
  remote_user: root

  tasks:
    - name: create file
      file: name={{ ansible_fqdn }}.log state=touch   #引用setup中的ansible_fqdn


# 执行playbook
[root@ansible PlayBook]# ansible-playbook variables.yml
```

### 独立的变量YAML文件中定义

为了方便管理将所有的变量统一放在一个独立的变量 YAML 文件中，playbook文件直接引用文件调用变量即可。
 

```shell
# 定义存放变量的文件
[root@ansible PlayBook]# cat var.yml 
var1: vsftpd
var2: httpd

# 编写playbook
[root@ansible PlayBook]# cat variables.yml 
---
- hosts: all
  remote_user: root
  vars_files:    #引用变量文件
    - ./var.yml   #指定变量文件的path（这里可以是绝对路径，也可以是相对路径）

  tasks:
    - name: install package
      yum: name={{ var1 }}   #引用变量
    - name: create file
      file: name=/tmp/{{ var2 }}.log state=touch   #引用变量


# 执行playbook
[root@ansible PlayBook]# ansible-playbook  variables.yml
```

## Playbook 中标签的使用

一个 playbook 文件中，执行时如果想执行某一个任务，那么可以给每个任务集进行打标签，这样在执行的时候可以通过 -t 选择指定标签执行，还可以通过 --skip-tags 选择除了某个标签外全部执行等。



```shell
# 编辑playbook
[root@ansible PlayBook]# cat httpd.yml 
---
- hosts: 192.168.1.31
  remote_user: root

  tasks:
    - name: install httpd
      yum: name=httpd state=installed
      tags: inhttpd

    - name: start httpd
      service: name=httpd state=started
      tags: sthttpd

    - name: restart httpd
      service: name=httpd state=restarted
      tags: 
        - rshttpd
        - rs_httpd

# 正常执行的结果
[root@ansible PlayBook]# ansible-playbook httpd.yml 

PLAY [192.168.1.31] **************************************************************************************************************************

TASK [Gathering Facts] ***********************************************************************************************************************
ok: [192.168.1.31]

TASK [install httpd] *************************************************************************************************************************
ok: [192.168.1.31]

TASK [start httpd] ***************************************************************************************************************************
ok: [192.168.1.31]

TASK [restart httpd] *************************************************************************************************************************
changed: [192.168.1.31]

PLAY RECAP ***********************************************************************************************************************************
192.168.1.31               : ok=4    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
# 通过-t指定tags名称，多个tags用逗号隔开
[root@ansible PlayBook]# ansible-playbook -t rshttpd httpd.yml 

PLAY [192.168.1.31] **************************************************************************************************************************

TASK [Gathering Facts] ***********************************************************************************************************************
ok: [192.168.1.31]

TASK [restart httpd] *************************************************************************************************************************
changed: [192.168.1.31]

PLAY RECAP ***********************************************************************************************************************************
192.168.1.31               : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

## Playbook 中模板的使用

template 模板为我们提供了动态配置服务，使用 jinja2 语言，里面支持多种条件判断、循环、逻辑运算、比较操作等。其实说白了也就是一个文件，和之前配置文件使用 copy 一样，只是使用 copy，不能根据服务器配置不一样进行不同动态的配置。这样就不利于管理。
说明：
1、多数情况下都将 template 文件放在和 playbook 文件同级的 templates 目录下（手动创建），这样playbook 文件中可以直接引用，会自动去找这个文件。如果放在别的地方，也可以通过绝对路径去指定；
2、模板文件后缀名为 .j2。

```shell
[root@ansible PlayBook]# cat testtmp.yml 
#模板示例
---
- hosts: all
  remote_user: root
  vars:
    - listen_port: 88    #定义变量

  tasks:
    - name: Install Httpd
      yum: name=httpd state=installed
    - name: Config Httpd
      template: src=httpd.conf.j2 dest=/etc/httpd/conf/httpd.conf    #使用模板
      notify: Restart Httpd
    - name: Start Httpd
      service: name=httpd state=started
      
  handlers:
    - name: Restart Httpd
      service: name=httpd state=restarted
[root@ansible PlayBook]# cat templates/httpd.conf.j2 |grep ^Listen
Listen {{ listen_port }}
# 目录结构
[root@ansible PlayBook]# tree .
.
├── templates
│   └── httpd.conf.j2
└── testtmp.yml

1 directory, 2 files
```

<details class="lake-collapse"><summary id="u32dd120d"><span class="ne-text" style="color: rgb(0, 0, 0)">when</span></summary><p id="u673b824b" class="ne-p" style="margin: 0; padding: 0; min-height: 24px"><span class="ne-text" style="color: rgb(0, 0, 0); font-size: 16px">条</span><span class="ne-text">件测试：如果需要根据变量、facts 或此前任务的执行结果来做为某 task 执行与否的前提时要用到条件测试，通过 when 语句执行，在 task 中使用 jinja2 的语法格式、<br></span><span class="ne-text">when 语句：<br></span><span class="ne-text">在 task 后添加 when 子句即可使用条件测试；when 语句支持 jinja2 表达式语法。</span></p><pre data-language="shell" id="caswm" class="ne-codeblock language-shell" style="border: 1px solid #e8e8e8; border-radius: 2px; background: #f9f9f9; padding: 16px; font-size: 13px; color: #595959"><code>[root@ansible PlayBook]# cat testtmp.yml 
#when示例
---
- hosts: all
  remote_user: root
  vars:
    - listen_port: 88

  tasks:
    - name: Install Httpd
      yum: name=httpd state=installed
    - name: Config System6 Httpd
      template: src=httpd6.conf.j2 dest=/etc/httpd/conf/httpd.conf
      when: ansible_distribution_major_version == "6"   #判断系统版本，为6便执行上面的template配置6的配置文件
      notify: Restart Httpd
    - name: Config System7 Httpd
      template: src=httpd7.conf.j2 dest=/etc/httpd/conf/httpd.conf
      when: ansible_distribution_major_version == "7"   #判断系统版本，为7便执行上面的template配置7的配置文件
      notify: Restart Httpd
    - name: Start Httpd
      service: name=httpd state=started

  handlers:
    - name: Restart Httpd
      service: name=httpd state=restarted</code></pre></details>

<details class="lake-collapse"><summary id="u64454930"><span class="ne-text" style="color: rgb(0, 0, 0)">with_items</span></summary><p id="u371973b4" class="ne-p" style="margin: 0; padding: 0; min-height: 24px"><span class="ne-text">with_items 迭代，当有需要重复性执行的任务时，可以使用迭代机制。<br></span><span class="ne-text">对迭代项的引用，固定变量名为“item”，要在 task 中使用 with_items 给定要迭代的元素列表。<br></span><span class="ne-text">列表格式：<br></span><span class="ne-text"> 字符串<br></span><span class="ne-text"> 字典</span></p><pre data-language="shell" id="HI65P" class="ne-codeblock language-shell" style="border: 1px solid #e8e8e8; border-radius: 2px; background: #f9f9f9; padding: 16px; font-size: 13px; color: #595959"><code>[root@ansible PlayBook]# cat testwith.yml 
# 示例with_items
---
- hosts: all
  remote_user: root

  tasks:
    - name: Install Package
      yum: name={{ item }} state=installed   #引用item获取值
      with_items:     #定义with_items
        - httpd
        - vsftpd
        - nginx

[root@ansible PlayBook]# cat testwith01.yml 
# 示例with_items嵌套子变量
---
- hosts: all
  remote_user: root

  tasks:
    - name: Create New Group
      group: name={{ item }} state=present
      with_items: 
        - group1
        - group2
        - group3 

    - name: Create New User
      user: name={{ item.name }} group={{ item.group }} state=present
      with_items:
        - { name: 'user1', group: 'group1' } 
        - { name: 'user2', group: 'group2' } 
        - { name: 'user3', group: 'group3' }</code></pre></details>

<details class="lake-collapse"><summary id="u1b1d44e2"><span class="ne-text" style="color: rgb(0, 0, 0)">for if</span></summary><pre data-language="shell" id="EVNuP" class="ne-codeblock language-shell" style="border: 1px solid #e8e8e8; border-radius: 2px; background: #f9f9f9; padding: 16px; font-size: 13px; color: #595959"><code>[root@ansible PlayBook]# cat testfor01.yml 
# template for 示例
---
- hosts: all
  remote_user: root
  vars:
    nginx_vhost_port:
      - 81
      - 82
      - 83

  tasks:
    - name: Templage Nginx Config
      template: src=nginx.conf.j2 dest=/tmp/nginx_test.conf

# 循环playbook文件中定义的变量，依次赋值给port
[root@ansible PlayBook]# cat templates/nginx.conf.j2 
{% for port in nginx_vhost_port %}
server{
     listen: {{ port }};
     server_name: localhost;
}
{% endfor %}</code></pre></details>

# Roles

## 介绍

ansible 自1.2版本引入的新特性，用于层次性、结构化地组织 playbook。roles 能够根据层次型结构自动装载变量文件、tasks 以及 handlers 等。要使用 roles 只需要在 playbook 中使用 include 指令引入即可。简单来讲，roles 就是通过分别将变量、文件、任务、模板及处理器放置于单独的目录中，并可以便捷的 include 它们的一种机制。角色一般用于基于主机构建服务的场景中，但也可以是用于构建守护进程等场景中。主要使用场景代码复用度较高的情况下。

## 目录结构

![img](https://cdn.nlark.com/yuque/0/2023/png/25552087/1679498922883-7c6137b3-9f99-4802-8ef3-8dc1c562593c.png)

<details class="lake-collapse"><summary id="u82225005"><span class="ne-text">各目录含义</span></summary><p id="u6d2be2dc" class="ne-p" style="margin: 0; padding: 0; min-height: 24px"><span class="ne-text">roles：所有的角色必须放在 roles 目录下，这个目录可以自定义位置，默认的位置在 </span><code class="ne-code" style="font-family: SFMono-Regular, Consolas, Liberation Mono, Menlo, Courier, monospace; background-color: rgba(0, 0, 0, 0.06); border: 1px solid rgba(0, 0, 0, 0.08); border-radius: 2px; padding: 0px 2px"><span class="ne-text">/etc/ansible/roles</span></code><span class="ne-text">；</span></p><p id="ue0ecd548" class="ne-p" style="margin: 0; padding: 0; min-height: 24px"><span class="ne-text">project：具体的角色项目名称，比如 nginx、tomcat、php；</span></p><p id="u4706c0b4" class="ne-p" style="margin: 0; padding: 0; min-height: 24px"><span class="ne-text">files：用来存放由 copy 模块或 script 模块调用的文件；</span></p><p id="ua05aad9a" class="ne-p" style="margin: 0; padding: 0; min-height: 24px"><span class="ne-text">templates：用来存放 jinjia2 模板，template 模块会自动在此目录中寻找 jinjia2 模板文件；</span></p><p id="u2fe5f944" class="ne-p" style="margin: 0; padding: 0; min-height: 24px"><span class="ne-text">tasks：此目录应当包含一个main.yml文件，用于定义此角色的任务列表，此文件可以使用include包含其它的位于此目录的task文件。</span></p><p id="u9b621060" class="ne-p" style="margin: 0; padding: 0; min-height: 24px"><span class="ne-text">      main.yml</span></p><p id="uae5767d7" class="ne-p" style="margin: 0; padding: 0; min-height: 24px"><span class="ne-text"> handlers：此目录应当包含一个 main.yml 文件，用于定义此角色中触发条件时执行的动作；</span></p><p id="ue920189a" class="ne-p" style="margin: 0; padding: 0; min-height: 24px"><span class="ne-text">      main.yml</span></p><p id="u19f1b9a9" class="ne-p" style="margin: 0; padding: 0; min-height: 24px"><span class="ne-text">  vars：此目录应当包含一个 main.yml 文件，用于定义此角色用到的变量；</span></p><p id="u82641d23" class="ne-p" style="margin: 0; padding: 0; min-height: 24px"><span class="ne-text">      main.yml</span></p><p id="u4de8c55c" class="ne-p" style="margin: 0; padding: 0; min-height: 24px"><span class="ne-text">  defaults：此目录应当包含一个 main.yml 文件，用于为当前角色设定默认变量；</span></p><p id="u46af6843" class="ne-p" style="margin: 0; padding: 0; min-height: 24px"><span class="ne-text">      main.yml</span></p><p id="u100fb422" class="ne-p" style="margin: 0; padding: 0; min-height: 24px"><span class="ne-text"> meta：此目录应当包含一个 main.yml 文件，用于定义此角色的特殊设定及其依赖关系；</span></p><p id="uef8f6070" class="ne-p" style="margin: 0; padding: 0; min-height: 24px"><span class="ne-text">      main.yml</span></p></details>

```shell
[root@ansible ~]# cd /etc/ansible/roles/
# 创建需要用到的目录
[root@ansible roles]# mkdir -p httpd/{handlers,tasks,templates,vars}
[root@ansible roles]# cd httpd/
[root@ansible httpd]# tree .
.
├── handlers
├── tasks
├── templates
└── vars

4 directories, 0 file

[root@ansible httpd]# vim vars/main.yml
PORT: 8088        #指定httpd监听的端口
USERNAME: www     #指定httpd运行用户
GROUPNAME: www    #指定httpd运行组

# copy一个本地的配置文件放在templates/下并已j2为后缀
[root@ansible httpd]# cp /etc/httpd/conf/httpd.conf templates/httpd.conf.j2

# 进行一些修改，调用上面定义的变量
[root@ansible httpd]# vim templates/httpd.conf.j2
Listen {{ PORT }} 
User {{ USERNAME }}
Group {{ GROUPNAME }}

# 创建组的task
[root@ansible httpd]# vim tasks/group.yml
- name: Create a Startup Group
  group: name=www gid=60 system=yes

# 创建用户的task
[root@ansible httpd]# vim tasks/user.yml
- name: Create Startup Users
  user: name=www uid=60 system=yes shell=/sbin/nologin

# 安装软件的task
[root@ansible httpd]# vim tasks/install.yml
- name: Install Package Httpd
  yum: name=httpd state=installed

# 配置软件的task
[root@ansible httpd]# vim tasks/config.yml
- name: Copy Httpd Template File
  template: src=httpd.conf.j2 dest=/etc/httpd/conf/httpd.conf
  notify: Restart Httpd

# 启动软件的task
[root@ansible httpd]# vim tasks/start.yml
- name: Start Httpd Service
  service: name=httpd state=started enabled=yes

# 编写main.yml，将上面的这些task引入进来
[root@ansible httpd]# vim tasks/main.yml
- include: group.yml
- include: user.yml
- include: install.yml
- include: config.yml
- include: start.ym

[root@ansible httpd]# vim handlers/main.yml
# 这里的名字需要和task中的notify保持一致
- name: Restart Httpd
  service: name=httpd state=restarted

[root@ansible httpd]# cd ..
[root@ansible roles]# vim httpd_roles.yml
---
- hosts: all
  remote_user: root
  roles:
    - role: httpd        #指定角色名称

[root@ansible roles]# tree .
.
├── httpd
│   ├── handlers
│   │   └── main.yml
│   ├── tasks
│   │   ├── config.yml
│   │   ├── group.yml
│   │   ├── install.yml
│   │   ├── main.yml
│   │   ├── start.yml
│   │   └── user.yml
│   ├── templates
│   │   └── httpd.conf.j2
│   └── vars
│       └── main.yml
└── httpd_roles.yml

5 directories, 10 files

[root@ansible roles]# ansible-playbook -C httpd_roles.yml 

PLAY [all] **************************************************************************************************

TASK [Gathering Facts] **************************************************************************************
ok: [192.168.1.33]
ok: [192.168.1.32]
ok: [192.168.1.31]
ok: [192.168.1.36]

TASK [httpd : Create a Startup Group] ***********************************************************************
changed: [192.168.1.31]
changed: [192.168.1.33]
changed: [192.168.1.36]
changed: [192.168.1.32]

TASK [httpd : Create Startup Users] *************************************************************************
changed: [192.168.1.33]
changed: [192.168.1.32]
changed: [192.168.1.31]
changed: [192.168.1.36]

TASK [httpd : Install Package Httpd] ************************************************************************
changed: [192.168.1.33]
changed: [192.168.1.32]
changed: [192.168.1.31]
changed: [192.168.1.36]

TASK [httpd : Copy Httpd Template File] *********************************************************************
changed: [192.168.1.33]
changed: [192.168.1.36]
changed: [192.168.1.32]
changed: [192.168.1.31]

TASK [httpd : Start Httpd Service] **************************************************************************
changed: [192.168.1.36]
changed: [192.168.1.31]
changed: [192.168.1.32]
changed: [192.168.1.33]

RUNNING HANDLER [httpd : Restart Httpd] *********************************************************************
changed: [192.168.1.36]
changed: [192.168.1.33]
changed: [192.168.1.32]
changed: [192.168.1.31]

PLAY RECAP **************************************************************************************************
192.168.1.31               : ok=7    changed=6    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
192.168.1.32               : ok=7    changed=6    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
192.168.1.33               : ok=7    changed=6    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
192.168.1.36               : ok=7    changed=6    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

1. 编写任务( task )的时候，里面不需要写需要执行的主机，单纯的写某个任务是干什么的即可，装软件的就是装软件的，启动的就是启动的。单独做某一件事即可，最后通过main.yml将这些单独的任务安装执行顺序 include进来即可，这样方便维护且一目了然。
2. 定义变量时候直接安装 k:v 格式将变量写在 vars/main.yml 文件即可，然后 task 或者 template 直接调用即可，会自动去 vars/main.yml 文件里面去找。
3. 定义 handlers 时候，直接在 handlers/main.yml 文件中写需要做什么事情即可，多可的话可以全部写在该文件里面，也可以像 task 那样分开来写，通过 include 引入一样的可以。在 task 调用 notify 时直接写与 handlers名字对应即可(二者必须高度一直)。
4. 模板文件一样放在 templates 目录下即可，task 调用的时后直接写文件名字即可，会自动去到 templates 里面找。注意：如果是一个角色调用另外一个角色的单个 task 时后，那么 task 中如果有些模板或者文件，就得写绝对路径了。

http://www.ansible.com.cn/index.html

https://www.cnblogs.com/yanjieli/p/10969299.html