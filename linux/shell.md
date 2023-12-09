# 概述

## 简介

Shell 是一个命令行解释器，它接收应用程序或用户的命令，然后调用操作系统内核。Shell 还是一个功能强大的编程语言，易编写、易调试、灵活性强。

## Shell 解析器

```shell
# 查看Linux系统提供的Shell解析器
[root@test ~]# cat /etc/shells
/bin/sh
/bin/bash
/usr/bin/sh
/usr/bin/bash

# bash和sh的关系
[root@test ~]# ll /usr/bin | grep bash$
-rwxr-xr-x  1 root root     1150584 May 27  2021 bash
lrwxrwxrwx  1 root root           4 May 27  2021 sh -> bash

# Linux 默认的解析器
[root@test ~]# echo $SHELL
/bin/bash
```

## Shell 入门

步骤： 

1.  创建脚本文件：通常以 `.sh` 作为扩展名。 
2.  脚本内容： 

- - 第一行指定当前脚本的解析器：

```shell
#!/bin/bash
```

- - 实现具体功能：

```shell
echo "hello world"
```

1. Shell 脚本的运行方式： 

| 命令名             | 在当前进程运行 | 新建子进程运行 |
| ------------------ | -------------- | -------------- |
| source             | √              |                |
| .                  | √              |                |
| sh                 |                | √              |
| bash               |                | √              |
| chmod +x后直接运行 |                | √              |

其中，`.` 是 source 的另一种写法，在当前进程中发布的全局变量可以在当前进程的其他脚本中继续沿用，也可以在子进程中使用；但是，子进程 export 发布的变量仅限于子进程内部使用。

# 变量

变量表示命名的内存空间，将数据放在内存空间中，通过变量名引用，获取数据。

Shell 属于动态编译的弱类型语言，因此不用事先声明，可随时改变类型，并且语言运行时会隐式做数据类型转换，无须指定类型，默认均为字符型；参与运算会自动进行隐式类型转换；变量无须事先定义可直接调用。

## 变量命名法则

变量名和等号之间不能有空格，同时，变量名的命名须遵循如下规则：

- 命名只能使用英文字母，数字和下划线，首个字符不能以数字开头；
- 中间不能有空格，可以使用下划线 `_`；
- 不能使用标点符号；
- 不能使用 bash 里的关键字（可用 help 命令查看保留关键字）。

## 变量作用域

- 普通变量：生效范围为当前 shell 进程，对当前 shell 之外的其它 shell 进程，包括当前 shell 的子 shell 进程均无效；
- 环境变量：生效范围为当前 shell 进程及其子进程；
- 本地变量：生效范围为当前 shell 进程中某代码片断，通常指函数。

## 变量赋值

```shell
# 直接字串
name='root'

# 变量引用
name="$USER"

# 命令引用
name=$(COMMAND)
```

## 变量引用

弱引用和强引用：

- `"$name"`：弱引用，其中的变量引用会被替换为变量值；
- `'$name'`：强引用，其中的变量引用不会被替换为变量值，而保持原字符串。

```shell
echo $var

# 变量名外面的花括号是可选的，加不加都行，加花括号是为了帮助解释器识别变量的边界，
# 但以下情况必须添加：
for skill in Ada Coffe Action Java; do
    echo "I am good at ${skill}Script"
done
```

## 删除变量

```shell
unset <var>
```

## 环境变量

可以使子进程继承父进程的变量，但是无法让父进程使用子进程的变量。

```shell
# 声明并赋值环境变量
export name=VALUE
declare -x name=VALUE

# 查看环境变量
env
printenv
```

