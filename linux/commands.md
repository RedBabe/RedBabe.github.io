## 指令和管道

### grep

> Globally search a Regular Expression and Print

搜索正则并打印。

```bash
# 在文件中查找"path"字符串
$ grep path /etc/profile

# 查找有空格的字符串时，需要加引号
$ grep "e /usr" /etc/profile

# 忽略大小写
$ grep -i path /etc/profile

# 排除二进制文件(大写的i)
$ grep -I path /etc/profile

# 显示行号
$ grep -n path /etc/profile

# 显示文本不在的行
$ grep -v path /etc/profile # 不包含"path"的行都会显示

# 递归，所有子文件和子目录的文件都会显示
$ grep -r path /etc

# 使用正则 regular Expression
$ grep -E P.* /etc/profile
```
#### rgrep

> 等于grep -r

#### egrep

> 等于grep -e

#### 正则

> 正则是正则，和shell中的模糊查询是两码事

| 字符 | 含义                         |
| ---- | ---------------------------- |
| .    | 匹配除"\n"之外的任意单个字符 |
| ^    | 匹配字符串起始位置           |
| $    | 匹配字符串结尾位置           |
| []   | 匹配括号中的任意字符         |
| ?    | 匹配前面的字符0次或1次       |
| *    | 匹配前面的字符任意次         |
| +    | 匹配前面的字符1次或多次      |
| \|   | 逻辑或                       |
| ()   | 分组                         |

### sort

> 排序。按行排，就是同一行的它不管

```bash
# 顺序排列
$ sort file.txt

# 倒序排序
$ sort -r file.txt

# 随机排序。瞎排，随机的
$ sort -R file.txt

# 输出排序结果到文件
$ sort -o result.txt file.txt

# 对数字排序。默认情况下是按字符串排序
$ sort -n nums.txt
```

### wc

> word count。字符统计

``` bash
$ more file.txt # aa cc bb dd ee

# 统计结果：行数、单词数、字符数。注意每一行有一个\n字节
$ wc file.txt # 显示：1 5 15 file.txt

# 只统计行数
$ wc -l file.txt # 1 file.txt

# 只统计单词数
$ wc -w file.txt

# 只统计字符数
$ wc -c file.txt
$ wc -m file.txt # 不知道-m和-c的区别
```

### uniq

> unique。把连续的重复行变为一行

```bash
# 去重
$ uniq file.txt

# 去重并写入到新的文件
$ uniq file.txt result.txt

# 统计重复的行数
$ uniq -c file.txt # count

# 只显示重复的行数
$ uniq -d file.txt # duplicated
```

### cut

> 对每一行进行剪切

-c: count。从第几列到第几列
-d: delimiter。按照某个字符分隔
-f: field。输出的区域，如`-f 1`输出第1部分，`-f 1,3`输出第1和第3部分，`-f 2-`输出第2部分面部分。

`CSV`: Comma Separated Values，逗号分隔值，通常可以被Excel打开。

假设有个文件file.csv：

```txt
Mia,Famale,20
Jack,Male,30
```

``` bash
# 按照位置剪切
$ cut -c 2-4 file.csv # 每行：从第2个字符开始剪切，到第4个字符。-c: count
# 输出 
## ia,
## ack

# 按照分隔符剪切：
$ cut -d , -f 1,3 file.csv # -d: del
# 输出
## Mia,20
## Jack,30
```

### 重定向、管道和流

> pipeline

把两个命令连接起来使用，一个命令的输出作为另一个命令的输入。

`stdin`: 从键盘向终端输入数据，是标准输入。其文件描述符为0

`stdout`: 终端接收键盘的输入，会产生`标准输出`或标准错误输出。其文件描述符为1

`stderr`: `标准错误输出`。其文件描述符为2

#### 将标准输出重定向到文件中

`>`会直接新建/覆盖文件。不能重定向标准错误输出

```bash
# 之前会打印结果，现在会新建一个result.csv
$ cut -d , -f 1,3 file.csv > result.csv
```

`>>`会在文件末尾追加内容

``` bash
$ cut -d , -f 1,3 file.csv >> result.csv
```

#### 将标准错误输出重定向

`2>`会直接新建或覆盖文件

``` bash
$ cat not_exist_file > result.txt 2> error.log
```

`2>>`会在文件末尾追加内容

#### 将标准错误输出重定向到标准输出的文件

`2>&1`: 该符号不控制是覆盖还是追加

``` bash
# 覆盖
$ cat not_exist_file > result.txt 2>&1

# 追加
$ cat not_exist_file >> result.txt 2>&1
```

#### 标准输入重定向

`<`: 将输入重定向到指定文件

``` bash
# 先打开文件，再打印文件内容
$ cat file.csv

# cat接收文件内容并打印；打开文件的工作是终端完成的
$ cat < file.csv
```

`<<`: 将输入重定向到键盘。

``` bash
$ sort -n << END
> 12
> 9
> 27
> END
# 输出 9 12 27

$ wc -c << END
> Hi Mia. I am John.
> END
# 输出19
```

#### 输入和输出重定向

``` bash
$ sort -n << END >> result.txt 2>&1
```

#### 黑洞文件

/dev/null是一个黑洞文件。它不是目录，是一个特殊文件。所有输入进去的内容都会作废。

#### 管道

`|`: 管道符。

``` bash
# 
$ du /etc | sort -nr | head # head只显示前10个
```

### w

> 打印当前登录用户信息

``` bash
$ w
```

`load average`: 平均负载。过去的1分钟、5分钟、5分钟

打印：
 15:53:16 up 8 days, 16:24,  1 user,  load average: 0.00, 0.01, 0.05
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
root     pts/0    114.250.143.222  15:53    3.00s  0.02s  0.00s w

`USER`: 用户
`TTY`: 终端
`FROM`: 本地地址
`LOGIN@`: 登录时间
`IDLE`: 有多长时间未活跃
`JCPU`: 
`PCPU`: 
`WHAT`: 运行的内容

