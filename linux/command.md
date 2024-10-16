# 目录简介

| 目录        | 作用                                                         |
| ----------- | ------------------------------------------------------------ |
| /bin        | bin 是 binary 的缩写，这个目录存放着最经常使用的命令         |
| /boot       | 这里存放的是启动 Linux 时用到的引导程序文件                  |
| /dev        | device 的缩写，该目录下存放的是 Linux 的外部设备             |
| /etc        | 存放系统和第三方应用程序的配置文件                           |
| /home       | 存放普通用户家目录                                           |
| /lib /lib64 | 系统开机所需要最基本的动态连接共享库                         |
| /media      | 挂载 Linux 系统会自动识别的设备，例如 U 盘、光驱等           |
| /mnt        | 专门用于挂载操作的目录                                       |
| /opt        | 存放安装第三方应用程序时使用的压缩包文件                     |
| /proc       | 这个目录是一个虚拟的目录，它是系统内存的映射                 |
| /root       | 超级管理员 root 用户的家目录                                 |
| /run        | 存放进程产生的临时文件，关机重启后会消失                     |
| /sbin       | s 是 Super User 的意思，这里存放的是系统管理员使用的系统管理程序 |
| /srv        | service 缩写，该目录存放一些服务启动之后需要提取的数据       |
| /sys        | 该目录下安装了2.6内核中新出现的一个文件系统 sysfs            |
| /tmp        | 存放临时文件                                                 |
| /usr        | 应用程序的默认安装目录，类似于 Windows 下的 program files 目录 |
| /var        | 存放经常变化的内容，例如日志文件                             |

# 基本命令

## 文件目录相关

```shell
# 递归创建目录
mkdir -p aaa/bbb

# 查看目录
ls -l # 等于 ll

# 打印当前所在的目录
pwd

# 创建空文件
touch aaa.txt

# 复制文件/命令
cp 被复制的文件的路径 目标目录的路径
cp -r 被复制的目录的路径 目标目录的路径

# 将目录或文件在当前位置移动可以起到重命名的作用
mv 被移动的文件或目录的路径 目标目录

# 删除文件/目录
rm 被删除文件的路径
rm -f 被删除文件的路径 # 强制删除
rm -rf 被删除的目录的路径 # 强制递归删除

# 命令行环境下编辑文本文件
# vim 有三种模式：（ESC 退出当前模式）
# 一般模式：通过按键控制 vim 工作。
# 编辑模式：可以自由输入。  
# 指令模式：通过执行指令完成一些特殊的操作。
:set nu # 显示行号 
:wq # 保存退出 
:q! # 不保存退出
gg or 1 + shift + g # 前往第一行
n + shift + g # 前往第 n 行
dd # 删除光标所在的行
d5d # 从光标所在行开始，向下连续删除5行（包括光标所在行）
u # 撤销刚才的操作
Ctrl + r # 重做刚才撤销的操作
yy # 复制光标所在的行
p # 将当前复制的行粘贴到光标所在位置的下一行
y5y # 从光标所在行开始，向下连续复制5行（包括光标所在行）
r # 替换光标所在位置的一个字符。第一步：按一下 r 键 第二步：输入新的字符
i # 当前位置输入
I # 移动到行头输入
a # 向后移动一格输入
A # 移动到行尾输入
o # 在下面插入一行输入
/hello # 搜索 hello
:%s/hello/hi # 替换第一个 hello 为 hi
:%s/hello/hi/g # 替换全部

# 显示文件内容（适合较小文件）
cat 文件路径

# 分屏查看文件内容
less 文件路径

# 显示文件内容（可实时查看）
tail -n 5 文件的路径 # 显示文件末尾的5行
tail -F 文件的路径 # 实时查看文件末尾新增的内容

# 查找文件
find 查找范围 参数 表达式

# 基于索引查询文件或目录
locate 文件名
updatedb # 新建的文件或目录不会被系统将路径存入索引库中，需要更新索引库

# 将文件内容中匹配的行返回
grep 参数 查找内容 文件路径
-v 表示返回不匹配的行
-n 表示显示行号

# 管道符
命令 A | 命令 B # 将命令 A 的输出作为命令 B 的输入

# 解压
tar -zxvf 包的路径 -C 需要解压的目录路径
# -z 用 gzip 对存档压缩或解压
# -x 解压
# -v 详细显示处理的文件
# -f 指定存档或设备（缺省为 /dev/rmt0）
```