<details class="lake-collapse"><summary id="u5c5952e5"><span class="ne-text">bash 内建的环境变量</span></summary><p id="u9745407b" class="ne-p" style="margin: 0; padding: 0; min-height: 24px"><code class="ne-code" style="font-family: SFMono-Regular, Consolas, Liberation Mono, Menlo, Courier, monospace; background-color: rgba(0, 0, 0, 0.06); border: 1px solid rgba(0, 0, 0, 0.08); border-radius: 2px; padding: 0px 2px"><span class="ne-text">PATH</span></code></p><p id="u7e317775" class="ne-p" style="margin: 0; padding: 0; min-height: 24px"><code class="ne-code" style="font-family: SFMono-Regular, Consolas, Liberation Mono, Menlo, Courier, monospace; background-color: rgba(0, 0, 0, 0.06); border: 1px solid rgba(0, 0, 0, 0.08); border-radius: 2px; padding: 0px 2px"><span class="ne-text">SHELL</span></code></p><p id="uc5a74308" class="ne-p" style="margin: 0; padding: 0; min-height: 24px"><code class="ne-code" style="font-family: SFMono-Regular, Consolas, Liberation Mono, Menlo, Courier, monospace; background-color: rgba(0, 0, 0, 0.06); border: 1px solid rgba(0, 0, 0, 0.08); border-radius: 2px; padding: 0px 2px"><span class="ne-text">USER</span></code></p><p id="uc9af8734" class="ne-p" style="margin: 0; padding: 0; min-height: 24px"><code class="ne-code" style="font-family: SFMono-Regular, Consolas, Liberation Mono, Menlo, Courier, monospace; background-color: rgba(0, 0, 0, 0.06); border: 1px solid rgba(0, 0, 0, 0.08); border-radius: 2px; padding: 0px 2px"><span class="ne-text">UID</span></code></p><p id="u777a9fba" class="ne-p" style="margin: 0; padding: 0; min-height: 24px"><code class="ne-code" style="font-family: SFMono-Regular, Consolas, Liberation Mono, Menlo, Courier, monospace; background-color: rgba(0, 0, 0, 0.06); border: 1px solid rgba(0, 0, 0, 0.08); border-radius: 2px; padding: 0px 2px"><span class="ne-text">HOME</span></code></p><p id="u81c588f7" class="ne-p" style="margin: 0; padding: 0; min-height: 24px"><code class="ne-code" style="font-family: SFMono-Regular, Consolas, Liberation Mono, Menlo, Courier, monospace; background-color: rgba(0, 0, 0, 0.06); border: 1px solid rgba(0, 0, 0, 0.08); border-radius: 2px; padding: 0px 2px"><span class="ne-text">PWD</span></code></p><p id="u1d33581b" class="ne-p" style="margin: 0; padding: 0; min-height: 24px"><code class="ne-code" style="font-family: SFMono-Regular, Consolas, Liberation Mono, Menlo, Courier, monospace; background-color: rgba(0, 0, 0, 0.06); border: 1px solid rgba(0, 0, 0, 0.08); border-radius: 2px; padding: 0px 2px"><span class="ne-text">SHLVL</span></code><span class="ne-text"> # Shell 的嵌套层数，即深度</span></p><p id="uf7049916" class="ne-p" style="margin: 0; padding: 0; min-height: 24px"><code class="ne-code" style="font-family: SFMono-Regular, Consolas, Liberation Mono, Menlo, Courier, monospace; background-color: rgba(0, 0, 0, 0.06); border: 1px solid rgba(0, 0, 0, 0.08); border-radius: 2px; padding: 0px 2px"><span class="ne-text">LANG</span></code></p><p id="ubccf42e4" class="ne-p" style="margin: 0; padding: 0; min-height: 24px"><code class="ne-code" style="font-family: SFMono-Regular, Consolas, Liberation Mono, Menlo, Courier, monospace; background-color: rgba(0, 0, 0, 0.06); border: 1px solid rgba(0, 0, 0, 0.08); border-radius: 2px; padding: 0px 2px"><span class="ne-text">MAIL</span></code></p><p id="u256fc727" class="ne-p" style="margin: 0; padding: 0; min-height: 24px"><code class="ne-code" style="font-family: SFMono-Regular, Consolas, Liberation Mono, Menlo, Courier, monospace; background-color: rgba(0, 0, 0, 0.06); border: 1px solid rgba(0, 0, 0, 0.08); border-radius: 2px; padding: 0px 2px"><span class="ne-text">HOSTNAME</span></code></p><p id="u418b908a" class="ne-p" style="margin: 0; padding: 0; min-height: 24px"><code class="ne-code" style="font-family: SFMono-Regular, Consolas, Liberation Mono, Menlo, Courier, monospace; background-color: rgba(0, 0, 0, 0.06); border: 1px solid rgba(0, 0, 0, 0.08); border-radius: 2px; padding: 0px 2px"><span class="ne-text">HISTSIZE</span></code></p><p id="u4b5d0369" class="ne-p" style="margin: 0; padding: 0; min-height: 24px"><code class="ne-code" style="font-family: SFMono-Regular, Consolas, Liberation Mono, Menlo, Courier, monospace; background-color: rgba(0, 0, 0, 0.06); border: 1px solid rgba(0, 0, 0, 0.08); border-radius: 2px; padding: 0px 2px"><span class="ne-text">_</span></code><span class="ne-text">       # 下划线：表示前一命令的最后一个参数</span></p></details>

## 只读变量

```shell
# 声明只读变量
readonly name
declare -r name

# 查看只读变量
readonly [-p]
declare -r
```

## 位置变量

在 bash 中内置的变量，在脚本代码中调用通过命令行传递给脚本的参数：

- `$1, $2, ...`：对应第1个、第2个等参数；
- `$0`：命令本身，包括路径；
- `$*`：传递给脚本的所有参数，全部参数合为一个字符串；
- `$@`：传递给脚本的所有参数，每个参数为独立字符串；
- `$#`：传递给脚本的参数的个数。

`$@` 和 `$*` 只在被双引号包起来的时候才会有差异。当被双引号包围时，`$@` 与没有被双引号包围时没有变化，每个参数依然是独立的。但是 `$*` 被双引号包围时，会将所有参数看作一个整体。

## 退出状态码变量

进程执行后，将使用变量`$?`保存状态码的相关数字，`$?`的值为0时代表成功，其余则都代表失败。

用户可以在脚本中使用以下命令自定义退出状态码：`exit [n]`。

脚本中一旦遇到 exit 命令，脚本会立即终止，终止退出状态取决于 exit 命令后面的数字；如果未给脚本指定退出状态码，整个脚本的退出状态码取决于脚本中执行的最后一条命令的状态码。

## 高级

### 高级变量赋值