### uptime

> 系统运行了多长时间

它的输出结果就是`w`的第一行。

``` bash
$ uptime
```

### tload

> 实时显示load average

### who

> 查看用户。它输出的结果是`w`的一部分

### ps

> process status，列出运行的进程。

进程：就是加载到内存中运行的程序

``` bash
$ ps
```
显示：
  PID TTY          TIME CMD
23540 pts/0    00:00:00 bash
31374 pts/0    00:00:00 ps

`PID`: process identifier，进程号
`TTY`: 进程运行所在的终端
`TIME`: 进程运行的时间
`CMD`: 产生这个进程的程序名。同样的程序可以产生多个进程

`PPID`: Parent PID，父进程id

``` bash
# 列出所有进程
$ ps -ef | less

# 列出指定用户的进程
$ ps -u goyave | less

# 列出所有进程，但是更详细
$ ps -aux | less

# 根据CPU使用率来降序排列
$ ps -aux --sort -pcpu | less

# 根据内存使用率来降序排列
$ ps -aux --sort -pmem | less

# 综合CPU使用率和内存使用率来降序排列
$ ps -aux --sort -pcpu,+pmem | less

# 树形结构显示进程
$ ps -axjf # 等于pstree
```

#### pstree

> 等于ps -axjf

### top

> 进程的动态列表。按照CPU的使用率排列

``` bash
$ top
```

top界面的操作：

`q`: 退出
`h`: 显示帮助文档
`B`: 加粗某些信息
`f`/`F`: 添加/删除某些列
`u`: 输入用户，只显示指定用户的进程
`k`: 输入pid，结束指定进程
`s`: 修改刷新时间。默认是3s刷新一次
`Esc`: 取消当前操作

### glance 和 htop

这是两个非系统软件。很好用。百度下载。

### kill

> 杀死进程，后跟PID。

``` bash
$ kill 1088 # 1088是PID

# 强制结束，非常粗暴
$ kill -9 1088
```

#### killall

> 结束进程。后跟程序名

``` bash
$ killall find # 结束find程序
```

#### halt

> 停止。就是停止、关机

Halt the system

#### reboot

> 重启

Reboot the system.

#### shutdown

> Shut down the system

#### poweroff

> Power off the system。不需要root

### 后台进程

比如我们要执行find，它可能耗时比较长。这时候可以使用后台进程。

运行后台进程的方法

#### 1. 在命令的最后加上`&`

``` bash
# 会返回PID
$ touch file.txt & # 输出 [1] 89

# 使用需要sudo的指令时，需要先sudo，再执行指令
$ find / -name "*log" > result.txt &
```

#### 2. nohup

使用`&`时，若终端关闭，该进程也会随之关闭。

``` bash
# 执行后台进程
$ nohup cp file.txt copy.txt
# 查询进程状况
$ ps -ax | grep "cp file.txt copy.txt"
```

#### 3. ctrl + Z 配合 bg

``` bash
$ find / -name log # 搜索

# 打断：ctrl + Z

$ bg # 重启stopped进程
$ bg % 1 # 重启[1]这个stopped进程
```

#### fg

> foreground。

操作类似于bg，但是它会让进程在前台重新运行

#### jobs

查看shell当前正在处理的进程。

#### 5种常见的进程状态

1. 运行：正在运行或在运行队列种等待。状态码是R（Runnable）
2. 中断：休眠中，受阻。状态码是S（Sleeping）
3. 不可中断：进程不响应系统异步信号。状态码是D（uninterruptible sleep）
4. 僵死：进程已终止，但进程描述符依然存在。状态码是Z
5. 停止：进程收到停止信号后停止执行。状态码是T（Traced or stopped）

### 任务延时执行

#### date

``` bash
# 打印小时
$ date "+%H"

# 打印时分秒
$ date "+%H:%M:%S"

# 修改系统时间
$ date 10121350 # 10月12日13点50分
```

#### at

> 定时执行任务

可以使用的关键字：minutes, hours, days, weeks, months, years

``` bash
# 在指定时刻执行程序
$ at 22:10 # 在22:10执行

at> touch file.txt # 在指定时间会执行任务
at> <EOT> # 按下了ctrl + D
job 1 at Sun Jun 14 22:10:00 2020

# 指定某天的某个时刻
$ at 22:10 12/9/20

# 指定10分钟以后执行
$ at now +10 minutes
```
#### atq

> at queue

列出当前正在等待执行的任务。它会显示执行时间这一列

#### atrm

> at remove

移除任务队列的任务

``` bash
# 列出任务队列
$ atq 
# 1       Sun Jun 14 22:10:00 2020 a root

# 移除任务
 $ atrm 1
```

#### sleep

> 休眠一会

单位有：m(minute), h(hour), d(day)

不写单位时，单位是秒

``` bash
# 休眠10s
$ touch file.txt; sleep 10; rm file.txt
```

#### && 和 ||

`&&`: 前一个命令成功，后一个才会执行

`||`: 前一个执行失败，后一个才会执行

`;`: 都执行，互不影响

#### crontab

> cron table，时间表。定时执行程序

##### 安装

``` bash
# 在Red hat中安装
$ yum install vixie-cron crontabs # 安装
$ chkconfig crond on # 设为开机自启动
$ service crond start # 启动

# 在Dibian中安装
$ apt install cron # 安装
$ service cron restart # 重启。或restart cron
```

修改.bashrc

``` bash
# 设置默认编辑器
$ echo "export EDITOR=nano" >> ~/.bashrc
```

##### 使用

`crontab`: 修改crontab文件

`cron`: 执行定时程序

``` bash
# 显示crontab文件
$ crontab -l

# 新增/编辑crontab文件
$ crontab -e

# 删除crontab文件
$ crontab -r
```