## 进程相关

```shell
# 查看系统中运行的进程
ps -ef
# 参数：
# -e 显示系统中全部的进程信息
# -f 表示完整格式
# 结果：
# 列名   含义
# UID	进程的用户信息
# PID	进程 id
# PPID	父进程的 id
# CMD	当前进程所对应的程序。
# C	    用整数表示的 CPU 使用率
# STIME	进程启动时间
# TTY	进程所在终端
# TIME	进程所占用的 CPU 时间

# 查看父子进程
pstree

# 杀进程
kill -9 pid

# 实时查看系统运行情况和健康状态
top 
# 参数：
# -d 更新间隔秒数	
# -i 不显示闲置或者僵死进程	
# -p 进程id
# 操作按键：
# P	默认值，根据 CPU 使用率排序
# M	以内存的使用率排序
# N	以 PID 排序
# d	设置数据刷新的时间间隔，单位是秒
# q	退出
# 结果：
# 第一行信息为任务队列信息：
# 14:12:01	                    系统当前时间
# up 6:04	                    系统的运行时间，前面例子表示本机已经运行6小时4分钟
# 2 users	                    当前登录了2个用户
# load average:0.00, 0.02, 0.05	系统在之前1分钟，5分钟，15分钟的平均负载。 一般认为小于1时，负载较小。
# 第二行为进程信息：
# Tasks: 210 total	系统中的进程总数
# 1 running	        正在运行的进程数
# 209 sleeping	    睡眠的进程
# 0 stopped	        正在停止的进程
# 0 zombie	        僵尸进程。如果不是0，需要手工检查僵尸进程
# 第三行为 CPU 信息：
# Cpu(s):3.0%us	用户空间占用的 CPU 百分比
# 3.0%sy	    内核空间占用的 CPU 百分比
# 0.0%ni	    改变过优先级的进程占用的 CPU 百分比
# 93.9%id	    空闲 CPU 的 CPU百分比
# 0.1%wa	    等待输入/输出的进程的占用 CPU 百分比
# 0.0%hi	    硬中断请求服务占用的CPU百分比
# 0.0%si	    软中断请求服务占用的CPU百分比
# 0.0%st	    虚拟时间百分比，当有虚拟机时，虚拟 CPU 等待实际 CPU 的时间百分比。
# 第四行为物理内存信息：
# 3861295 total	        物理内存的总量，单位KB
# 1037800 free	        空闲的物理内存数量
# 943564 used	        已经使用的物理内存数量
# 1879928 buff/cache	作为缓冲的内存数量
# 纳入内核管理的内存不见得都在使用中，还包括过去使用过的现在可以被重复利用的内存，内核并不把这些可被重新使用的内存还给 free ，因# 此 Linux 系统运行过程中 free 内存会越来越少，但不影响系统运行。因为这表示更多的空闲内存被内核管理了
# 第五行为交换分区（swap）信息
# 3145724 total	     交换分区（虚拟内存）的总大小
# 3145724 free	     空闲交换分区的大小
# 0 used	         已经使用的交互分区的大小
# 2649008 avail Mem  在不交换的情况下，对启动新应用程序可用内存的估计
# 交换分区是一个非常值得关注的地方，如果 swap 区的 used 数值持续发生变化那么说明在内核和交换分区之间正在持续发生数据交换，这表示内存不够用了——必须不断把内存中的数据保存到硬盘上

# 查看网络状态
netstat -anp
# -a 显示所有正在或不在侦听的套接字
# -n 显示数字形式地址而不是去解析主机、端口或用户名
# -p 显示套接字所属进程的 PID 和名称
```

## 辅助命令