![img](https://cdn.nlark.com/yuque/0/2022/png/25552087/1669282711044-b6145569-7a0a-480f-8626-927789234e37.png)

### 有变量类型

变量一般是无类型的，但是 bash 提供了 declare 和 typeset 两个命令用于指定变量的类型，两个命令是等价的。

语法：`declare [选项] 变量名`

选项：

-r 	声明或显示只读变量

-i 	将变量定义为整型数

-a 	将变量定义为数组

-A	 将变量定义为关联数组

-f 	显示已定义的所有函数名及其内容

-F 	仅显示已定义的所有函数名

-x 	声明或显示环境变量和函数

-l 	声明变量为小写字母

-u 	声明变量为大写字母

### 变量间接引用

#### eval

eval 命令将会首先扫描命令行进行所有的置换，然后再执行该命令。该命令适用于那些一次扫描无法实现其功能的变量,该命令对变量进行两次扫描。

```shell
CMD=whoami
echo  $CMD
# 结果为：whoami

eval $CMD
# 结果为：root
```

#### 间接变量引用

如果第一个变量的值是第二个变量的名字，从第一个变量引用第二个变量的值就称为间接变量引用  variable1 的值是 variable2，而 variable2 又是变量名，variable2 的值为 value，间接变量引用是指通过 variable1 获得变量值 value 的行为：

```
variable1=variable2
variable2=value
```

bash 提供了两种格式实现间接变量引用：

```
eval tempvar=\$$variable1
tempvar=${!variable1}
```

#### 变量引用 reference

```shell
#!/bin/bash
ceo=mage
title=ceo
declare -n ref=$title
[ -R ref ] && echo  reference
echo $ref
ceo=wang
echo $ref
```

# 运算符

## 算术运算符

| 运算符 | 说明                                        |
| ------ | ------------------------------------------- |
| +      | 加法                                        |
| -      | 减法                                        |
| *      | 乘法                                        |
| /      | 除法                                        |
| %      | 取余                                        |
| =      | 赋值                                        |
| ==     | 相等，用于比较两个数字，相同则返回 true     |
| !=     | 不相等，用于比较两个数字，不相同则返回 true |

```shell
#!/bin/bash
a=10
b=20

val=`expr $a + $b`

let var=$a + $b

((var=$a + $b))

if [ $a == $b ]
then
   echo "a 等于 b"
fi
if [ $a != $b ]
then
   echo "a 不等于 b"
fi
```

条件表达式要放在方括号之间，并且要有空格，例如: `[$a==$b]` 是错误的，必须写成 `[ $a == $b ]`。

## 关系运算符

| 运算符 | 说明                                                |
| ------ | --------------------------------------------------- |
| -eq    | 检测两个数是否相等，相等返回 true                   |
| -ne    | 检测两个数是否不相等，不相等返回 true               |
| -gt    | 检测左边的数是否大于右边的，如果是，则返回 true     |
| -lt    | 检测左边的数是否小于右边的，如果是，则返回 true     |
| -ge    | 检测左边的数是否大于等于右边的，如果是，则返回 true |
| -le    | 检测左边的数是否小于等于右边的，如果是，则返回 true |

```shell
#!/bin/bash

a=10
b=20

if [ $a -eq $b ]
then
   echo "$a -eq $b : a 等于 b"
else
   echo "$a -eq $b: a 不等于 b"
fi
if [ $a -ne $b ]
then
   echo "$a -ne $b: a 不等于 b"
else
   echo "$a -ne $b : a 等于 b"
fi
```

## 布尔运算符

| 运算符 | 说明                                               |
| ------ | -------------------------------------------------- |
| !      | 非运算，表达式为 true，则返回 false，否则返回 true |
| -o     | 或运算，有一个表达式为 true，则返回 true           |
| -a     | 与运算，两个表达式都为 true 才返回 true            |

```shell
#!/bin/bash

a=10
b=20

if [ $a != $b ]
then
   echo "$a != $b : a 不等于 b"
else
   echo "$a == $b: a 等于 b"
fi
if [ $a -lt 100 -a $b -gt 15 ]
then
   echo "$a 小于 100 且 $b 大于 15 : 返回 true"
else
   echo "$a 小于 100 且 $b 大于 15 : 返回 false"
fi
```

## 逻辑运算符

| 运算符 | 说明       |
| ------ | ---------- |
| &&     | 逻辑的 AND |
| \|\|   | 逻辑的 OR  |

```shell
#!/bin/bash

a=10
b=20

if [[ $a -lt 100 && $b -gt 100 ]]
then
   echo "返回 true"
else
   echo "返回 false"
fi

if [[ $a -lt 100 || $b -gt 100 ]]
then
   echo "返回 true"
else
   echo "返回 false"
fi
```

## 字符串运算符

| 运算符 | 说明                                      |
| ------ | ----------------------------------------- |
| =      | 检测两个字符串是否相等，相等返回 true     |
| !=     | 检测两个字符串是否不相等，不相等返回 true |
| -z     | 检测字符串长度是否为0，为0返回 true       |
| -n     | 检测字符串长度是否不为0，不为0返回 true   |
| $      | 检测字符串是否不为空，不为空返回 true     |

```shell
#!/bin/bash

a="abc"
b="efg"

if [ $a = $b ]
then
   echo "$a = $b : a 等于 b"
else
   echo "$a = $b: a 不等于 b"
fi
if [ $a != $b ]
then
   echo "$a != $b : a 不等于 b"
else
   echo "$a != $b: a 等于 b"
fi
if [ -z $a ]
then
   echo "-z $a : 字符串长度为 0"
else
   echo "-z $a : 字符串长度不为 0"
fi
```

## 文件测试运算符

| 操作符  | 说明                                                         |
| ------- | ------------------------------------------------------------ |
| -b file | 检测文件是否是块设备文件，如果是，则返回 true                |
| -c file | 检测文件是否是字符设备文件，如果是，则返回 true              |
| -d file | 检测文件是否是目录，如果是，则返回 true                      |
| -f file | 检测文件是否是普通文件（既不是目录，也不是设备文件），如果是，则返回 true |
| -g file | 检测文件是否设置了 SGID 位，如果是，则返回 true              |
| -k file | 检测文件是否设置了粘着位(Sticky Bit)，如果是，则返回 true。  |
| -p file | 检测文件是否是有名管道，如果是，则返回 true                  |
| -u file | 检测文件是否设置了 SUID 位，如果是，则返回 true              |
| -r file | 检测文件是否可读，如果是，则返回 true                        |
| -w file | 检测文件是否可写，如果是，则返回 true                        |
| -x file | 检测文件是否可执行，如果是，则返回 true                      |
| -s file | 检测文件是否为空（文件大小是否大于0），不为空返回 true       |
| -e file | 检测文件（包括目录）是否存在，如果是，则返回 true            |

```shell
#!/bin/bash

file="/var/www/runoob/test.sh"

if [ -r $file ]
then
   echo "文件可读"
else
   echo "文件不可读"
fi
if [ -w $file ]
then
   echo "文件可写"
else
   echo "文件不可写"
fi
if [ -x $file ]
then
   echo "文件可执行"
else
   echo "文件不可执行"
fi
```

## 流程控制

## if else

```shell
if condition1
then
    command1
elif condition2 
then 
    command2
else
    commandN
fi
```

if else 的`[...]` 判断语句中大于使用 `-gt`，小于使用 `-lt`，如果使用 `((...))` 作为判断语句，大于和小于可以直接使用 `>` 和 `<`。

```shell
# 使用 [...]
a=10
b=20
if [ $a == $b ]
then
   echo "a 等于 b"
elif [ $a -gt $b ]
then
   echo "a 大于 b"
elif [ $a -lt $b ]
then
   echo "a 小于 b"
else
   echo "没有符合的条件"
fi

# 使用 ((...))
if (( $a == $b ))
then
   echo "a 等于 b"
elif (( $a > $b ))
then
   echo "a 大于 b"
elif (( $a < $b ))
then
   echo "a 小于 b"
else
   echo "没有符合的条件"
fi

# 使用 test
num1=$[2*3]
num2=$[1+5]
if test $[num1] -eq $[num2]
then
    echo '两个数字相等!'
else
    echo '两个数字不相等!'
fi
```

## for 循环

```shell
for var in item1 item2 ... itemN
do
    command1
    command2
    ...
    commandN
done
for loop in 1 2 3 4 5
do
    echo "The value is: $loop"
done
```

## while

```shell
while condition
do
    command
done
#!/bin/bash
int=1
while(( $int <= 5 ))
do
    echo $int
    let "int++"
done
```

## 无限循环

```shell
# 第一种
while :
do
    command
done

# 第二种
while true
do
    command
done

# 第三种
for (( ; ; ))
```

## until 循环

```shell
until condition
do
    command
done
```

## case ... esac

```shell
case 值 in
模式1)
    command1
    command2
    ...
    commandN
    ;;
模式2)
    command1
    command2
    ...
    commandN
    ;;
esac
echo '输入 1 到 4 之间的数字:'
echo '你输入的数字为:'
read aNum
case $aNum in
    1)  echo '你选择了 1'
    ;;
    2)  echo '你选择了 2'
    ;;
    3)  echo '你选择了 3'
    ;;
    4)  echo '你选择了 4'
    ;;
    *)  echo '你没有输入 1 到 4 之间的数字'
    ;;
esac
```

## 跳出循环

### break

break 命令允许跳出所有循环（终止执行后面的所有循环）。

### continue

continue 命令与 break 命令类似，只有一点差别，它不会跳出所有循环，仅仅跳出当前循环。

# 函数

## 声明函数

```shell
[ function ] func_name [()]
{

 ...

}
```

1. 可以带`function func_name()`定义，也可以直接`func_name()`定义，不带任何参数；
2. 参数返回，可以显示加：return 返回，如果不加，将以最后一条命令运行结果，作为返回值。return 后跟数值 n（0-255）。

## 函数调用

## 脚本中定义并调用函数

```shell
#!/bin/bash

funcWithReturn(){
    echo "这个函数会对输入的两个数字进行相加运算..."
    echo "输入第一个数字: "
    read aNum
    echo "输入第二个数字: "
    read anotherNum
    echo "两个数字分别为 $aNum 和 $anotherNum !"
    return $(($aNum+$anotherNum))
}
funcWithReturn
echo "输入的两个数字之和为 $? !"
```

所有函数在使用前必须定义，这意味着必须将函数放在脚本开始部分，直至 shell 解释器首次发现它时，才可以使用。调用函数仅使用其函数名即可。

### 使用函数文件

可以将经常使用的函数存入一个单独的函数文件，然后将函数文件载入 shell，再进行调用函数。可以使用`delcare -f` 或 set 命令查看所有定义的函数，其输出列表包括已经载入 shell 的所有函数。

实现函数文件的过程：

1. 创建函数文件，只存放函数的定义；
2. 在 shell 脚本或交互式 shell 中调用函数文件，格式如下：`. filename` 或 `source filename`。

## 函数返回值

函数的执行结果返回值：

- 使用 echo 等命令进行输出；
- 函数体中调用命令的输出结果。

函数的退出状态码：

- 默认取决于函数中执行的最后一条命令的退出状态码；
- 自定义退出状态码，return 从函数中返回，用最后状态命令决定返回值`return 0`（无错误返回），`return 1-255`（有错误返回）。

## 函数参数

| 参数处理 | 说明                                                        |
| -------- | ----------------------------------------------------------- |
| $#       | 传递到脚本或函数的参数个数                                  |
| $*       | 以一个单字符串显示所有向脚本传递的参数                      |
| $$       | 脚本运行的当前进程 ID 号                                    |
| $!       | 后台运行的最后一个进程的 ID 号                              |
| $@       | 与 $* 相同，但是使用时加引号，并在引号中返回每个参数        |
| $-       | 显示 shell 使用的当前选项，与 set 命令功能相同              |
| $?       | 显示最后命令的退出状态，0表示没有错误，其他任何值表明有错误 |

```shell
#!/bin/bash

funWithParam(){
    echo "第一个参数为 $1 !"
    echo "第二个参数为 $2 !"
    echo "第十个参数为 $10 !"
    echo "第十个参数为 ${10} !"
    echo "第十一个参数为 ${11} !"
    echo "参数总数有 $# 个!"
    echo "作为一个字符串输出所有参数 $* !"
}
funWithParam 1 2 3 4 5 6 7 8 9 34 73
```

# 数组

数组中可以存放多个值。bash 只支持一维数组（不支持多维数组），初始化时不需要定义数组大小。

## 声明数组

普通数组可以不事先声明,直接使用：`declare -a ARRAY_NAME`。

关联数组必须先声明,再使用：`declare -A ARRAY_NAME`。

## 数组赋值

- 一次只赋值一个元素：`ARRAY_NAME[INDEX]=VALUE`；
- 一次赋值全部元素：`ARRAY_NAME=("VAL1" "VAL2" "VAL3" ...)`；
- 只赋值特定元素：`ARRAY_NAME=([0]="VAL1" [3]="VAL2" ...)`。

```shell
#!/bin/bash

# 数组定义
my_array=(A B "C" D)

array_name[0]=value0
array_name[1]=value1
array_name[2]=value2

# 关联数组
# bash支持关联数组，可以使用任意的字符串、或者整数作为下标来访问数组元素
# 定义关联数组
declare -A site=(["google"]="www.google.com" ["runoob"]="www.runoob.com" ["taobao"]="www.taobao.com")

declare -A site
site["google"]="www.google.com"
site["runoob"]="www.runoob.com"
site["taobao"]="www.taobao.com"
```

## 引用数组

- 引用数组元素：`${ARRAY_NAME[INDEX]}`；
- 引用数组所有元素：`${ARRAY_NAME[*]}`或`${ARRAY_NAME[@]}`。

```shell
# 读取数组
echo "第一个元素为: ${my_array[0]}"
echo "第二个元素为: ${my_array[1]}"
echo "第三个元素为: ${my_array[2]}"
echo "第四个元素为: ${my_array[3]}"

# 访问关联数组元素
echo ${site["runoob"]}

# 获取数组中的所有元素
my_array[0]=A
my_array[1]=B
my_array[2]=C
my_array[3]=D

echo "数组的元素为: ${my_array[*]}"
echo "数组的元素为: ${my_array[@]}"

# 在数组前加一个感叹号!可以获取数组的所有键
declare -A site
site["google"]="www.google.com"
site["runoob"]="www.runoob.com"
site["taobao"]="www.taobao.com"

echo "数组的键为: ${!site[*]}"
echo "数组的键为: ${!site[@]}"

# 获取数组的长度
my_array[0]=A
my_array[1]=B
my_array[2]=C
my_array[3]=D

echo "数组元素个数为: ${#my_array[*]}"
echo "数组元素个数为: ${#my_array[@]}"
```

## 删除数组

删除数组中的某元素（会导致稀疏格式）：`unset ARRAY[INDEX]`；

删除整个数组：`unset ARRAY`。

## 数组切片

语法：`${ARRAY[@]:offset:number}`

说明：

offset	要跳过的元素个数 

number	要取出的元素个数 

取偏移量之后的所有元素：`{ARRAY[@]:offset}`。

# 字符串

```shell
# 返回字符串变量var的长度
${#var}

# 返回字符串变量var中从第offset个字符后（不包括第offset个字符）的字符开始，到最后的部分，offset的取值在0 到 ${#var}-1 之间(bash4.2后，允许为负值)
${var:offset}

# 返回字符串变量var中从第offset个字符后（不包括第offset个字符）的字符开始，长度为number的部分
${var:offset:number}

# 取字符串的最右侧几个字符,取字符串的最右侧几个字符, 注意：冒号后必须有一空白字符
${var:  -length}

# 从最左侧跳过offset字符，一直向右取到距离最右侧lengh个字符之前的内容,即:掐头去尾
${var:offset:-length}

# 先从最右侧向左取到length个字符开始，再向右取到距离最右侧offset个字符之间的内容,注意：-length前空格
${var:  -length:-offset}
# 其中word可以是指定的任意字符,自左而右，查找var变量所存储的字符串中，第一次出现的word, 删除字符串开头至第一次出现word字符串（含）之间的所有字符
${var#*word}

# 同上，贪婪模式，不同的是，删除的是字符串开头至最后一次由word指定的字符之间的所有内容
${var##*word}

# 其中word可以是指定的任意字符,功能：自右而左，查找var变量所存储的字符串中，第一次出现的word, 删除字符串最后一个字符向左至第一次出现word字符串（含）之间的所有字符
${var%word*}

# 同上，只不过删除字符串最右侧的字符向左至最后一次出现word字符之间的所有字符
${var%%word*}
# 查找var所表示的字符串中，第一次被pattern所匹配到的字符串，以substr替换之
${var/pattern/substr}

# 查找var所表示的字符串中，所有能被pattern所匹配到的字符串，以substr替换之
${var//pattern/substr}

# 查找var所表示的字符串中，行首被pattern所匹配到的字符串，以substr替换之
${var/#pattern/substr}

# 查找var所表示的字符串中，行尾被pattern所匹配到的字符串，以substr替换之
${var/%pattern/substr}
# 删除var表示的字符串中第一次被pattern匹配到的字符串
${var/pattern}

# 删除var表示的字符串中所有被pattern匹配到的字符串
${var//pattern}

# 删除var表示的字符串中所有以pattern为行首匹配到的字符串
${var/#pattern}

# 删除var所表示的字符串中所有以pattern为行尾所匹配到的字符串
${var/%pattern}
# 把var中的所有小写字母转换为大写
${var^^}

# 把var中的所有大写字母转换为小写
${var,,}
# 字符串可以用单引号，也可以用双引号，也可以不用引号。
str='this is a string'

# 单引号字符串的限制：
# ● 单引号里的任何字符都会原样输出，单引号字符串中的变量是无效的；
# ● 单引号字串中不能出现单独一个的单引号（对单引号使用转义符后也不行），但可成对出现，
#   作为字符串拼接使用
your_name="runoob"
str="Hello, I know you are \"$your_name\"! \n"
echo -e $str

# 双引号的优点：
# ● 双引号里可以有变量
# ● 双引号里可以出现转义字符

# 拼接字符串
your_name="runoob"

# 使用双引号拼接
greeting="hello, "$your_name" !"
greeting_1="hello, ${your_name} !"
echo $greeting  $greeting_1

# 使用单引号拼接
greeting_2='hello, '$your_name' !'
greeting_3='hello, ${your_name} !'
echo $greeting_2  $greeting_3

# 获取字符串长度
string="abcd"
echo ${#string}

# 变量为数组时，${#string} 等价于 ${#string[0]}
string="abcd"
echo ${#string[0]}

# 提取子字符串
string="runoob is a great site"
echo ${string:1:4}

# 查找子字符串
# 查找字符 i 或 o 的位置(哪个字母先出现就计算哪个)
string="runoob is a great site"
echo `expr index "$string" io`
```

# 输入输出重定向

| 命令            | 说明                                               |
| --------------- | -------------------------------------------------- |
| command > file  | 将输出重定向到 file。                              |
| command < file  | 将输入重定向到 file。                              |
| command >> file | 将输出以追加的方式重定向到 file。                  |
| n > file        | 将文件描述符为 n 的文件重定向到 file。             |
| n >> file       | 将文件描述符为 n 的文件以追加的方式重定向到 file。 |
| n >& m          | 将输出文件 m 和 n 合并。                           |
| n <& m          | 将输入文件 m 和 n 合并。                           |
| << tag          | 将开始标记 tag 和结束标记 tag 之间的内容作为输入。 |

## 输入重定向

```shell
command1 > file1
```

## 输出重定向

```shell
command1 < file1
```

## 重定向深入讲解

一般情况下，每个 Unix/Linux 命令运行时都会打开三个文件：

- 标准输入文件(stdin)：stdin 的文件描述符为0，Unix 程序默认从 stdin 读取数据。
- 标准输出文件(stdout)：stdout 的文件描述符为1，Unix 程序默认向 stdout 输出数据。
- 标准错误文件(stderr)：stderr 的文件描述符为2，Unix 程序会向 stderr 流中写入错误信息。

默认情况下，`command > file` 将 stdout 重定向到 file，`command < file` 将 stdin 重定向到 file。

- 如果希望 stderr 重定向到 file，可以这样写：`$ command 2>file`
- 如果希望 stderr 追加到 file 文件末尾，可以这样写：`$ command 2>>file`，2 表示标准错误文件(stderr)。
- 如果希望将 stdout 和 stderr 合并后重定向到 file，可以这样写：`$ command > file 2>&1 或者 $ command >> file 2>&1`
- 如果希望对 stdin 和 stdout 都重定向，可以这样写：`$ command < file1 >file2`

## Here Document

Here Document 是 shell 中的一种特殊的重定向方式，用来将输入重定向到一个交互式 shell 脚本或程序。

```shell
command << delimiter
    document
delimiter
```

结尾的 delimiter 一定要顶格写，前面不能有任何字符，后面也不能有任何字符，包括空格和 tab 缩进。开始的delimiter 前后的空格会被忽略掉。

## /dev/null 文件

如果希望执行某个命令，但又不希望在屏幕上显示输出结果，那么可以将输出重定向到 /dev/null：`$ command > /dev/null`。

`/dev/null` 是一个特殊的文件，写入到它的内容都会被丢弃；如果尝试从该文件读取内容，那么什么也读不到。但是 /dev/null 文件非常有用，将命令的输出重定向到它，会起到"禁止输出"的效果。

如果希望屏蔽 stdout 和 stderr，可以这样写：`$ command > /dev/null 2>&1`。

0 是标准输入（STDIN），1 是标准输出（STDOUT），2 是标准错误输出（STDERR）。这里的 2 和 > 之间不可以有空格，2> 是一体的时候才表示错误输出。

## 文件包含

和其他语言一样，shell 也可以包含外部脚本。这样可以很方便的封装一些公用的代码作为一个独立的文件。

```shell
. filename   # 注意点号(.)和文件名中间有一空格

或

source filename
```

被包含的文件 test1.sh 不需要可执行权限。

# 其他

## bash 配置文件

bash shell 的配置文件很多，可以分成以下类别：

### 按生效范围划分

1. 全局配置：`/etc/profile`、`/etc/profile.d/*.sh`和`/etc/bashrc`；
2. 个人配置：`~/.bash_profile`和`~/.bashrc`；

### 按登录方式划分

1. 交互式登录：

- 直接通过终端输入账号密码登录；
- 使用`su - 用户名`切换用户。

执行顺序：

```
/etc/profile.d/*.sh 
/etc/bashrc 
/etc/profile
/etc/bashrc
.bashrc 
.bash_profile
```

1. 非交互式登录：

- `su 用户名`；
- 图形界面打开终端；
- 执行脚本；
- 任何其他的 bash 实例。

执行顺序：

```
/etc/profile.d/*.sh 
/etc/bashrc 
.bashrc
```

### 按功能划分分类

1. profile 类：profile 类为交互式登录的 shell 提供配置，用于定义环境变量或者运行命令或脚本。

- 全局：`/etc/profile`和`/etc/profile.d/*.sh`；
- 个人：`~/.bash_profile`。

1. bashrc 类：为非交互式和交互式登录的 shell 提供配置，用于定义命令别名和函数或者定义本地变量。

- 全局：`/etc/bashrc`；
- 个人：`~/.bashrc`。

### 使配置文件生效

修改 profile 和 bashrc 文件后需生效两种方法:

1. 重新启动 shell 进程；
2. `source|.  配置文件`。

## trap

`trap`命令设置信号处理行为。

语法：`trap [-lp] [[参数] 信号声明 …]`

其中，参数可以是 shell 命令或者自定义函数，信号声明为定义在`<signal.h>`中的信号名或者数值。信号名的大小写不敏感，SIG 这个前缀也是可选的。

```shell
#!/bin/bash
# EXIT：在shell退出前执行trap设置的命令，也可以指定为0
# RETURN：在函数返回时，或者.和source执行其他脚本返回时，执行trap设置的命令
# DEBUG：在任何命令执行前执行trap设置的命令，但对于函数仅在函数的第一条命令前执行一次
# ERR：在命令结果为非0时，执行trap设置的命令
foo()
{
echo "foo 1" 
echo "foo 2"
}
trap "echo 123" DEBUG
foo
```

`trap`捕捉到信号之后，可以有三种反应方式：

1. 执行一段程序来处理这一信号；
2. 接受信号的默认操作；
3. 忽视这一信号。

`trap`对上面三种方式提供了三种基本形式：

1. `trap`命令在 shell 接收到`<信号列表>`清单中数值相同的信号时，将执行双引号中的命令串：

```
trap '<命令>' <信号列表>
trap "<命令>" <信号列表>
```

1. 为了恢复信号的默认操作，使用第二种形式的`trap`命令：

```
trap <信号列表>
```

1. 第三种形式的`trap`命令允许忽视信号：

```
trap " " <信号列表>
```

- `trap`可以在收到信号前的任意位置设置，并非需要在脚本的第一行，但是 shell 是按照顺序执行语句的，不会优先执行`trap`；
- 在函数中设置`trap`，也是全局生效的；
- 对于同一个信号，只有最后一次`trap`生效；
- trap 只在本进程内有效，它的子进程不会继承`trap`的设置。
- 在捕捉到`<信号列表>`中指定的信号并执行完相应的命令之后，如果这些命令没有将 shell 程序终止的话，shell 程序将继续执行收到信号时所执行的命令后面的命令，这样将很容易导致 shell 程序无法终止。
- 在`trap`语句中，单引号和双引号是不同的，当 shell 程序第一次碰到`trap`语句时，将把`<命令>`中的命令扫描一遍。此时若`<命令>`是用单引号括起来话，那么 shell 不会对`<命令>`中的变量和命令进行替换，否则`<命令>`中的变量和命令将用当时具体的值来替换。

## set

set 指令可根据不同的需求来设置当前所使用 shell 的执行方式，同时也可以用来设置或显示 shell 变量的值。

最常用的两个参数就是 -e 与 -x ，一般写在 shell 代码逻辑之前，这两个组合在一起用，可以在 debug 的时候替你节省许多时间 。

- `set -x`：会在执行每一行 shell 脚本时，把执行的内容输出来。它可以让你看到当前执行的情况，里面涉及的变量也会被替换成实际的值。
- `set -e`：会在执行出错时结束程序，就像其他语言中的“抛出异常”一样。

## expect

主要应用于自动化交互式操作的场景，借助 expect 处理交互的命令，可以将交互过程如：ssh 登录，ftp 登录等写在一个脚本上，使之自动化完成。

语法：`expect [选项] [ -c cmds ] [ [ -[f|b] ] cmdfile ] [ args ]`

选项：

-c	从命令行执行`expect`脚本，默认`expect`是交互地执行的

-d	可以输出输出调试信息

```shell
#!/bin/bash
ip=$1 
user=$2
password=$3
expect <<EOF
set timeout 20
spawn ssh $user@$ip
expect {
        "yes/no" { send "yes\n";exp_continue }
        "password" { send "$password\n" }
}
expect "]#" { send "useradd hehe\n" }
expect "]#" { send "echo magedu |passwd --stdin hehe\n" }
expect "]#" { send "exit\n" }
expect eof
EOF 
```

## awk

awk 适合格式化文本，对文本进行较复杂格式处理

```shell
awk '{pattern + action}' {filenames}
```

### 基本用法

```shell
# 用法一
awk '{[pattern] action}' {filenames}   # 行匹配语句 awk '' 只能用单引号
# 实例一
awk '{print $1,$4}' log.txt

# 用法二
awk -F  #-F相当于内置变量FS, 指定分割字符
# 实例二
awk -F ',' '{print $1,$2}'   log.txt
awk 'BEGIN{FS=","} {print $1,$2}'   log.txt   # 使用内建变量
# 使用多个分隔符，先使用空格分割，然后对分割结果再使用","分割
awk -F '[ ,]'  '{print $1,$2,$5}'   log.txt

# 用法三
awk -v  # 设置变量
# 实例三
awk -va=1 -vb=s '{print $1,$1+a,$1b}' log.txt

# 用法四
awk -f {awk脚本} {文件名}
# 实例四
awk -f cal.awk log.txt
```

### 运算符

| 运算符                  | 描述                             |
| ----------------------- | -------------------------------- |
| = += -= *= /= %= ^= **= | 赋值                             |
| ?:                      | C条件表达式                      |
| \|\|                    | 逻辑或                           |
| &&                      | 逻辑与                           |
| ~ 和 !~                 | 匹配正则表达式和不匹配正则表达式 |
| < <= > >= != ==         | 关系运算符                       |
| 空格                    | 连接                             |
| + -                     | 加，减                           |
| * / %                   | 乘，除与求余                     |
| + - !                   | 一元加，减和逻辑非               |
| ^ ***                   | 求幂                             |
| ++ --                   | 增加或减少，作为前缀或后缀       |
| $                       | 字段引用                         |
| in                      | 数组成员                         |

```shell
# 过滤第一列大于2的行
awk '$1>2' log.txt

# 过滤第一列等于2的行
awk '$1==2 {print $1,$3}' log.txt

# 过滤第一列大于2并且第二列等于'Are'的行
awk '$1>2 && $2=="Are" {print $1,$2,$3}' log.txt
```

### 内建变量

| 变量        | 描述                                              |
| ----------- | ------------------------------------------------- |
| $n          | 当前记录的第n个字段，字段间由FS分隔               |
| $0          | 完整的输入记录                                    |
| ARGC        | 命令行参数的数目                                  |
| ARGIND      | 命令行中当前文件的位置(从0开始算)                 |
| ARGV        | 包含命令行参数的数组                              |
| CONVFMT     | 数字转换格式(默认值为%.6g)ENVIRON环境变量关联数组 |
| ERRNO       | 最后一个系统错误的描述                            |
| FIELDWIDTHS | 字段宽度列表(用空格键分隔)                        |
| FILENAME    | 当前文件名                                        |
| FNR         | 各文件分别计数的行号                              |
| FS          | 字段分隔符(默认是任何空格)                        |
| IGNORECASE  | 如果为真，则进行忽略大小写的匹配                  |
| NF          | 一条记录的字段的数目                              |
| NR          | 已经读出的记录数，就是行号，从1开始               |
| OFMT        | 数字的输出格式(默认值是%.6g)                      |
| OFS         | 输出字段分隔符，默认值与输入字段分隔符一致。      |
| ORS         | 输出记录分隔符(默认值是一个换行符)                |
| RLENGTH     | 由match函数所匹配的字符串的长度                   |
| RS          | 记录分隔符(默认是一个换行符)                      |
| RSTART      | 由match函数所匹配的字符串的第一个位置             |
| SUBSEP      | 数组下标分隔符(默认值是/034)                      |

### 正则匹配

```shell
# 输出第二列包含 "th"，并打印第二列与第四列
awk '$2 ~ /th/ {print $2,$4}' log.txt
# 输出包含 "re" 的行
awk '/re/ ' log.txt
```

### 脚本

关于 awk 脚本，我们需要注意两个关键词 BEGIN 和 END。

- BEGIN{ 这里面放的是执行前的语句 }
- END {这里面放的是处理完所有的行后要执行的语句 }
- {这里面放的是处理每一行时要执行的语句}

## sed

sed 可依照脚本的指令来处理、编辑文本文件，主要用来自动编辑一个或多个文件、简化对文件的反复操作、编写转换程序等。

```shell
sed [-hnV][-e<script>][-f<script文件>][文本文件]
```

参数说明：

- -e<script>或--expression=<script> 以选项中指定的script来处理输入的文本文件；
- -f<script文件>或--file=<script文件> 以选项中指定的script文件来处理输入的文本文件；
- -h或--help 显示帮助；
- -n或--quiet或--silent 仅显示script处理后的结果；
- -V或--version 显示版本信息。

动作说明：

- a ：新增， a 的后面可以接字串，而这些字串会在新的一行出现(目前的下一行)；
- c ：取代， c 的后面可以接字串，这些字串可以取代 n1,n2 之间的行；
- d ：删除，因为是删除啊，所以 d 后面通常不接任何东东；
- i ：插入， i 的后面可以接字串，而这些字串会在新的一行出现(目前的上一行)；
- p ：打印，亦即将某个选择的数据印出。通常 p 会与参数 sed -n 一起运行；
- s ：取代，可以直接进行取代的工作哩！通常这个 s 的动作可以搭配正则表达式。

### 基本用法

```shell
# 在 testfile 文件的第四行后添加一行，并将结果输出到标准输出
sed -e 4a\newLine testfile

# 以行为单位的新增/删除
# 将 testfile 的内容列出并且列印行号，同时将第 2~5 行删除
nl testfile | sed '2,5d'
# 删除第3到最后一行
nl testfile | sed '3,$d'
# 第二行后(即加在第三行)加上 drink tea
nl testfile | sed '2a drink tea'
# 增加两行以上，在第二行后面加入两行字
nl testfile | sed '2a Drink tea or ......\
drink beer ?'

# 以行为单位的替换与显示
# 将第 2-5 行的内容取代成为 No 2-5 number
nl testfile | sed '2,5c No 2-5 number'

# 数据的搜寻并显示
# 搜索 testfile 有 oo 关键字的行
nl testfile | sed -n '/oo/p'

# 数据的搜寻并删除
# 删除 testfile 所有包含 oo 的行，其他行输出
nl testfile | sed  '/oo/d'

# 数据的搜寻并执行命令
# 搜索 testfile，找到 oo 对应的行，执行后面花括号中的一组命令
nl testfile | sed -n '/oo/{s/oo/kk/;p;q}'

# 数据的查找与替换
sed 's/要被取代的字串/新的字串/g'
# 将 testfile 文件中每行第一次出现的 oo 用字符串 kk 替换，然后将该文件内容输出到标准输出
sed -e 's/oo/kk/' testfile
# g 标识符表示全局查找替换，使 sed 对文件中所有符合的字符串都被替换，
# 修改后内容会到标准输出，不会修改原文件
sed -e 's/oo/kk/g' testfile
# 选项i使sed修改文件
sed -i 's/oo/kk/g' testfile

# 多点编辑
# 删除 testfile 第三行到末尾的数据，并把 HELLO 替换为 RUNOOB
nl testfile | sed -e '3,$d' -e 's/HELLO/RUNOOB/'

# 直接修改文件内容
# 将regular_express.txt内每一行结尾若为 . 则换成 !
sed -i 's/\.$/\!/g' regular_express.txt
```

# 参考资料

[1]: http://www.yunweipai.com/34318.html
[2]: https://www.yuque.com/fairy-era/yg511q/xceoof
[3]: https://blog.csdn.net/m0_57515995/article/details/125713566
[4]: https://www.cnblogs.com/chinas/p/6957701.html
[5]: https://zhuanlan.zhihu.com/p/339054596