修改crontab文件：
``` vim
# m h dom mon dow command

# 每天22:10执行：家目录创建文件
10 22 * * * touch ~/file.txt

# 每周一周二周三的0点执行
0 0 * * 1,2,3 touch ~/file.txt

# 每两个小时的整点都执行
0 */2 * * *

# 每个月的1-15天的12点
0 12 1-15 * *
```

m: minute，分
h: hour，小时
dom: day of month，一个月的哪天
mon: month，月份
dow: day of week，一周的哪天
command: 要执行的命令

### 压缩和解压

打包：将多个文件变成一个总的文件。这个总的文件称作archive(归档)

压缩：将一个大文件通过某些压缩算法变成一个小文件

`tar`: 将多个文件归档为总的文件

`gzip`或`bzip2`: 将archive压缩为小文件

#### tar

> 将多个文件归档。注意参数可以不加 -

参数：
`c`: create，创建
`v`: verbose，冗余，会显示操作细节
`f`: file，指定归档文件
`x`: extract，提取

所以参数`cf`就可以创建

``` bash
$ mkdir compression

# 创建归档
$ tar -cvf folder.tar folder/

# 归档多个文件。不推荐，因为解压时没有文件夹
$ tar -cvf archive.tar f1.txt f2.txt

# 查看归档内容
$ tar -tf archive.tar

# 追加文件
$ tar -rvf archive.tar file.txt

# 解开归档
$ tar -xvf archive.tar
```

#### gzip & bzip2

> bzip2不常用。它压缩后的文件更大，也更耗时

`gzip`压缩后的文件后缀是`.tar.gz`
`bzip2`压缩后的文件后缀是`.tar.bz2`

`gunzip`：解压
`bunzip2`：解压

``` bash
# gzip压缩文件
$ gzip archive.tar # 生成archive.tar.gz，原文件消失

# bzip2压缩文件
$ bzip2 archive.tar # 生成archive.tar.bz2，原文件消失

# gunzip解压文件
$ gunzip archive.tar.gz # 生成archive.tar，原文件消失

# bunzip2解压文件
$ bunzip2 archive.tar.bz2 # 生成archive.tar，原文件消失
```

#### 使用tar一步到位

``` bash
# 直接用tar完成归档和压缩
$ tar -zcvf archive.tar.gz folder/

# 直接用tar完成解压和解归档
$ tar -zxvf archive.tar.gz

# 用tar完成归档和压缩：bz2类型
$ tar -jcvf archive.tar.bz2 folder/

# 用tar完成解压：bz2类型
$ tar -jxvf archive.tar.bz2
```

#### 直接查看压缩文件的内容

`zcat`, `zmore`, `zless`: 显示gz文件的内容

`bzcat`, `bzmore`, `bzless`: 显示bz2文件的内容

``` bash
$ zcat archive.tar.gz
```

#### zip和rar格式

`zip`, `unzip`

``` bash
# 安装zip
$ sudo yum install zip
$ sudo yum install unzip

# 解压
$ unzip archive.zip

# 不解压，查看内容
$ unzip -l archive.zip

# 压缩
$ zip -r archive.zip folder/ # r是递归，必须加
```

`rar`, `unrar`

> rar不支持yum直接安装，需要通过源代码编译生成可执行文件安装

安装
``` bash
# 获取压缩包
$ wget https://www.rarlab.com/rar/rarlinux-x64-5.7.0.tar.gz

$ ls # rarlinux-x64-5.7.0.tar.gz

$ tar zxvf rarlinux-x64-5.7.0.tar.gz

$ cd rar/

$ ls # readme.txt rar unrar makefile ..

$ sudo make # 不用make install了，makefile中安装完了

$ ls /usr/local/bin # rar unrar ..
```

使用
``` bash
# 压缩
$ rar a archive.rar folder/ # a 是 add

# 解压
$ unrar e archive.rar # e 是 extract 提取

# 不解压只看内容
$ unrar l archive.rar 
```

### 通过源码安装软件

安装方式：找.rpm后缀的安装包，用于Red hat（Debian的用.deb）

`alien`: 将rpm和deb安装包相互转换。但是不能保证可用，因为它的前置依赖可能不存在。

``` bash
# 安装alien
$ yum install alien

# 转换为rpm包
$ sudo alien -r xx.deb # 生成rpm文件

# 安装rpm包
$ rpm -i xx.rpm
```

或通过源码安装(.tar.gz文件)

``` bash
# 现在假设我们下载了一个源码
$ ls # htop-2.2.0.tar.gz

# 解压
$ tar zxvf htop-2.2.0.tar.gz

$ cd htop-2.2.0/

# 执行configure文件
$ ./configure 

# 这里居然说"missing libraries: libncurses"，装个库吧。正常流程忽略这一步
$ yum install ncurses-devel

# 执行configure
$ ./configure

# 编译源码
$ make # 会执行Makefile

# 安装
$ sudo make install

# 完成
$ ls usr/local/bin
```

## ssh

### 查看IP

#### ifconfig

> 比较旧的命令。属于net-tools

如果没有这个命令，执行`yum install net-tools`

#### ip addr

> 比较新的命令。属于iproute2

### ssh

> secure shell
> shell (贝壳)

Bash是一个Shell程序。

#### 加密算法

##### 对称加密

Symmetric-Key Encryption。加密和解密使用的是同一个密钥。这种方法的问题是：必须把密钥也通过网络传输给对方，比如客户端向服务器发送密码，同时也必须要发送密钥。

##### 非对称加密

Asymmetric-Key Encryption。这种方法把密钥也加密了。流程是：加密用公钥，解密用私钥；公钥和私钥总是成对出现。最有名的是`RSA`。

非对称加密的缺点：非常消耗电脑资源，要比对称加密慢100-1000倍。

##### 用ssh创建一个安全的通信管道

1. 两台电脑之间首先交换对称加密的密钥（用非对称加密的方式）
2. 之后用非对称加密来通信

流程：

服务器生成`公钥`和`私钥` ->

`公钥`明文传给客户端 ->

客户端使用`公钥`将`传输密钥`加密传给服务器 ->