```shell
# 查看命令历史
history

# 将数据输出到标准输出
echo $PATH

# 查看帮助文档
man 要查询的命令
info 要查询的命令

# 重启
sync	    # 将内存数据保存到硬盘上
poweroff	# 关机
reboot	    # 重启

# 给服务器发送请求
curl [-X] [GET|POST...] URL

# 客户端断开连接后，命令启动的进程仍然运行
nohup java -jar spring-boot-demo.jar > springboot.log 2 > &1 &

# 下载资源
wget [-P 指定目标目录] URL
```

## 字符串相关

正则表达式：

| 符号     | 含义                        |
| -------- | --------------------------- |
| ^        | 匹配字符串开始位置的字符    |
| $        | 匹配字符串结束位置的字符    |
| .        | 匹配任何一个字符            |
| *        | 匹配前面的字符出现 0 ~ n 次 |
| [a,m,u]  | 匹配字符 a 或 m 或 u        |
| [a-z]    | 匹配所有小写字母            |
| [A-Z]    | 匹配所有大写字母            |
| [a-zA-Z] | 匹配所有字母                |
| [0-9]    | 匹配所有数字                |
| \        | 特殊符号转义                |

```shell
# 返回路径字符串中的资源（文件或目录本身）部分
basename /aa/bb/cc/dd

# 返回路径字符串中的目录部分
dirname /aa/bb/cc/dd

# 根据指定符号拆分字符串并提取（默认根据 \t 拆分）
cut -d xxx -f n 字符串

# 文件或内容进行排序
sort [参数] [文件路径]
# n	依照数值大小排序
# r	相反顺序排序
# t	设置排序时使用的分隔字符
# k	指定需要排序的列

# 将管道符号提供的 stdin 数据转换为后面命令的命令行参数
| xargs
```

## 权限控制

```shell
# 返回用户信息
id root

# 创建用户组
groupadd 组名

# 创建用户
useradd -g 组名 用户名

# 用户设置密码
passwd 用户名

# 删除用户
userdel [-r] 用户名 # -r 表示将用户的家目录一起删除

# 文件权限
# 权限的符号表示	权限的二进制表示	权限的十进制表示
# rwx r-x r-x	 111 101 101	      7 5 5
# rw- r-- r--	 110 100 100	      6 4 4
# chmod	 修改权限信息
# chown	 修改文件或目录的所属主
# chgrp	 修改文件或目录的所属组

# 权限提升
sudo # /etc/sudoers 文件中添加配置

# 服务命令
systemctl start 服务名 # 启动服务
systemctl restart 服务名 # 重启服务
systemctl stop 服务名 # 停止服务
systemctl reload 服务名 # 重新加载服务
systemctl status 服务名 # 查看服务状态
systemctl list-unit-files # 查看服务列表
systemctl enable 服务名 # 设置具体服务开机自动启动
systemctl disable 服务名 # 取消具体服务开启自动启动

# 查看运行级别
cat /etc/inittab

# 防火墙
systemctl stop firewalld # 停止防火墙
systemctl disable firewalld # 取消防火墙开机自动启动
```

## 软件包管理

```shell
# rpm 可以管理 Linux 环境下的安装包
# qa	    查询系统中已经安装的程序，通常配合管道，使用 grep 精确匹配想要查询的包
# ivh	    执行 rpm 包安装操作
# e	        卸载 rpm 包
# nodeps	在卸载过程中忽略依赖关系 

# yum 基于 RPM 包管理，能够从指定的服务器自动下载 RPM 包并且安装，可以自动处理依赖性关系，并且一次安装所有依赖的软件包，无须繁琐地一次次下载、安装。
yum [选项] [参数]
# install	    安装 rpm 软件包
# update	    更新 rpm 软件包
# check-update	检查是否有可用的更新 rpm 软件包
# remove	    删除指定的 rpm 软件包
# list	        显示软件包信息
# clean	        清理 yum 过期的缓存
# deplist	    显示 yum 软件包的所有依赖关系
# 修改 yum 镜像源
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup # 备份
wget -O /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-7.repo # 下载最新源
yum clean all # 清理缓存
yum makecache # 生成缓存
```



# 参考资料

[1]: https://www.yuque.com/fairy-era/yg511q/oeybmv