服务器使用`私钥`解开`传输密钥` ->

双方都有了`传输密钥`（superKey），而且从未在网络上明文传输过

#### openssh

##### 连接到服务器

``` bash
# 安装openssh客户端
$ yum install openssh-clients
```

##### 将自己的电脑作为服务器

``` bash
# 安装openssh服务端
$ yum install openssh-server
```

安装完毕后，它会自动开启`sshd`这个Daemon Process(守护进程/精灵进程)。

或手动开启
``` bash
# 开启
$ sudo systemctl start sshd

# 停止
$ sudo systemctl stop sshd

# 重启
$ sudo systemctl restart sshd

# 查看状态
$ sudo systemctl status sshd

# 设置为开机自启
$ sudo systemctl enable sshd

# 在进程中查看
$ ps -ef | grep ssh

# 查看所有开机自启的软件
$ systemctl list-unit-files | grep enabled
```

#### 常用ssh软件

Windows：Putty, XShell, SecureCRT

Linux: openssh-clients

``` bash
$ ssh root@xx.xx.xx.xx
```

#### 用config文件配置ssh

> 客户端配置写在客户端，服务端配置写在服务端

ssh客户端配置：`/etc/ssh/ssh_config`（这个是系统级的）；用户级的在`~/.ssh/config`

ssh服务端配置：`/etc/ssh/sshd_config`

``` bash
# 查看手册
$ man sshd_config
$ man ssh_config

# 修改客户端配置文件
$ vim /etc/ssh/ssh_config

# 修改用户级的客户端配置文件
$ vim ~/.ssh/config # 如果不存在创建以下

# 修改完毕，要保证文件的权限为600
$ chmod 600 ~/.ssh/config

# 使用快速登录
$ ssh y
```

客户端配置文件示例：
```bash
Host y
    HostName xx.xx.xx.xx
    Port 22
    User root
    # 没写完往下看
```

服务端配置文件示例：

``` config
Port 22 # 默认值是22
# PermitRootLogin # 默认值是yes
# PasswordAuthentication # 默认yes
# PubkeyAuthentication # 默认yes
# PermitEmptyPasswords # 默认no
```

修改配置文件不会立即生效，除非重启服务

``` bash
$ sudo systemctl restart sshd
```

> 如果root用户家目录下没有.ssh文件夹，使用`ssh localhost`本地登录可以生成

#### 免密码登录ssh

> 用户名密码登录是基于口令的方法

公钥验证登录的步骤：

##### 1. 在客户机中生成密钥对（公钥和私钥）

`ssh-keygen`默认使用RSA非对称加密。

``` bash
# 生成密钥对
$ ssh-keygen # 等价于ssh-keygen -t rsa
```

会在~/.ssh/下生成id_rsa.pub和id_rsa，公钥和私钥

##### 2. 把公钥传输到服务器

`ssh-copy-id`传输公钥到服务器



``` bash
$ ssh-copy-id root@xx.xx.xx.xx # 等价于ssh-copy-id -i ~/.ssh/id_rsa.pub root@xx.xx.xx.xx

# -i可以指定文件
$ ssh-copy-id -i ~/.ssh/id_rsa_g.pub y # 写Host也可以
```

注意，指定特定的公钥文件时，需要配置~/.ssh/config文件：
``` config
Host g
  HostName xx.xx.xx.xx
  Port 22
  User root
  IdentityFile ~/.ssh/id_rsa_g
```

这个指令会在服务器端~/.ssh/生成或修改`authorized_keys`文件，把客户端的公钥填写进去

##### 3. 直接登录

不用输密码啦：

``` bash
$ ssh root@xx.xx.xx.xx

$ ssh y # 这个也可以
```

## 文本编辑和传输

### vim

`vim`有4种模式：交互模式，插入模式，命令模式，可视模式

#### 交互模式

> Interactive Mode | Normal Mode

这个模式不能输入文本。

按`h`：向左移动光标
按`j`：向下移动光标
按`k`：向右移动光标
按`l`：向上移动光标

按`0`：光标移动到当前行的行首

按$：光标移动到当前行的行尾

按^：光标移动到当前行文本的行首

按`w`：光标移动到下一个单词的位置


按`x`：删除光标所在的字符。如果先按`3`，再按`x`，则会删除3个字符

按`dd`：删除当前行。如果先按`3`，再按`dd`，则会删除3行
按`dw`：删除当前字符到下一个空格（光标在首字母上时才算是删除一个单词）。如果先按`3`，再按`dw`，则会删除3个单词
按`d0`：删除光标到行首的所有字符
按`d$`：删除光标到行尾的所有字符

按`yy`：和`dd`类似，但`yy`是复制，`dd`是剪切 # yank
`yw`：略
`y0`：略
`y$`：略

按`p`：粘贴。如果先按`3`，再按`p`，则是粘贴3次 # paste
按`r`：替换。比如按下`rs`，会把当前字符换成s # replace
按`R`：进入替换模式。直接写字符，它会替换当前位置的字符。Esc退出替换模式

按`u`：撤销操作 # undo
按`ctrl + r`：取消撤销 # redo
按`g`：跳转到指定行。比如按`3gg`或`3G`，跳转到第3行；`gg`跳转到第1行；`G`跳转到最后一行

按`/`：进入查找模式。从当前光标开始向文件末尾查找。如按`/see`，所有"see"会高亮。按`n`是下一个，按`N`是上一个
按`?`：进入查找模式。向文件开头查找。其他功能一样

按`:s`：查找并替换。如`:s/boy/girl`，boy会变成girl。`:s/boy/girl/g`，全行替换，注意是全行。`:2,4 s/boy/girl/g`，替换第2-4行。`:%s/boy/girl/g`，全局替换，最常用。

按`:r`：再光标处插入另一个文件的内容，Tab键可以补全文件名。如`:r file2.txt`。

按`:sp`：横向分屏，split。分屏会打开新的vim编辑窗口。按`ctrl + w + ctrl + w`，切换到下一个窗口。按`ctrl + w + ↑`，切换到上面的viewport，同样的有上下左右。按`ctrl  + w + +`，放大当前窗口，减号是缩小；按`ctrl + w + =`，重新分配大小；按`ctrl + w + r`，调换窗口位置；按`ctrl + w + R`，反向调换窗口位置。按`ctrl + w + q`或`ctrl + w + c`是退出当前窗口，和`:quit`或`:close`是同样的效果。按`ctrl + w + o`，只保留当前窗口，和`:only`是同样效果。

按`:!`：运行终端指令。如`:!ls`，会在vim模式下执行ls。

按`:vsp`：竖向分屏，vertically split。
#### 插入模式

> Insert Mode

从交互模式下按字母`i`或`a`或`o`都可以进。

按`i`：默认进入插入模式
按`I`：光标移动到当前行行首
按`a`：光标移动到下一个字符
按`A`：光标移动到当前行末尾
按`o`：在当前行下面增加一个空行，并将光标移动到新行
按`O`：在当前行上面增加一个空行，并将光标移动到新行
按`s`：删除光标所在字符，并进入插入模式
按`s`：删除当前行，并进入插入模式

#### 命令模式

> Command Mode

从交互模式下按冒号

`:w myfile`：保存为myfile。保存完毕后，会有提示语

`:q`：退出vim。如果文件被修改，却没有保存，需要使用`:q!`强制退出

`:x`：等价于`:wq`

`:set nu`：显示行号

#### 可视模式

> visual mode

从交互模式下按`v`。选中一些内容后按`d`可以删除。`x`也可以删除。`U`把文本变为大写，`u`把文本变成小写。

按`v`：默认进入
按`V`：区别是，选中的最小单位是行
按`ctrl + v`：块可视模式。选中的一块一块的。自己试试

#### vim配置

可以通过激活选项参数，或安装插件

命令模式可以设置参数：

``` bash
# 激活
$ :set 选项

# 取消激活
$ :set no选项

# 查看选项当前状态
$ :set 选项?
```

命令模式下的设置都是一次性的。所以需要修改配置。全局的配置文件是/etc/vimrc，当前用户的在家目录下：

``` bash
# 查看全局配置。vim的配置文件中，注释是用"符号，就是一个双引号
$ cat /etc/vimrc 

# 创建配置文件
$ touch ~/.vimrc

# 编辑
$ vim ~/.vimrc
```

``` vim
syntax on  " 语法高亮

set number " 显示行号

set showcmd " 显示当前命令

set ignorecase " 查找时忽略大小写

set mouse=a " 激活鼠标
```

大神的vim配置：https://github.com/amix/vimrc

### 版本控制

> Version Control

`SCM`: Source/Software Configuration Management，源码/软件配置管理。SVN、Mercurial、Git都称为SCM软件

#### 安装git

> `qgit`和`gitk`是git的图形界面软件

``` bash
$ yum install git

# 激活颜色选项
$ git config --global color.ui auto

# 关闭颜色选项
$ git config --global color.ui auto

# 激活制定项的颜色
$ git config --global color.diff auto
$ git config --global color.status auto
$ git config --global color.branch auto

# 配置用户名
$ git config --global user.name "goyave"

# 修改用户名
$ git config --global --replace-all user.name "goyave2"

# 配置邮箱
$ git config --global user.email "goyave4@163.com"

# 修改邮箱
$ git config --global --replace-all user.email "newemail"

# 查看config更多选项
$ man git-config

# 查看git配置
$ git config --list
# 或者通过家目录文件查看git配置
$ vim ~/.gitconfig

# 设置git别名
$ git config --global alias.co checkout # 之后'git co' 就等于 'git checkout'
```

Clone with SSH: 需要将公钥连通到github/gitlab上

```bash
# 生成密钥对
$ cd ~/.ssh 
$ ssh-keygen -C "注册github的邮箱"

# 此时会显示“Enter file in which to save the key”，可以写：~/.ssh/id_rsa_github，否则会覆盖以前的

# 把公钥粘贴到github

# 测试是否连通
$ ssh -i ~/.ssh/id_rsa_github -T git@github.com # 就是这个地址

# ssh可以使用-i，但是git不可以，所以还是在~/.ssh/config文件配置一下
```

```config
Host gitee.com
  IdentityFile ~/.ssh/id_rsa_github
  IdentitiesOnly yes
```


#### hexo

免费创建个人博客的库

### 文件传输

#### wget

> 下载文件

``` bash
# 安装
$ yum install wget

# 下载文件
$ wget file_url

# 断点续下：如果文件因为某些原因没下完，可以继续下载
$ wget -c file_url

# 查看手册
$ man wget
```

#### scp

> Secure Copy，安全拷贝。它基于ssh，所以两台电脑需要有ssh连接

``` bash
# 格式
$ scp source_file destination_file

# 其中，文件都用这样的格式：user@ip:file_name

# 示例：拷贝本地文件到服务器
$ scp file.txt root@xx.xx.xx.xx:/root

# 默认端口号是22。修改端口号
$ scp -P 7777 root@xx.xx.xx.xx:/root/file.txt .
```

#### ftp

> 著名的ftp软件：FileZila

``` bash
# 安装
$ yum install ftp

# 连接到一个公有服务器：https://debian.org/mirror/list
$ ftp -p ftp.fr.debian.org # -p是passive，被动的，区别于主动模式

# 此时让你输入用户名：anonymous，匿名的
# 让你输入密码：随便写，都能进

# 此时就进来啦。可以使用cd, ls, pwd等等
ftp>

# 下载
ftp> get file_name

# 上传
ftp> put my_file

# 执行自己电脑的指令：加个感叹号
ftp> !ls

# 查看ftp指令
$ man ftp

# 退出ftp：ctrl + d / bye / quit / exit
```

#### sftp

> ftp不安全，因为它的传输是明文。sftp是基于ssh的

它的操作和`ftp`一样

``` bash
# 连接服务器。默认端口是22，一般情况不用设置端口
$ sftp -oPort 7777 root@xx.xx.xx.xx

$ sftp y
```

#### rsync

> remote synchronize，远程同步。

用于增量备份。每一次只会备份新增/修改的文件。
可以备份到同一台电脑，也可以是两台电脑。`rsync`更像是智能的`scp`。

参数
`a`：archive，归档
`r`：recursive，递归
`v`：verbose，冗余，显示详细信息
`--delete`：同步删除操作。默认不会同步删除操作

``` bash
# 安装
$ yum install rsync

# 备份
$ rsync -arv images/ backups/

# 远程备份
$ rsync -arv --delete images/ root@xx.xx.xx.xx:backups/

# 查看手册
$ man rsync

# 更多具体的使用方式请百度
```

### IP

> Internet Protocol，互联网协议

IP地址：如xx.xx.xx.xx，这样的形式被称为`IPv4`（以小数点分隔）。每个数字的范围是0-255，即8位2进制（或2位16进制）。共2的32(4 * 8)次方个IP。

`IPv6`的格式：以冒号分隔的8组4位的16进制。共2的128(8 * 16)次方个IP。

每个IP可以绑定一个完整主机名。

#### 完整主机名

> Fully Qualified Domain Name(FQDN)。其实就主机名或域名都可以啦，不用区分那么细。

由主机名(host name)和域名(domain name)构成。如：

`www`是主机名，`github.com`是域名，`xx.xx...`是IP。

#### host

> 查看完整的IP地址

``` bash
$ host github.com # 查看它的IP
```

#### DNS

> Domain Name System，域名解析系统，它用来解析IP地址和域名

我们可以自定义自己电脑上的主机名和IP地址的对应关系

``` bash
$ sudo vim /etc/hosts
```

``` shell
# 格式为
127.0.0.1 localhost
192.168.9.187 console.internal.xx.xx
192.168.0.5 mia-laptop
```

修改hosts之后，可以直接用`ssh@mia-laptop`

#### whois

> 查看域名的登记信息，比如姓、名、地址、联系方式

``` bash
# 安装
$ yum install whois

# 查看信息
$ whois reveries.cc
```

#### ifconfig

> Network Interface Configuration，列出网络接口配置

``` bash
# 安装
$ yum install net-tools

# 列出网络接口
$ ifconfig 
```
旧版的网络接口是：
`eth0`：有线连接。eth是Ethernet(以太网)
`lo`：本地回环（Local Loopback），对应一个虚拟网卡。它的IP地址是127.0.0.1
`wlan0`：Wi-Fi无线连接。Wireless Local Area Network，无线局域网。

内容中：
`RX`：接收数据包
`TX`：发送数据包(Transmit)
`inet`：IPv4地址
`inet6`：IPv6地址
`ether`：MAC地址

接口激活/关闭：
``` bash
# ifconfig interface state
$ ifconfig enp0s3 down # 关闭enp0s3接口。up是开
```

### netstat

> net statistics，网络统计。它属于net-tools包。

``` bash
# 列出所有网络接口的统计信息
$ netstat -i
```

显示列的信息：
`MTU`：Maximum Transmission Unit，最大传输单元
`RX-OK`：在此接口接收的包中正确的包数
`RX-ERR`：在此接口接收的包中错误的包数
`RX-DRP`：在此接口接收的包中丢弃的包数。drop
`RX-OVR`：在此接口接收的包中，由于过速，丢失的包数。
`TX-OK`：在次接口库发送的包中正确的包数
`Flg`：Flag

列出所有开启的连接

> -u: 显示UDP连接
> -t: 显示TCP连接
> -a: all，所有
> -l: listen，只列出状态是listen的连接
> -s: summary，只列出总结性的信息

``` bash
$ netstat -uta
```

### ss

> ss是iproutes的命令。它和netstat作用类似。

### 端口

`80`是网页，HTTP的
`443`是网页，https的
`21`是文件，FTP
`110`是邮件

### emacs

> 文本编辑软件

## Shell脚本

### shell

    > 壳。终端命令行环境。它内部是计算机硬件和内核，外部是用户

`Sh`：Bourne Shell。所有Shell的祖先

`Bash`：Bourne Again Shell

`Ksh`

`Csh`：C Shell。语法类似C语言

`Tcsh`：Csh的升级版

`Zsh`：比较新的Shell，集Bash、Ksh和Tcsh一体。见https://github.com/ohmyzsh/ohmyzsh

``` bash
# 安装shell
$ yum install ksh

# 进入shell
$ sh

# 切换shell
$ chsh # 输密码，然后输入/bin/sh
```

### shell脚本

`#!`称作`sha-bang`或`shebang`，使系统以/bin/bash来执行脚本。

``` bash
# 创建test.sh
$ touch test.sh 

# 默认创建的文件的权限大概是666，所以需要添加执行权限
$ chmod +x test.sh # 变成了777

$ vim test.sh
```

``` vim
#!/bin/bash

ls
```

运行
``` bash
# 直接运行
$ ./test.sh
$ bash test.sh

# 以调试模式运行
$ bash -x test.sh
```

#### PATH

> 环境变量。里面存放了所有可以直接运行文件的目录的值

``` bash
# 打印PATH
$ echo $PATH #多个目录用分号相隔
```

#### shell变量

创建一个a.sh文件
``` bash
#!/bin/bash

# 别加空格
msg='hello world'
msg2="我是引号\'，嘻嘻"

echo "msg is $msg"
```

在shell脚本中，`$#`表示参数的数量，`$0`是被运行的脚本的名称，`$1`是第1个参数。
但是，如果脚本中有shift（不加引号），$1就会往后移动。如

``` bash
echo "First param is $1"
shift
echo "Second param is $1" # 实际上打印的是$2
```

#### 定义数组

> Bash的下标是从0开始，Csh、Zsh的下标是从1开始的

``` bash
$ array=('a' 'b' 'c')

# 跳过赋值
$ array[5]='f'

# 访问
$ echo ${array[2]} # c

# 全部访问
$ echo ${array[*]} # a b c f
```

#### echo

> 回声。打印用

``` bash
$ echo Hello World # 这里实际上给echo传入了两个参数

# 使用转义字符
$ echo -e "line1\nline2"

# 打印变量
$ echo $var
```

#### 引号

单引号：忽略所有被它括起来的特殊字符
双引号：忽略大多数字符，不包括美元符、反引号、反斜线等
反引号：反引号的内容会被shell执行

#### read

> 读取终端的输入

-p：提示语
-n：最大字符数
-t：限制输入时间(单位s)
-s：不显示输入内容。secret

比如read.sh
``` bash
read -p 'Please input your name: ' -n 5 -t 5 name

echo "Hello, $name"
```

#### let

bash脚本默认没法数学计算。let可以直接执行运算。

``` bash
$ let 'a=1,b=2'

# 打印
$ let "$a+$b"
```

#### bc

> 运算小数。

``` bash
$ man bc
```

#### env

> 打印所有的环境变量/全局变量

``` bash
$ env

# 查找PWD的值
$ env | grep PWD
```

#### export

可以添加全局变量。修改~/.bashrc

``` bash
$ echo 'export EDITOR=vim' > ~/.bashrc

# 保存
$ source ~/.bashrc

# 查看
$ env | grep EDITOR
```

#### if语句

``` bash
#!/bin/bash

name=$1

if [ name='mia' ] # 条件的两边必须有空格，且等号为一个
then
  echo 'This is Mia'
elif [ name='Jack' ]
  echo 'This is Jack'
else
  echo 'This is no one'
fi # 表示if语句的结束
```

判断条件

字符串：
`$str1=$str2`：两个字符串是否相等
`$str1!=$str2`：两个字符串是否不相等
`-z $str`：字符串是否为空。zero
`-n $str`：字符串是否不为空。no

数字：
`$num1 -eq $num2`：两个数字相等
`$num1 -ne $num2`：两个数字不相等
`$num1 -lt $num2`：小于
`$num1 -le $num2`：小于等于
`$num1 -gt $num2`：大于
`$num1 -ge $num2`：大于等于

文件：
`-e $file`：文件存在
`-d $file`：目标是目录
`-f $file`：目标是文件
`-L $file`：文件是链接
`-r $file`：文件可读
`-w $file`：文件可写
`-x $file`：文件可执行
`$file1 -nt $file2`：文件1比文件2更新。newer than
`$file1 -ot $file2`：文件1比文件2更旧。older than

逻辑与、逻辑或和逻辑非：
`if [ 1 -lt 2 ] && [ 3 -lt 4 ]`：逻辑与
`if [ ! 1 -lt 2 ]`：逻辑非

case语句（其他语言的switch）：
``` bash
case $1 in
  "Mia" | "mia" | "mi")
    echo "This is Mia"
    ;;
  "Jack")
    echo 'This is Jack'
    ;;
  *)
    echo 'This is no one'
    ;;
esac
```

#### 循环

``` bash
# while循环
a=1
while [ $a -lt 10 ]
do
  echo $a
  let "a=$a+1"
done

# until循环
b=1
until [ $b -eq 10 ]
do
  echo $b
  let "b=$b+1"
done

# for循环：像编程语言里的for-in
for file in `ls`
do
  echo "File found: $file"
done

# 利用seq命令
for i in `seq 1 10`
do
  echo $i
done
```

#### 函数

> function

函数的圆括号不能传任何参数。
函数调用必须在定义之前。
只能返回数字，不能返回字符串。通常0代表成功，其他代表失败。

``` bash
# 定义
function f1 {
  #
}
# 定义2
f2 () {
  #
}

# 调用
f1
f2

# 定义带参数的函数
print1 () {
  echo Hello $1
  return 8
}

# 执行
print1 Mia
```

此时可以在bash中拿到返回值：
``` bash
$ echo $?
```

#### 局部变量

变量默认都是全局的。
第一次给变量赋值时，前面加上`local`

#### shell脚本示例

生成一个html页面：

``` bash
# 安装图片处理包
$ yum install ImageMagick

# 有了convert。
# 生成缩略图
$ convert a.png -thumbnail '200x200>' ./imgs/a.png # >号表示，如果原始图片尺寸已经小于200*200，则直接使用原图
```

gallery.sh:

``` sh
#!/bin/bash

# If no parameter, use a default value
if [ -z $1 ]
then
  output='gallery.html'
else
  output=$1
fi

# Preparation of files and folders
echo '' > $output

if [ ! -e thumbnails ]
then
  mkdir thumbnails
fi

# Beginning of HTML
echo '<!DOCTYPE html>
<html>
  <head>
    <title>My Gallery</title>
  </head>
  <body>
    <p>' >> $output
    
# Generation of thumbnails and the HTML web page
for image in `ls *.jpg *.png *.jpeg *.gif 2>/dev/null`
do
  convert $image -thumbnail '200x200>' thumbnails/$image
  echo " <a href=\"$image\"><img src=\"thumbnails/$image\" alt=\"\" /></a>" >> $output
done

# End of HTML
echo '   </p>
  </body>
</html>' >> $output
```

#### 用shell做统计练习

``` shell
# 创建一个文本
$ vim words.txt
```

统计示例：

``` bash
# i是忽略大小写，o是只显示要匹配的文本
# 会打印所有匹配的a
# 通过管道统计所有的行
grep -io a words.txt | wc -l
```

写sh：

``` sh
#!/bin/bash

# Verification of parameter
if [ -z $1 ]
then
  echo "Please enter the file of directory."
  exit
fi

# Verification of file existence
if [ ! -e $1 ]
then
  echo "Please make sure that the file of dictionary exists."
  exit
fi

# Definition of function
statistics () {
  for char in {a..z}
  do
    echo "$char - `grep -io "$char" $1 | wc -l`" | tr /a-z/ /A-Z/ >> tmp.txt
    sort -rn -k 2 -t - tmp.txt
    rm tmp.txt
  done
}

# Use of function
statistics $1
```

#### 利用函数来重载命令

`command ls`表示执行命令`ls`。

``` bash
#!/bin/bash

ls () {
  command ls -lh
}

ls
```

### 进程守护

Centos 7 以后，PID为1的进程是`systemd`。
PID或PPID为1的进程只在系统关闭时才会销毁。所有Parent Process ID为1的进程被称为守护进程（Daemon）。

守护进程的结尾通常会有一个'd'字母，如systemd, httpd, smbd

systemd提供`systemctl`命令，它使得我们可以管理Unit。

`systemctl`（以toto服务为例）：
``` bash
# 启动
$ systemctl start toto
# 停止
$ systemctl stop toto
# 重启
$ systemctl restart toto
# 查看服务状态
$ systemctl status toto
# 重载配置文件，不重启
$ systemctl reload toto
# 开机自动启动
$ systemctl enable toto
# 开机不自动启动
$ systemctl disable toto
# 查看服务是否开机自启动
$ systemctl is-enabled toto

# 遮盖。遮盖后无法再设置启动
$ systemctl mask toto
# 取消遮盖
$ system unmask toto

# 查看各个级别下服务的启动和禁用情况
$ systemctl list-unit-files --type=service
# 列出所有的活动单元
$ systemctl list-units

# 列出类型为service的单元
$ systemctl list-units --type=service

# 列出类型为target的所有单元
$ systemctl list-units --type=target --all

# 列出所有对指定target的依赖
$ systemctl list-dependencies graphical.target

# 查看默认target
$ systemctl get-default

# 设置默认target
$ systemctl set-default graphical.target
```

#### samba

> SMB（Server Messages Block，信息服务块）是一种在局域网上共享文件和打印机的一种通信协议

``` bash
# 安装
$ yum install samba

# 启动。注意是smb，不是samba
$ systemctl start smb

# 查看
$ ps -aux | grep smb

# 查看
 $ systemd cat smb.service
 
 # 编辑smb.service文件。它实际上不会修改原文件，只会创建一个新文件去重载
 $ systemd edit smb.service
 
 # 编辑smb.service原文件
 $ systemd edit --full smb.service
 # 重载守护进程
 $ systemctl daemon-reload
```

#### journalctl

> 显示systemd管理的所有日志

`-b`：显示上一次登录时的日志
`-k`：显示内核的日志。kernel
`-u`：只显示指定服务的日志。unit

``` bash
# 按时间顺序
$ journalctl
```

#### systemd-analyze

``` bash
# 开机分析
$ systemd-analyze

# 详细分析
$ systemd-analyze blame
```

### Apache

> web服务器
> Red Hat中是httpd，Debian是apache

``` bash
# 安装
$ yum install httpd 

# 启动
$ systemctl start httpd

# 如果80端口被墙，可以打开
$ firewall-cmd --zone=public --add-port=80/tcp --permanent
# 立即生效
$ firewall-cmd --reload
# 列出开放端口
$ firewall-cmd --list-ports
# 移除端口
$ firewall-cmd --zone=public --remove-port=80/tcp --permanent
```

#### firewalld

> 防火墙

``` bash
# 停止防火墙。慎用
$ systemctl stop firewalld
```

#### 配置Apache服务器

##### 文件路径

服务目录：/etc/httpd
主配置文件：/etc/httpd/conf/httpd.conf
网站数据目录：/var/www/html
访问日志：/var/log/httpd/access_log
错误日志：/var/log/httpd/error_log

##### 参数

ServerRoot 服务目录
Listen 监听的IP地址与宽口号
User 运行服务的用户
Group 运行服务的群组
ServerAdmin 管理员邮箱
DocumentRoot 网站数据目录
Directory 网站数据目录的权限（这是一个局部配置，写法是<Directory></Directory>）
DirectoryIndex 默认的索引页页面
ErrorLog 错误日志文件
CustomLog 访问日志文件

##### 更换目录

修改DocumentRoot字段。但是由于SELinux的限制，修改之后也不会生效。

### SELinux子系统

> Security-Enhanced Linux，安全增强型Linux，是MAC的安全子系统

`MAC`：Mandatory Access Control，强制访问控制，指一种由操作系统约束的访问控制。

#### Selinux的双重保险

域限制（Domain Limitation）：对服务程序的功能进行限制
安全上下文（Security Context）：对文件资源的访问限制

#### 和防火墙的区别

防火墙是防盗门，SeLinux是保险柜

#### 3种配置模式

`enforcing`：强制执行
`permissive`：警告但不拦截
`disabled`：关闭

``` bash
# 查看SELinux的状态
$ sestatus

$ sestatus -v # verbose

# 获取当前SELinux的模式
$ getenforce

# 设置模式为permissive
$ setenforce 0

# 设置模式为enforcing
$ setenforce 1
```

##### 安全上下文

``` bash
$ ls -Zd /var/www/html
```

会打印其安全上下文是：system_u:object_r:httpd_sys_content__t:s0，分别表示系统进程身份、文件目录角色、**类型**、其他

##### semanage

> SELinux Manage，管理SELinux的策略

参数：
`-l`：查询。list
`-a`：添加。add
`-m`：修改。modify
`-d`：删除。delete
`-t`：类型。type


现在我们修改了Apache的服务目录为/home/my_www。需要配置SELinux来使其生效。
``` bash
#  如果没有该命令：查看哪个包提供了此命令
$ yum provides semanage # policycoreutils-python

# 安装

# 修改目录的安全上下文类型
$ semanage fcontext -a -t httpd_sys_content__t /home/my_www
$ semanage fcontext -a -t httpd_sys_content__t /home/my_www/*

# 使生效
$ restorecon -Rv /home/my_www # restore context。-R是递归，v是verbose
```