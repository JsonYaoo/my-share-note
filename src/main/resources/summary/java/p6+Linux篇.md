# **十五、Linux 篇**

### 1.1. 虚拟机配置模板

```shell
# 1、修改静态IP：
vi /etc/sysconfig/network-scripts/ifcfg-ens33

TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=static
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=ens33
UUID=8ebf0bf9-240e-4de4-80b0-8ae2a2b609d4
DEVICE=ens33
ONBOOT=yes
IPADDR=192.168.1.140
NETMASK=255.255.255.0
GATEWAY=192.168.1.1
DNS1=192.168.1.1
	
# 2、重启网络：
systemctl restart network
ping www.baidu.com

# 3、修改HostNme：
vi /etc/hostname
node-140

# 4、修改Hosts：
vi /etc/hosts
192.168.1.140 node-140

# 5、重启虚拟机，改用ssh连接
reboot

# 6、安装vim：
yum -y install vim

# 7、安装ifconfig：
yum -y install net-tools

# 8、安装telnet
yum list telnet*
yum -y install telnet-server.*
yum -y install telnet

# 9、安装unzip：
yum -y install unzip zip

# 10、安装nslookup：
yum -y install bind-utils

# 11、安装killall
yum -y install psmisc

# 12、安装wget
yum -y install wget

# 13、安装rz sz
yum -y install lrzsz

# 14、安装lsof
yum -y install lsof

# 15、安装git
yum -y install git

# 16、同步阿里云时间
yum -y install ntp
ntpdate -u time1.aliyun.com
vi /etc/crontab
*  *  *  *  * root       /usr/sbin/ntpdate -u time1.aliyun.com

# 17、安装解压缩.tar.bz2支持
yum -y install bzip2

# 18、永久关闭防火墙
systemctl disable firewalld
```

### 1.2. 文件目录

![1660043488037](D:\Users\yaocs2\AppData\Roaming\Typora\typora-user-images\1660043488037.png)

| 目录 | 作用                                                         |
| ---- | ------------------------------------------------------------ |
| bin  | 存放二进制可执行文件（ls，cat，mkdir 等）                    |
| boot | 存放用于系统引导时使用的各种文件                             |
| dev  | 存放设备文件                                                 |
| etc  | 存放系统配置文件                                             |
| home | 存放所有用户文件的根目录                                     |
| lib  | 存放跟文件系统中的程序运行所需要的共享库及内核模块           |
| proc | 虚拟文件系统，存放当前内存的映射                             |
| usr  | 存放系统应用程序，比较重要的目录 /usr/local（管理员软件安装目录） |
| var  | 存放运行时需要改变数据的文件                                 |
| mnt  | 挂载目录                                                     |
| sbin | 是系统管理员专用的二进制代码存放目录，主要用于系统管理       |
| root | 超级用户主目录                                               |
| opt  | 额外安装的可选应用程序包安装位置                             |

### 1.3. 基本命令 - 文件查看

#### 1）pwd

查看当前所在的路径

```shell
[root@localhost ~]# pwd
/root
```

#### 2）ls

列出当前目录下的所有文件

```shell
[root@localhost ~]# ls
anaconda-ks.cfg
```

#### 3）ll

ls -l 的缩写，用于列出当前目录下的文件（带文件信息）。

```shell
[root@localhost ~]# ll
total 4
-rw-------. 1 root root 1243 Mar 28 20:59 anaconda-ks.cfg

[root@localhost ~]# ls -l
total 4
-rw-------. 1 root root 1243 Mar 28 20:59 anaconda-ks.cfg
```

![1660045420030](D:\Users\yaocs2\AppData\Roaming\Typora\typora-user-images\1660045420030.png)

| 段数 | 文件属性                                                     |
| ---- | ------------------------------------------------------------ |
| 1    | 第一个字符代表文件（-）、目录（d），链接（l）<br/>其余字符每 3 个一组（rwx），读（r）、写（w）、执行（x) <br/>第一组：文件所有者的权限 <br/>第二组：与文件所有者同组用户的权限 <br/>第三组：不与文件所有者同组的其他用户的权限也可用数字表示为：r=4，w=2，x=1，如：权限 6 可以表示为 r + w = 6 |
| 2    | 目录/链接个数 <br/>对于目录文件，表示它的第一级子目录的个数 <br/>注意：此处的值要减 2 才等于该目录下的子目录的实际个数，因为目录下默认包含 . 和 .. 这两个目录，而对于其他文件，默认是 1 |
| 3    | 所属用户                                                     |
| 4    | 所属组                                                       |
| 5    | 文件大小（字节）                                             |
| 6    | 最后修改时间                                                 |
| 7    | 文件/文件夹名称                                              |

#### 4）ll -a

列出当前目录下的所有文件（包括隐藏文件）

```shell
[root@localhost ~]# ll -a
total 28
dr-xr-x---.  2 root root  135 Mar 28 21:00 .
dr-xr-xr-x. 17 root root  224 Mar 28 20:58 ..
-rw-------.  1 root root 1243 Mar 28 20:59 anaconda-ks.cfg
-rw-------.  1 root root    8 Mar 28 21:00 .bash_history
-rw-r--r--.  1 root root   18 Dec 29  2013 .bash_logout
-rw-r--r--.  1 root root  176 Dec 29  2013 .bash_profile
-rw-r--r--.  1 root root  176 Dec 29  2013 .bashrc
-rw-r--r--.  1 root root  100 Dec 29  2013 .cshrc
-rw-r--r--.  1 root root  129 Dec 29  2013 .tcshrc
```

#### 5）ll -help

查看 ls 用法，–help 是一个帮助命令

```shell
[root@localhost ~]# ll --help
Usage: ls [OPTION]... [FILE]...
List information about the FILEs (the current directory by default).
Sort entries alphabetically if none of -cftuvSUX nor --sort is specified.

Mandatory arguments to long options are mandatory for short options too.
  -a, --all                  do not ignore entries starting with .
.............
```

### 1.4. 基本命令 - 创建、重命名文件/文件夹

#### 1）touch

创建空文件

```shell
[root@localhost ~]# touch hello.txt
[root@localhost ~]# ll
total 4
-rw-------. 1 root root 1243 Mar 28 20:59 anaconda-ks.cfg
-rw-r--r--. 1 root root    0 Mar 28 22:21 hello.txt
```

#### 2）mkdir

创建目录

```shell
[root@localhost ~]# mkdir abc
[root@localhost ~]# ll
total 4
drwxr-xr-x. 2 root root    6 Mar 28 22:22 abc
-rw-------. 1 root root 1243 Mar 28 20:59 anaconda-ks.cfg
-rw-r--r--. 1 root root    0 Mar 28 22:21 hello.txt
```

#### 3）mkdir -p

目标目录存在也不报错，当我们在创建目录时，如果不确定这个目录是否已存在的话，可以使用 -p 参数， 这样就算目录已存在也不会报错，而如果不指定 -p 参数，则会报错，会提示目录已存在。

```shell
[root@localhost ~]# mkdir -p abc
[root@localhost ~]# ll
total 4
drwxr-xr-x. 2 root root    6 Mar 28 22:22 abc
-rw-------. 1 root root 1243 Mar 28 20:59 anaconda-ks.cfg
-rw-r--r--. 1 root root    0 Mar 28 22:21 hello.txt
[root@localhost ~]# mkdir abc
mkdir: cannot create directory ‘abc’: File exists
```

#### 4）mv

重命名文件\文件夹

```shell
[root@localhost ~]# ll
total 4
drwxr-xr-x. 2 root root    6 Mar 28 22:22 abc
-rw-------. 1 root root 1243 Mar 28 20:59 anaconda-ks.cfg
-rw-r--r--. 1 root root    0 Mar 28 22:21 hello.txt

[root@localhost ~]# mv abc abx
[root@localhost ~]# ll
total 4
drwxr-xr-x. 2 root root    6 Mar 28 22:22 abx
-rw-------. 1 root root 1243 Mar 28 20:59 anaconda-ks.cfg
-rw-r--r--. 1 root root    0 Mar 28 22:21 hello.txt
```

### 1.5. 基本命令 - 链接文件

Linux 有两种链接，分别是硬链接和符号（软）链接。

1. 软链接：功能类似类似于 windows 的快捷方式，主要用于节省磁盘空间。
2. 硬链接：相当于对原始文件的一个复制，但不能对目录使用硬链接。

#### 1）ln

创建硬链接

```shell
[root@localhost ~]# ll
total 4
drwxr-xr-x. 2 root root    6 Mar 28 22:22 abx
-rw-------. 1 root root 1243 Mar 28 20:59 anaconda-ks.cfg
-rw-r--r--. 1 root root    0 Mar 28 22:21 hello.txt

[root@localhost ~]# ln hello.txt hlink
[root@localhost ~]# ll
total 4
drwxr-xr-x. 2 root root    6 Mar 28 22:22 abx
-rw-------. 1 root root 1243 Mar 28 20:59 anaconda-ks.cfg
-rw-r--r--. 2 root root    0 Mar 28 22:21 hello.txt
-rw-r--r--. 2 root root    0 Mar 28 22:21 hlink
```

#### 2）ln -s

创建软链接，相当于快捷方式，删除时不会删除原文件

```shell
[root@localhost ~]# ll
total 4
drwxr-xr-x. 2 root root    6 Mar 28 22:22 abx
-rw-------. 1 root root 1243 Mar 28 20:59 anaconda-ks.cfg
-rw-r--r--. 2 root root    0 Mar 28 22:21 hello.txt
-rw-r--r--. 2 root root    0 Mar 28 22:21 hlink

[root@localhost ~]# ln -s hello.txt  vlink
[root@localhost ~]# ll
total 4
drwxr-xr-x. 2 root root    6 Mar 28 22:22 abx
-rw-------. 1 root root 1243 Mar 28 20:59 anaconda-ks.cfg
-rw-r--r--. 2 root root    0 Mar 28 22:21 hello.txt
-rw-r--r--. 2 root root    0 Mar 28 22:21 hlink
lrwxrwxrwx. 1 root root    9 Mar 28 22:33 vlink -> hello.txt
```

### 1.6. 基本命令 - 切换目录

#### 1）cd .

进入当前目录

```shell
[root@localhost ~]# pwd
/root

[root@localhost ~]# cd .
[root@localhost ~]# pwd
/root
```

#### 2）cd ..

进入上一级目录

```shell
[root@localhost ~]# pwd
/root

[root@localhost ~]# cd ..
[root@localhost /]# pwd
/
```

#### 3）cd /

进入根目录

```shell
[root@localhost /]# cd /
[root@localhost /]# pwd
/
```

#### 4）cd [dir]

指定 dir 目录，可以切换到指定目录

```shell
[root@localhost ~]# cd /bin/
[root@localhost bin]# pwd
/bin
```

#### 5）cd ~

去当前用户主目录（HOME ）

```shell
[root@localhost bin]# cd ~
[root@localhost ~]# pwd
/root
```

### 1.7. 基本命令 - 删除文件/文件夹

#### 1）rm

删除文件，但是会有提示确认对话，此时输入 y 确认删除

```shell
[root@localhost test]# touch abc.txt
[root@localhost test]# ll
total 0
-rw-r--r--. 1 root root 0 Mar 29 13:53 abc.txt

[root@localhost test]# rm abc.txt
rm: remove regular empty file ‘abc.txt’? y
[root@localhost test]# ll
total 0
```

#### 2）rm - r

删除目录，此时需要指定 r 参数，否则会提示不能删除。

1. r 是给 rm 加入递归（recursion）的特性，也就是目标为文件夹时，删除文件夹下所有数据。
2. 使用 rm -r 在删除目录时，也会有提示确认对话，此时输入 y 确认删除。

```shell
[root@localhost abx]# ll
total 0
drwxr-xr-x. 2 root root 6 Mar 29 14:01 test

[root@localhost abx]# rm test
rm: cannot remove ‘test’: Is a directory

[root@localhost abx]# rm -r test
rm: remove directory ‘test’? y
[root@localhost abx]# ll
total 0
```

#### 3）rm -f

强制删除

1. f 是给 rm 加入强制（force）特性，也就是遇到删除时，不需要询问即可直接删除。
2. 但注意，这个操作还是比较危险的，建议慎用，因为删除之后就找不到了，因为 Linux 系统中，是没有回收站的。

```shell
[root@localhost abx]# touch a.txt
[root@localhost abx]# ll
total 0
-rw-r--r--. 1 root root 0 Mar 29 14:03 a.txt

[root@localhost abx]# rm -f a.txt 
[root@localhost abx]# ll
total 0
```

#### 4）rm -rf

递归删除目录及其文件

1. Linux中最危险的操作，最具破坏性。
2. rf 参数可以强制递归删除任何数据，并且没有任何提示，慎用！慎用！慎用！

```shell
[root@localhost ~]# ll
drwxr-xr-x. 2 root root    6 Mar 29 14:03 abx
-rw-------. 1 root root 1243 Mar 28 20:59 anaconda-ks.cfg

[root@localhost ~]# mkdir -p abx/test/aaa
[root@localhost ~]# cd abx/test/aaa/
[root@localhost aaa]# touch a.txt

[root@localhost aaa]# cd ~
[root@localhost ~]# rm -rf abx
[root@localhost ~]# ll
-rw-------. 1 root root 1243 Mar 28 20:59 anaconda-ks.cfg
```

### 1.8. 基本命令 - 复制/粘贴/剪切

#### 1）cp

复制 & 粘贴文件

```shell
[root@localhost ~]# ll
-rw-r--r--. 1 root root    0 Mar 28 22:21 hello.txt

[root@localhost ~]# cp hello.txt  hello-bak.txt  
[root@localhost ~]# ll
-rw-r--r--. 1 root root    0 Mar 29 14:20 hello-bak.txt
-rw-r--r--. 1 root root    0 Mar 28 22:21 hello.txt
```

#### 2）cp -r

复制 & 粘贴文件或目录，其中复制目录，需要指定 r 参数。

```shell
[root@localhost ~]# mkdir abc
[root@localhost ~]# ll
drwxr-xr-x. 2 root root    6 Mar 29 14:21 abc

[root@localhost ~]# cp abc xyz 【错误用法，复制目录必须指定-r参数】
cp: omitting directory ‘abc’

[root@localhost ~]# cp -r abc xyz
[root@localhost ~]# ll
drwxr-xr-x. 2 root root    6 Mar 29 14:21 abc
drwxr-xr-x. 2 root root    6 Mar 29 14:21 xyz
```

#### 3）mv

移动（剪切）文件或目录、以及重命名文件/文件夹

```shell
[root@localhost ~]# ll
drwxr-xr-x. 2 root root    6 Mar 29 14:21 abc
drwxr-xr-x. 2 root root    6 Mar 29 14:21 xyz
[root@localhost ~]# ll abc/
total 0

[root@localhost ~]# mv xyz abc 
[root@localhost ~]# ll abc/
drwxr-xr-x. 2 root root 6 Mar 29 14:21 xyz
```

#### 4）scp [src] [dest]

scp 命令，有 Security 的文件 copy，基于 ssh 登录，如果没有配置则免密码登陆，否则需要输入主机密码登录，用于在网络中不同主机之间复制文件或目录。

```shell
[root@localhost ~]# scp /root/hello.txt  192.168.182.130:/root/
The authenticity of host '192.168.182.130 (192.168.182.130)' can't be established.
ECDSA key fingerprint is SHA256:uUG2QrWRlzXcwfv6GUot9DVs9c+iFugZ7FhR89m2S00.
ECDSA key fingerprint is MD5:82:9d:01:51:06:a7:14:24:a9:16:3d:a1:5e:6d:0d:16.
Are you sure you want to continue connecting (yes/no)? yes 【第一次会提示此信息，输入yes即可，以后就不提示了】
Warning: Permanently added '192.168.182.130' (ECDSA) to the list of known hosts.
root@192.168.182.130's password: 【在这里输入192.168.182.130机器的密码即可】
hello.txt                        100%    0     0.0KB/s   00:00
```

#### 5）scp -v

显示进度

#### 6）scp -r

复制目录

#### 7）scp -q

静默复制模式

```shell
[root@localhost ~]# scp -rq /root/abc/ 192.168.182.130:/root/
root@192.168.182.130's password:【在这里输入192.168.182.130机器的密码即可】
```

### 1.9. 基本命令 - 权限分配

#### 1）chmod u+x

给当前所有者，添加执行权限【x表示是执行权限】

```shell
[root@localhost ~]# ll
-rw-r--r--. 1 root root    0 Mar 28 22:21 hello.txt

[root@localhost ~]# chmod u+x hello.txt 
[root@localhost ~]# ll
-rwxr--r--. 1 root root    0 Mar 28 22:21 hello.txt
```

#### 2）chmod 777

添加 rwx rwx rwx 权限

```shell
[root@localhost ~]# chmod 777 hello.txt
[root@localhost ~]# ll
-rwxrwxrwx. 1 root root    0 Mar 28 22:21 hello.txt
```

#### 3）chmod -R 777

给指定目录及其子目录，递归地添加 rwx rwx rwx 权限

```shell
[root@localhost ~]# ll
drwxr-xr-x. 3 root root   17 Mar 29 14:24 abc

[root@localhost ~]# cd abc/
[root@localhost abc]# ll
total 0
drwxr-xr-x. 2 root root 6 Mar 29 14:21 xyz

[root@localhost abc]# cd ..
[root@localhost ~]# chmod -R 777 abc
[root@localhost ~]# ll
drwxrwxrwx. 3 root root   17 Mar 29 14:24 abc
[root@localhost ~]# cd abc/
[root@localhost abc]# ll
total 0
drwxrwxrwx. 2 root root 6 Mar 29 14:21 xyz
```

### 2.0. 基本命令 - 内容查看

#### 1）cat

显示文本内容（顺序输出）

```shell
[root@localhost ~]# cat anaconda-ks.cfg 
#version=DEVEL
#System authorization information
auth --enableshadow --passalgo=sha512
......
```

#### 2）cat -b

显示行号输出

```shell
[root@localhost ~]# cat -b anaconda-ks.cfg 
     1  #version=DEVEL
     2  # System authorization information
     3  auth --enableshadow --passalgo=sha512
     4  # Use CDROM installation media
........
```

#### 3）more

1. 分屏显示，根据屏幕大小显示一屏内容，一次显示一屏，没有显示完时最后一行则显示进度。
2. 回车显示下一行，按 b 显示上一页，空格显示下一页，q 退出。

```shell
[root@localhost ~]# more anaconda-ks.cfg 
#version=DEVEL
#System authorization information
auth --enableshadow --passalgo=sha512
#Use CDROM installation media
cdrom
#Use graphical install
graphical
#Run the Setup Agent on first boot
firstboot --enable
ignoredisk --only-use=sda
#Keyboard layouts
keyboard --vckeymap=us --xlayouts='us'
#System language
lang en_US.UTF-8

#Network information
network  --bootproto=dhcp --device=ens33 --ipv6=auto --activate
network  --hostname=localhost.localdomain

#Root password
rootpw --iscrypted $6$sx2fOWF8AMiHgQTV$ExkpEX6Sq1EfZVHaP4RxfYls3B0o
dX2ouFfaTX2S0TttzWz7tX3L3cWRFeb1M4qfGUA2FGzrkylhlGfp4psze.
--More--(48%)
```

### 2.1. 基本命令 - 压缩、解压

| 参数 | 作用                                                         |
| ---- | ------------------------------------------------------------ |
| -z   | 是否同时具有 gzip 的属性？亦即是否需要用 gzip 压缩？         |
| -c   | 创建一个压缩文件的参数指令（create 的意思，其中 c/x 仅能存在一个！不可同时存在！） |
| -x   | 解开一个压缩文件的参数指令（其中 c/x 仅能存在一个！不可同时存在！） |
| -v   | 压缩的过程中显示文件                                         |
| -f   | 使用档案名字，这个参数是最后一个参数，后面只能接档案名！     |

#### 1）tar -zcvf

打包及压缩（gzip 方式）

```shell
[root@localhost ~]# ll
drwxrwxrwx. 3 root root   17 Mar 29 14:24 abc

[root@localhost ~]# tar -zcvf abc.tar.gz abc
abc/
abc/xyz/

[root@localhost ~]# ll
drwxrwxrwx. 3 root root   17 Mar 29 14:24 abc
-rw-r--r--. 1 root root  130 Mar 29 15:24 abc.tar.gz
```

#### 2）tar -zxvf

解压（gzip包）

```shell
[root@localhost ~]# ll
drwxrwxrwx. 3 root root   17 Mar 29 14:24 abc
-rw-r--r--. 1 root root  130 Mar 29 15:24 abc.tar.gz

[root@localhost ~]# mkdir test
[root@localhost ~]# cd test/
[root@localhost test]# mv ../abc.tar.gz  .
[root@localhost test]# ll
total 4
-rw-r--r--. 1 root root 130 Mar 29 15:24 abc.tar.gz

[root@localhost test]# tar -zxvf abc.tar.gz 
abc/
abc/xyz/

[root@localhost test]# ll
total 4
drwxrwxrwx. 3 root root  17 Mar 29 14:24 abc
-rw-r--r--. 1 root root 130 Mar 29 15:24 abc.tar.gz
```

### 2.2. 基本命令 - 输出及显示

#### 1）echo

不解析转义字符，直接将内容输出到设备

```shell
[root@localhost ~]# echo "hello\t\t world！"
hello\t\t world！
```

#### 2）echo -e

解析转义字符，再将解析后的内容输出到设备

```shell
[root@localhost ~]# echo -e "hello\t\t world！"
hello            world！
```

#### 3）echo $PATH

输出环境变量，等同于 `echo ${PATH} `

```shell
[root@localhost ~]# echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin

[root@localhost ~]# echo ${PATH}
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin
```

#### 4）1 & 2

1 表示标准输出，2 表示标准错误输出

```shell
# 标准输出
[root@bigdata01 ~]# ll
总用量 59724
-r--------. 1 root root 61084192 8月  12 20:43 access_2020_04_30.log
# 标准错误输出
[root@bigdata01 ~]# lk
-bash: lk: 未找到命令

# 省略时，默认认为是1，表示标准输出
[root@bigdata01 ~]# ll >b.txt
[root@bigdata01 ~]# cat b.txt 
总用量 59728
-r--------. 1 root root 61084192 8月  12 20:43 access_2020_04_30.log

# 标准错误输出，用1无效，必须用2
[root@bigdata01 ~]# lk 1>c.txt
-bash: lk: 未找到命令
[root@bigdata01 ~]# cat c.txt
[root@bigdata01 ~]# lk 2>c.txt
[root@bigdata01 ~]# cat c.txt
-bash: lk: 未找到命令
```

#### 5）> & >>

`>` 表示输出重定向并覆盖原始内容，`>>` 表示输出重定向并追加内容到末尾

```shell
# 输出重定向并覆盖原始内容
[root@bigdata01 ~]# rm -f a.txt
[root@bigdata01 ~]# cat a.txt 
cat: a.txt: 没有那个文件或目录
[root@bigdata01 ~]# ll 1>a.txt
[root@bigdata01 ~]# ll 1>a.txt
[root@bigdata01 ~]# cat a.txt 
总用量 59724
-r--------. 1 root root 61084192 8月  12 20:43 access_2020_04_30.log

# 输出重定向并追加内容到末尾
[root@bigdata01 ~]# rm -f a.txt
[root@bigdata01 ~]# cat a.txt 
cat: a.txt: 没有那个文件或目录
[root@bigdata01 ~]# ll 1>>a.txt
[root@bigdata01 ~]# ll 1>>a.txt
[root@bigdata01 ~]# cat a.txt 
总用量 59724
-r--------. 1 root root 61084192 8月  12 20:43 access_2020_04_30.log
总用量 59728
-r--------. 1 root root 61084192 8月  12 20:43 access_2020_04_30.log
```

#### 6）&

&，表示当前 shell 进程后台执行，但 shell 窗口关闭后，该进程也会被杀死

```shell
# 前台执行，前台打印
[root@bigdata01 ~]# cat while1.sh 
#!/bin/bash
while test 2 -gt 1
do
echo yes
sleep 1
done
[root@bigdata01 ~]# sh while1.sh 
yes
yes
^C

# 后台执行，前台打印
[root@bigdata01 ~]# sh while1.sh &
[1] 2227
[root@bigdata01 ~]# yes
yes
yes
^C
```

#### 7）nohup

指定 nohup 后，当前进程在当前 shell 窗口关闭后，不会收到任何杀死信号，所以也不会被停止

```shell
# 后台执行，后台打印
[root@bigdata01 ~]# nohup sh while1.sh &
[1] 2375
[root@bigdata01 ~]# nohup: 忽略输入并把输出追加到"nohup.out"
^C
# 后台打印位置
[root@bigdata01 ~]# tail -f nohup.out 
yes
yes
^C
```

#### 8）/dev/null

`/dev/null`，一个特殊路径，表示 linux 中的一个黑洞，任何数据丢进去以后，就再也找不到了

```shell
[root@bigdata01 ~]# rm -f nohup.out 
# 后台执行，后台打印，并把所有标准输出1(已省略)，都重定向到黑洞中，所有标准错误输出，都取标准输出1的地址，然后重定向到标准输出nohup.out中，这里必须用>&1，也有追加的效果
[root@bigdata01 ~]# nohup sh while1.sh >/dev/null 2>&1 &
[2] 2731
# 无标准错误输出
[root@bigdata01 ~]# tail -f nohup.out
tail: 无法打开"nohup.out" 读取数据: 没有那个文件或目录
tail: 没有剩余文件
```

### 2.3. 基本命令 - yum 安装及卸载

1. yum，在线安装，它集成了连接网络，软件安装、删除、更新等功能。
2. yum 在配置好 repo 后，机器只要连网，就能智能化安装软件，使用 yum 安装的好处在于，可以自动安装软件需要的依赖包。

#### 1）yum install

安装，追加 -y，则可以免提示安装

#### 2）yum update

升级

#### 3）yum info

显示包信息

#### 4）yum list

显示已安装或可安装包

#### 5）yum remove

删除程序

#### 6）yum clean all

清除所有缓存（包含文件、旧软件）

### 2.4. 基本命令 - 查看磁盘

#### 1）df -h

以人类易于识别的方式，显示硬盘的使用情况

```shell
[root@localhost ~]# df -h
Filesystem               Size  Used Avail Use% Mounted on
devtmpfs                 898M     0  898M   0% /dev
tmpfs                    910M     0  910M   0% /dev/shm
tmpfs                    910M  9.5M  901M   2% /run
tmpfs                    910M     0  910M   0% /sys/fs/cgroup
/dev/mapper/centos-root   17G  1.2G   16G   7% /
/dev/sda1               1014M  150M  865M  15% /boot
tmpfs                    182M     0  182M   0% /run/user/0
```

### 2.5. 基本命令 - 查看内存

#### 1）free -m

查看内存和交换空间的使用情况，-m 显示内存单位为 MB

```shell
[root@localhost ~]# free -m
              total        used        free      shared  buff/cache   available
Mem:           1819         169        1509           9         140        1501
Swap:          2047           0        2047
```

#### 2）free -h

查看内存和交换空间的使用情况，-h 根据值的大小，显示人类易于识别的单位

```shell
[root@localhost ~]# free -h
              total        used        free      shared  buff/cache   available
Mem:           1.8G        169M        1.5G        9.5M        140M        1.5G
Swap:          2.0G          0B        2.0G
```

### 2.6. 基本命令 - 查看操作历史

#### 1）history

留了最近执行的命令记录，默认保留 1000，其中历史清单从 0 开始编号到最大值。

#### 2）history N

显示最近N条命令

```shell
[root@localhost ~]# history 10
  142  ll
  143  tar -zxvf abc.tar.gz 
  144  ll
  145  cd .
  146  ll
  147  echo "hello\t\t world！"
  148  echo -e "hello\t\t world！"
  149  env
  150  echo $PATH
  151  history 10
```

#### 3）history -c

清除所有的历史记录

#### 4）history -w xxx.txt

保存历史记录到文本 xxx.txt

```shell
[root@localhost ~]# history  -w his.txt

[root@localhost ~]# ll
-rw-------. 1 root root 1395 Mar 29 15:41 his.txt
[root@localhost ~]# more his.txt 
ip addr
ls /
pwd
ls
ll
..........
```

### 2.7. 基本命令 - 清屏

#### 1）clear

清屏，类似于widows 中的 cls 命令

```shell
[root@localhost ~]# clear
```

### 2.8. 基本命令 - 防火墙

#### 1）查看防火墙状态

```shell
[root@bigdata01 ~]# systemctl status firewalld
● firewalld.service - firewalld - dynamic firewall daemon
   Loaded: loaded (/usr/lib/systemd/system/firewalld.service; disabled; vendor preset: enabled)
   Active: active (running) since 六 2022-08-13 20:08:42 CST; 8s ago
     Docs: man:firewalld(1)
 Main PID: 3502 (firewalld)
   CGroup: /system.slice/firewalld.service
           └─3502 /usr/bin/python2 -Es /usr/sbin/firewalld --nofork --nopid

8月 13 20:08:42 bigdata01 systemd[1]: Starting firewalld - dynamic firewall daemon...
8月 13 20:08:42 bigdata01 systemd[1]: Started firewalld - dynamic firewall daemon.
8月 13 20:08:43 bigdata01 firewalld[3502]: WARNING: AllowZoneDrifting is enabled. This is considered an insecure configuration option. It will be removed in a future release. Please consid...abling it now.
Hint: Some lines were ellipsized, use -l to show in full.

```

#### 2）临时打开防火墙

```shell
[root@bigdata01 ~]# systemctl start firewalld
```

#### 3）永久打开防火墙

```shell
[root@bigdata01 ~]# systemctl enable firewalld
Created symlink from /etc/systemd/system/dbus-org.fedoraproject.FirewallD1.service to /usr/lib/systemd/system/firewalld.service.
Created symlink from /etc/systemd/system/multi-user.target.wants/firewalld.service to /usr/lib/systemd/system/firewalld.service.
```

#### 4）临时关闭防火墙

```shell
[root@bigdata01 ~]# systemctl stop firewalld
```

#### 5）永久关闭防火墙

```shell
[root@bigdata01 ~]# systemctl disable firewalld
Removed symlink /etc/systemd/system/multi-user.target.wants/firewalld.service.
Removed symlink /etc/systemd/system/dbus-org.fedoraproject.FirewallD1.service.
```

### 2.9. 基本命令 - 关机/重启

#### 1）shutdown [-h] now

立即关机

#### 2）shutdown -r now

立即重启

#### 3）reboot [-h] now

立即重启，同 shutdown -r now

#### 4）exit

退出当前用户的登录状态

### 3.0 高级命令 - 文本内容编辑 vi

#### 1）命令行模式 + i

进入编辑模式，此时可以编辑文件内容，+ i 的意思表示，进入命令行模式后，按下 i

```shell
[root@localhost ~]# cp anaconda-ks.cfg anaconda-ks.cfg_bak
#version=DEVEL
# System authorization information
```

#### 2）编辑模式 + esc

退出编辑模式

#### 3）命令行模式 + wq

保存（write）、退出（quit）

#### 4）命令行模式 + q

不保存，直接退出

#### 5）命令行模式 + q!

不保存，强制退出

#### 6）命令行模式+ /hello

查找 hello 字符串，再按下 n 可以循环查找

#### 7）命令行模式 + :10

跳到文本第 10 行

#### 8）命令行模式 + :set nu

带行号显示文本

#### 9）命令行模式 + yy + p

快速复制 & 粘贴某一行内容

#### 10）命令行模式 + [目标行号] + dd

快速删除某一行，如果指定目标行号，则删除当前行~目标行之间的所有行。

#### 11）命令行模式 + G

快速跳到文本最后一行

#### 12）命令行模式 + gg

快速跳到文本第一行

#### 13）命令行模式 + $

快速定位到当前行的末尾

#### 14）临时文件 .swp

1. vi + recover，或者 vi -r 恢复上次未保存的内容。
2. vi + 回车，丢弃上次未保存的内容。
3. 但无论怎样，wq 保存退出后，下次再次进入时并未自动清除，此时恢复/丢弃后，删除同层级目录下，隐藏的 swp 临时文件即可。

```shell
E325: ATTENTION
Found a swap file by the name ".helloworld.txt.swp"
          owned by: root   dated: Tue Aug  9 21:07:29 2022
         file name: ~root/helloworld.txt
          modified: YES
         user name: root   host name: localhost.localdomain
        process ID: 1947
While opening file "helloworld.txt"
             dated: Tue Aug  9 21:05:59 2022

(1) Another program may be editing the same file.  If this is the case,
    be careful not to end up with two different instances of the same
    file when making changes.  Quit, or continue with caution.
(2) An edit session for this file crashed.
    If this is the case, use ":recover" or "vim -r helloworld.txt"
    to recover the changes (see ":help recovery").
    If you did this already, delete the swap file ".helloworld.txt.swp"
    to avoid this message.
```

### 3.1. 高级命令 - 文件内容统计 wc

#### 1）wc -c file

统计文件的字节大小

```shell
[root@localhost ~]# wc -c hello.txt 
13 hello.txt

[root@localhost ~]# cat hello.txt 
hello world!
```

#### 2）wc -m file

统计文件的字符个数，包括空格符、换行符等

```shell
[root@localhost ~]# cat hello.txt 
hello world!

[root@localhost ~]# wc -m hello.txt 
13 hello.txt
```

#### 3）wc -l file

统计文件的行数

```shell
[root@localhost ~]# wc -l hello.txt 
1 hello.txt

[root@localhost ~]# cat hello.txt 
hello world!
```

#### 4）wc -L file

统计最长一行的文本长度，包括空格，但不包括换行符

```shell
[root@localhost ~]# cat hello.txt 
hello world!

[root@localhost ~]# wc -L hello.txt 
12 hello.txt
```

#### 5）wc -w file

统计文件中单词的个数（用空格隔开，且支持中文）

```shell
[root@localhost ~]# wc -w hello.txt 
5 hello.txt

[root@localhost ~]# cat hello.txt 
hello world! 中国 人 民
```

### 3.2. 高级命令 - 文件内容排序 sort

#### 1）sort file

按每个字符的自然顺序排序

```shell
[root@localhost ~]# sort num.txt 
1
10
2
3
9

[root@localhost ~]# cat num.txt 
3
2
9
10
1
```

#### 2）sort -n file

按文本的数值大小排序

```shell
[root@localhost ~]# cat num.txt 
3
2
9
10
1

[root@localhost ~]# sort -n num.txt 
1
2
3
9
10
```

#### 3）sort -nr file

按文本的数值大小排序，再倒序

```shell
[root@localhost ~]# sort -nr num.txt 
10
9
3
2
1

[root@localhost ~]# cat num.txt 
3
2
9
10
1
```

#### 4）sort -k key -n file

指定第 2 列（从 1 开始）作为 key，然后再按文本的数值大小排序

```shell
[root@localhost ~]# cat num2.txt 
bc 2
ax 1
aa 9
dd 7
xc 15

[root@localhost ~]# sort -k 2 -n num2.txt 
ax 1
bc 2
dd 7
aa 9
xc 15
```

### 3.3. 高级命令 - 文件内容去重 uniq

#### 1）uniq file

去重文本内容，但需要有序

```shell
[root@localhost ~]# uniq hello.txt 
hello world!
[root@localhost ~]# cat hello.txt 
hello world!
hello world!
hello world!
hello world!
hello world!
hello world!

# 乱序文件去重
[root@localhost ~]# uniq test.txt 
hello
abc
hello
# 乱序达不到去重效果
[root@localhost ~]# cat test.txt 
hello
hello
abc
hello
hello
# 先排序
[root@localhost ~]# sort test.txt
abc
hello
hello
hello
hello
# 再去重
[root@localhost ~]# sort test.txt | uniq 
abc
hello
```

#### 2）uniq -c file

去重文本内容，并显示重复的行数，但需要有序

```shell
[root@localhost ~]# cat hello.txt 
hello world!
hello world!
hello world!
hello world!
hello world!
hello world!

[root@localhost ~]# uniq -c hello.txt 
      6 hello world!
```

#### 3）uniq -u file

返回内容不存在重复的行，但需要有序

```shell
[root@localhost ~]# uniq -u hello.txt 
abc

[root@localhost ~]# cat hello.txt 
hello world!
hello world!
hello world!
hello world!
hello world!
hello world!
abc
```

### 3.4. 高级命令 - 文本内容查看

#### 1）head -n file

查看文本开头前 n 条数据，默认查询全部

```shell
[root@localhost ~]# cat num.txt 
3
2
9
10
1

[root@localhost ~]# head num.txt 
3
2
9
10
1

[root@localhost ~]# head -3 num.txt 
3
2
9

# top3实现
[root@localhost ~]# sort -nr num.txt | head -3
10
9
3
```

#### 2）tail -n file

查看文本末尾 n 条数据，默认查询全部

```shell
[root@localhost ~]# cat num.txt 
3
2
9
10
1

[root@localhost ~]# tail num.txt 
3
2
9
10
1

[root@localhost ~]# tail -3 num.txt 
9
10
1
```

### 3.5. 高级命令 - 日期函数 date

#### 1）date

获取当前系统当前时间，按默认格式显示

```shell
[root@localhost ~]# date
2022年 08月 12日 星期五 18:41:48 CST
```

#### 2）date +"pattern"

获取当前系统当前时间，按指定的 pattern 格式显示，其中，pattern 连续时，双引号可以去掉

```shell
# 获取常用的【年-月-日 时：分：秒】格式
[root@localhost ~]# date +"%Y-%m-%d %H:%M:%S"
2022-08-12 18:41:50

# 获取当前时间戳：可以用来比较日期
[root@localhost ~]# date +%s
1660301109

# 获取毫秒数：时间戳+3个0
[root@localhost ~]# date +%s000
1660301227000
```

#### 3）date --date="str"

解析日期字符串，str 连续时，双引号可以去掉

```shell
[root@localhost ~]# date --date="2026-01-01 00:00:00"
2026年 01月 01日 星期四 00:00:00 CST
[root@localhost ~]# date --date=2026-01-01
2026年 01月 01日 星期四 00:00:00 CST

# 解析后，再格式化为时间戳
[root@localhost ~]# date --date="2026-01-01 00:00:00" +%s
1767196800

# 解析昨天日期，再格式化显示
[root@localhost ~]# date --date="1 days ago" +%Y-%m-%d
2022-08-11
[root@localhost ~]# date --date="1 days ago"
2022年 08月 11日 星期四 18:52:56 CST
[root@localhost ~]# date +%Y-%m-%d
2022-08-12

# 获取2026年2月份一共有多少天：解析2.28号的日期即可
[root@localhost ~]#  date --date="2026-03-01 1 days ago" +%d
28
```

### 3.6. 高级命令 - 进程相关函数

#### 1）ps -ef

显示系统中所有的进程信息

- ps：process status

```shell
[root@localhost ~]# ps -ef
UID        PID  PPID  C STIME TTY          TIME CMD
root         1     0  0 16:55 ?        00:00:00 /usr/lib/systemd/systemd --switched-root --system --deserialize 22
root         2     0  0 16:55 ?        00:00:00 [kthreadd]
...

# 只显示系统中包含 python 关键字的进程信息
[root@localhost ~]# ps -ef | grep python
root       670     1  0 16:56 ?        00:00:00 /usr/bin/python2 -Es /usr/sbin/firewalld --nofork --nopid
root      1048     1  0 16:56 ?        00:00:00 /usr/bin/python2 -Es /usr/sbin/tuned -l -P
# 这个是grep进程本身，grep：Global Regular Expression Print，全局正则表达式匹配
root     13215  1560  0 19:08 pts/0    00:00:00 grep --color=auto python
```

#### 2）netstat -anp

显示网络进程相关信息，包含端口信息

```shell
[root@localhost ~]#  netstat -anp
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      1044/sshd           
tcp        0      0 127.0.0.1:25            0.0.0.0:*               LISTEN      1378/master         
tcp        0     36 192.168.56.106:22       192.168.56.1:51098      ESTABLISHED 1556/sshd: root@pts 
...

# 只显示系统中包含 22 关键字（端口）的网络进程信息
[root@localhost ~]#  netstat -anp | grep 22
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      1044/sshd           
tcp        0     36 192.168.56.106:22       192.168.56.1:51098      ESTABLISHED 1556/sshd: root@pts 
tcp        0      0 192.168.56.106:22       192.168.56.1:51099      ESTABLISHED 1575/sshd: root@not 
tcp6       0      0 :::22                   :::*                    LISTEN      1044/sshd           
unix  3      [ ]         STREAM     CONNECTED     18322    1378/master          
unix  2      [ ]         DGRAM                    16122    670/python2 
```

#### 3）jps

只显示当前用户启动的 java 进程信息，这个是 java 工具，需要在 java 环境下运行

#### 4）top

动态显示系统进程资源消耗情况，包括内存、CPU 占用等，可通过键入 0~n 展示不同 CPU 的使用情况

```shell
# 抬头信息：当前时间、系统已运行时间、当前登录用户数、任务队列长度（系统负载）：1、5、15分钟平均值
top - 19:22:28 up  2:26,  2 users,  load average: 0.00, 0.01, 0.05
# 进程信息：总数、运行数、睡眠数、停止数、僵尸进程数
Tasks: 101 total,   1 running, 100 sleeping,   0 stopped,   0 zombie
# CPU信息：用户空间%、内核空间%、优先级改变%、空闲%、等待I/O%、硬中断%、软中断%、虚拟机占用%
%Cpu(s):  0.0 us,  0.0 sy,  0.0 ni, 99.7 id,  0.3 wa,  0.0 hi,  0.0 si,  0.0 st
# 内存信息：总量、使用量、空闲量、内核缓存量
KiB Mem :  1882048 total,  1557732 free,   165564 used,   158752 buff/cache
# 交换信息：总量、使用量、空闲量、内核->交换区的缓存量
KiB Swap:  2097148 total,  2097148 free,        0 used.  1565992 avail Mem 
# 表格信息：进程ID、进程所有者、优先级、nice优先级、虚拟内存总量、物理内存大小、共享内存大小、进程状态(D=不可中断的睡眠状态,R=运行,S=睡眠,T=跟踪/停止,Z=僵尸进程)、间隔CPU%、内存%、时间统计、命令名
  PID USER      PR  NI    VIRT    RES    SHR S %CPU %MEM     TIME+ COMMAND             
    1 root      20   0  128004   6624   4160 S  0.0  0.4   0:00.78 systemd             
    2 root      20   0       0      0      0 S  0.0  0.0   0:00.00 kthreadd  s
```

#### 5）kill PID

杀掉指定的 PID 进程，由于进程是主动杀死，所以算自杀，程序最后可以做一些收尾操作

#### 6）kill -9 PID

强制杀掉指定的 PID 进程，由于进程是被动杀死，所以算他杀，程序最后可能来不及做收尾操作

### 3.7. 高级命令 - Linux 三剑客 - 查找 grep

#### 1）grep pattern file

以 pattern 正则表达式去匹配 file 中的文本内容

- grep：Global Regular Expression Print，全局正则表达式匹配

```shell
[root@localhost ~]# cat hello.txt 
hello world!
hello world!
hello world!
hello world!
hello world!
hello world!
abc

# 是否包含abc
[root@localhost ~]# grep abc hello.txt 
abc
# grep 也可以与 | 管道搭配使用，效果与上面相等
[root@localhost ~]# cat hello.txt | grep abc
abc
# 正则表达式：以a开头
[root@localhost ~]# grep ^a hello.txt 
abc
```

#### 2）grep -n pattern file

匹配后还展示行号

```shell
[root@localhost ~]# grep -n abc hello.txt 
7:abc

[root@localhost ~]# cat hello.txt 
hello world!
hello world!
hello world!
hello world!
hello world!
hello world!
abc
```

#### 3）grep -i pattern file

忽略 pattern 大小写匹配

```shell
[root@localhost ~]# cat hello.txt 
hello world!
hello world!
hello world!
hello world!
hello world!
hello world!
abc

[root@localhost ~]# grep ABC hello.txt 
[root@localhost ~]# grep -i ABC hello.txt 
abc
```

#### 4）grep -v pattern file

忽略 -v 指定的内容，即再过滤一次

```shell
[root@localhost ~]# ps -ef | grep python
root       670     1  0 16:56 ?        00:00:00 /usr/bin/python2 -Es /usr/sbin/firewalld --nofork --nopid
root      1048     1  0 16:56 ?        00:00:01 /usr/bin/python2 -Es /usr/sbin/tuned -l -P
root     13489  1560  0 19:59 pts/0    00:00:00 grep --color=auto python

[root@localhost ~]# ps -ef | grep python | grep -v grep
root       670     1  0 16:56 ?        00:00:00 /usr/bin/python2 -Es /usr/sbin/firewalld --nofork --nopid
root      1048     1  0 16:56 ?        00:00:01 /usr/bin/python2 -Es /usr/sbin/tuned -l -P
```

### 3.8. 高级命令 - Linux 三剑客 - 编辑 sed

sed ”pattern“ file..，sed，stream editor，可以用来同时修改单个、或者多个文件，只会编辑缓冲区的内容，原文件不变

#### 1）添加内容

i 表示插入，a 表示追加

```shell
[root@localhost ~]# cat hello.txt 
hello world!
hello world!
hello world!
hello world!
hello world!
hello world!
abc

# 在第1行（从1开始）之前，插入（insert）一行内容
[root@localhost ~]# sed '1i\haha' hello.txt 
haha
hello world!
hello world!
hello world!
hello world!
hello world!
hello world!
abc

# 在最后一行（$表示）之后，添加（add）一行内容
[root@localhost ~]# sed '$a\haha' hello.txt 
hello world!
hello world!
hello world!
hello world!
hello world!
hello world!
abc
haha
```

#### 2）删除内容

d 表示删除

```shell
[root@localhost ~]# cat -n hello.txt 
     1  hello world!
     2  hello world!
     3  hello world!
     4  hello world!
     5  hello world!
     6  hello world!
     7  abc

# 删除第7行内容
[root@localhost ~]# sed "7d" hello.txt 
hello world!
hello world!
hello world!
hello world!
hello world!
hello world!
```

#### 3）替换内容

s 表示替换

> ```
> sed [address]s/pattern/replacement/flags
> address：表示指定要操作的具体行，是一个可选项
> s：表示替换操作
> pattern：指的是需要替换的内容
> replacement：指的是要替换的新内容
> flags：有多种用法
>    第一种：就是flags可以表示为1~512之间的任意一个数字，表示指定要替换的字符串在这一行中出现第几次时才进行替换
>    第二种：就是flags可以直接表示为g，这样的意识就是对每一行数据中所有匹配到的内容全部进行替换
>    第三种：如果flags位置的值为空，则只会在第一次匹配成功时做替换操作
> ```

```shell
[root@localhost ~]# cat -n hello.txt 
     1  hello world!
     2  hello world!
     3  hello world!
     4  hello world!
     5  hello world!
     6  hello world!
     7  abc

# 整个文本全量替换l为a
[root@localhost ~]# sed "s/l/a/g" hello.txt 
heaao worad!
heaao worad!
heaao worad!
heaao worad!
heaao worad!
heaao worad!
abc
# 全量替换第3次出现的l为a
[root@localhost ~]# sed "s/l/a/3" hello.txt 
hello worad!
hello worad!
hello worad!
hello worad!
hello worad!
hello worad!
abc
# 默认整个文本全量替换l为a
[root@localhost ~]# sed "s/l/a/" hello.txt 
healo world!
healo world!
healo world!
healo world!
healo world!
healo world!
abc
# 只替换第2行内容中所有的l为a
[root@localhost ~]# sed "2s/l/a/g" hello.txt 
hello world!
heaao worad!
hello world!
hello world!
hello world!
hello world!
abc
```

#### 4）修改内容并覆盖

-i 参数，可以把缓冲区修改后的结果，覆盖回原文件中

```shell
[root@localhost ~]# cat -n hello.txt 
     1  hello world!
     2  hello world!
     3  hello world!
     4  hello world!
     5  hello world!
     6  hello world!
     7  abc
     
[root@localhost ~]# sed -i "2s/l/a/g" hello.txt 
[root@localhost ~]# cat hello.txt 
hello world!
heaao worad!
hello world!
hello world!
hello world!
hello world!
abc
```

### 3.9. 高级命令 - Linux 三剑客 - 分析 awk

awk，命令于创始人，Alfred Aho 、Peter Weinberger 和 Brian Kernighan，是一个强大的文本分析工具。

#### 1）awk [option] program file

awk 会根据分割符，把每行文本切割开来，并从 1 开始，按顺序为每个单词设置变量

- option 表示可选参数。
- program 表示要操作的命令，包括正则表达式。

```shell
[root@localhost ~]# cat hello.txt 
hello world!
heaao worad!
hello world!
hello world!
hello world!
hello world!
abc

# 输出每行的第1个单词，多列输出用逗号隔开 => {print $1,$2}
[root@localhost ~]# awk '{print $1}' hello.txt 
hello
heaao
hello
hello
hello
hello
abc
# 输出每行的第2个单词
[root@localhost ~]# awk '{print $2}' hello.txt 
world!
worad!
world!
world!
world!
world!

# 输出每行的全部内容，$0表示整行
[root@localhost ~]# awk '{print $0}' hello.txt 
hello world!
heaao worad!
hello world!
hello world!
hello world!
hello world!
abc
# 整行输出包含world关键字的内容
[root@localhost ~]# awk '/world/{print $0}' hello.txt 
hello world!
hello world!
hello world!
hello world!
hello world!
# 整行输出第1列(从1开始)包含world关键字的内容
[root@localhost ~]# awk '($1 ~/world/){print $0}' hello.txt
# 整行输出第2列(从1开始)包含world关键字的内容
[root@localhost ~]# awk '($2 ~/world/){print $0}' hello.txt 
hello world!
hello world!
hello world!
hello world!
hello world!
# 整行输出第2列(从1开始)不包含world关键字的内容
[root@localhost ~]# awk '($2 !~/world/){print $0}' hello.txt 
heaao worad!
abc
```

#### 2）awk -F program file

-F 用来设定分割符，默认表示空格符

```shell
[root@localhost ~]# cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
...

# 没指定:分割符 => 分割失败
[root@localhost ~]# awk '{print $1}' /etc/passwd
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
...

# 指定了:分割符 => 分割成功
[root@localhost ~]# awk -F: '{print $1}' /etc/passwd
root
bin
```

### 4.0. 高级命令 - 字符串截取

#### 1）`#` 从左匹配，删除左边字符

```shell
[root@bigdata01 ~]# var=http://www.baidu.com/test.htm
[root@bigdata01 ~]# echo ${var}
http://www.baidu.com/test.htm

[root@bigdata01 ~]# echo ${var#*/}
/www.baidu.com/test.htm
```

#### 2）`##` 从右匹配，删除左边字符

```shell
[root@bigdata01 ~]# echo ${var##*/}
test.htm
```

#### 3）`%` 从右匹配，删除右边字符

```shell
[root@bigdata01 ~]# echo ${var%/*}
http://www.baidu.com
```

#### 4）`%%` 从左匹配，删除右边字符

```shell
[root@bigdata01 ~]# echo ${var%%/*}
http:
```

#### 5）从左 i 开始，截取 n 个字符

```shell
[root@bigdata01 ~]# echo ${var:0:5}
http:
```

#### 6）从左 i 开启，截取到末尾

```shell
[root@bigdata01 ~]# echo ${var:5}
//www.baidu.com/test.htm
```

#### 7）从右 i 开始，截取 n 个字符

```shell
# 向右从-1开始数起
[root@bigdata01 ~]# echo ${var:0-8:4}
test
```

#### 8）从右 i 开始，截取到末尾

```shell
# 向右从-1开始数起
[root@bigdata01 ~]# echo ${var:0-8}
test.htm
```

### 4.1. 高级命令 - 字符串替换

#### 1）替换第 1 个

```shell
[root@bigdata01 shell]# str=2022_08_13
[root@bigdata01 shell]# echo ${str/_/}
202208_13
```

#### 2）全量替换

```shell
[root@bigdata01 shell]# str=2022_08_13
[root@bigdata01 shell]# echo ${str//_/}
20220813
```

### 4.2. shell 脚本

shell，是用户与 Linux 沟通的桥梁，把一堆 shell 命令编程到一个文件中，可以构成一个 shell 脚本，从而实现批量执行命令。

- shell 脚本，通常约定以 `.sh` 结尾。
- shell 脚本规定，第一行内容为 `#!bin/bash`，表示把 shell 环境导入进来。

#### 1）执行方式

- bash、sh 执行：把文本内容作为参数，传入 `/bin/bash` 函数中执行。
- ./、/ 执行：需要执行 +x 执行权限。
- 直接执行：需要指定 `.:$PATH` 环境变量。

```shell
[root@bigdata01 ~]# cat hello.sh 
#!/bin/bash
# first command
echo hello world!

# bash、sh 执行
[root@bigdata01 ~]# sh hello.sh 
hello world!
# sh等同于bash
[root@bigdata01 ~]# bash hello.sh 
hello world!

# ./、/ 执行
[root@bigdata01 ~]# chmod u+x hello.sh 
[root@bigdata01 ~]# ll
总用量 59680
-r--------. 1 root root 61084192 8月  12 20:43 access_2020_04_30.log
-rw-------. 1 root root     1244 8月   9 17:19 anaconda-ks.cfg
-rwxr--r--. 1 root root       46 8月  12 22:36 hello.sh
-rw-r--r--. 1 root root       82 8月  12 20:20 hello.txt
-rw-r--r--. 1 root root       26 8月  12 17:28 num2.txt
-rw-r--r--. 1 root root       11 8月  12 17:23 num.txt
-rw-r--r--. 1 root root       28 8月  12 18:26 test.txt
[root@bigdata01 ~]# ./hello.sh 
hello world!

# 直接执行
[root@bigdata01 ~]# tail -1 /etc/profile
export PATH=.:$PATH
[root@bigdata01 ~]# echo $PATH
.:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin
[root@bigdata01 ~]# hello.sh 
hello world!
```

#### 2）变量赋值

- shell 中的变量不需要声明，初始化时也不需要指定类型。
- shell 变量命名只能使用数字、字母和下划线，不能以数字开头。
- shell 变量只能通过 `=` 赋值，且赋值时变量、等号和值之间，不能出现空格！

```shell
# 赋值时不能有空格
[root@bigdata01 ~]# name = zs
-bash: name: 未找到命令
[root@bigdata01 ~]# name =zs
-bash: name: 未找到命令

# 正常赋值
[root@bigdata01 ~]# name=zs
[root@bigdata01 ~]# name2_=zs

# 变量命名不能以数字开头
[root@bigdata01 ~]# 2name=zs
-bash: 2name=zs: 未找到命令

# 变量命名不能包含特殊字符
[root@bigdata01 ~]# name$=zs
-bash: name$=zs: 未找到命令

# 正式使用变量, $name是${name}的简化写法
[root@bigdata01 ~]# echo $name
zs
[root@bigdata01 ~]# echo ${name}
zs

# $name不能连续拼接，但${name}可以
[root@bigdata01 ~]# echo $namehehe
[root@bigdata01 ~]# echo ${name}hehe
zshehe
```

#### 3）变量作用域

| 变量格式                             | 作用域                                                       |
| ------------------------------------ | ------------------------------------------------------------ |
| var_name=value                       | 临时本地变量，只对当前 shell 进程有效，对子 shell 进程以及其他 shell 进程无效 |
| export var_nam=value                 | 临时环境变量，对子 shell 进程有效，但对其他 shell 进程无效   |
| /etc/profile => export var_nam=value | 永久环境变量，每次打开一个 shell 进程，linux 都会读取该配置  |
| xxx.sh abc xyz                       | 位置变量，shell 脚本中通过 `$0、$1` 来动态获取，相当于 java#main 中的 args 参数 |
| $?                                   | 特殊变量，表示上一条命令返回的状态码，执行成功为0，失败则在1~255 之间 |
| $#                                   | 特殊变量，表示传入 shell 脚本的参数个数                      |

##### 1、本地临时变量

```shell
# 赋值临时本地变量
[root@bigdata01 ~]# name=zs
# 正常使用变量
[root@bigdata01 ~]# echo $name
zs
# 查看当前进程树
[root@bigdata01 ~]# pstree
systemd─┬─NetworkManager─┬─2*[dhclient]
        ├─sshd─┬─sshd─┬─bash───pstree
        │      │      └─bash
        ...
# 切换子shell进程
[root@bigdata01 ~]# bash
# 查看切换后的进程树
[root@bigdata01 ~]# pstree
systemd─┬─NetworkManager─┬─2*[dhclient]
        ├─sshd─┬─sshd─┬─bash───bash───pstree
        │      │      └─bash
        │      └─sshd───6*[sftp-server]
		...
# 无法使用name变量
[root@bigdata01 ~]# echo $name
```

##### 2、临时环境变量

```shell
# 赋值临时环境变量
[root@bigdata01 ~]# export age=18
# 正常使用变量
[root@bigdata01 ~]# echo $age
18
# 切换到子shell进程
[root@bigdata01 ~]# bash
# 正常使用变量
[root@bigdata01 ~]# echo $age
18
# 退出子shell进程
[root@bigdata01 ~]# exit
exit
```

##### 3、永久环境变量

```shell
# 赋值永久环境变量
[root@bigdata01 ~]# tail -1 /etc/profile
export age=19
# 直接使用，发现没生效
[root@bigdata01 ~]# echo $age
18
# 立刻生效环境变量
[root@bigdata01 ~]# source /etc/profile
# 正常使用变量
[root@bigdata01 ~]# echo $age
19
```

##### 4、位置变量

```shell
# 预先使用位置变量
[root@bigdata01 ~]# cat location.sh 
#!/bin/bash
echo $0
echo $1
echo $2
echo $3
echo $4
echo $5
# 传入位置变量，用空格隔开 => 只打印前2个位置变量，后面4个则为空
[root@bigdata01 ~]# sh location.sh abc xyz
location.sh
abc
xyz





```

##### 5、特殊变量 $?

```shell
# 执行成功，$?返回0
[root@bigdata01 ~]# echo $age
19
[root@bigdata01 ~]# echo $?
0
# 执行失败，$?返回非0
[root@bigdata01 ~]# lk
bash: lk: 未找到命令
[root@bigdata01 ~]# echo $?
127
```

##### 6、特殊变量 $#

```shell
[root@bigdata01 ~]# cat paramnum.sh 
#!/bin/bash
echo $#
[root@bigdata01 ~]# sh paramnum.sh a b c
3
[root@bigdata01 ~]# sh paramnum.sh a b c d e
5
```

#### 4）单引号

shell 中单引号引起的变量，不会被解析

```shell
[root@bigdata01 ~]# echo $age
19
[root@bigdata01 ~]# echo '$age'
$age
```

#### 5）双引号

shell 中双引号引起的变量，可以被解析

```shell
[root@bigdata01 ~]# echo $age
19
[root@bigdata01 ~]# echo "$age"
19
```

#### 6）反引号

shell 中反引号引起的变量，表示执行并引用命令的执行结果，即相当于嵌套使用变量

```shell
[root@bigdata01 ~]# pwd
/root
[root@bigdata01 ~]# mypath=pwd
[root@bigdata01 ~]# echo $mypath
pwd
# 相当于执行 echo $pwd
[root@bigdata01 ~]# echo `$mypath`
/root
# 即相当于嵌套使用变量
[root@bigdata01 ~]# echo $($mypath)
/root
```

#### 7）for 循环

```shell
# 格式1：迭代多次，步长一致
[root@bigdata01 ~]# cat for1.sh 
#!/bin/bash
for((i = 0; i < 10; i++))
do
echo $i
done
[root@bigdata01 ~]# sh for1.sh 
0
1
2
3
4
5
6
7
8
9

# 格式2：元素有限且确定，元素步长无规律
[root@bigdata01 ~]# cat for2.sh 
#!/bin/bash
for i in 1 2 3
do
echo $i
done
[root@bigdata01 ~]# sh for2.sh 
1
2
3
```

#### 8）while 循环

```shell
# 格式1：test + 条件，-gt、-lt、-ge、-le、-eq、-ne、=、!=
[root@bigdata01 ~]# cat while1.sh 
#!/bin/bash
while test 2 -gt 1
do
echo yes
sleep 1
done
[root@bigdata01 ~]# sh while1.sh 
yes
yes
^C
[root@bigdata01 ~]#

# 格式2：[ 条件 ]，-gt、-lt、-ge、-le、-eq、-ne、=、!=
[root@bigdata01 ~]# cat while2.sh 
#!/bin/bash
while [ 2 -gt 1 ]
do
echo yes
sleep 1
done
[root@bigdata01 ~]# sh while2.sh 
yes
yes
^C
[root@bigdata01 ~]# 

# 测试字符串比较，=、!=
[root@bigdata01 ~]# cat str_while2.sh 
#!/bin/bash
while [ "abc" = "abc" ]
do
echo yes
sleep 1
done
[root@bigdata01 ~]# sh str_while2.sh 
yes
yes
^C
[root@bigdata01 ~]# 
```

#### 9）if 判断

##### 1、单分支

```shell
[root@bigdata01 ~]# cat if1.sh 
#!/bin/bash
if [ $# -lt 1 ]
then
    echo "not found param"
    exit 100
fi

flag=$1
if [ $flag -eq 1 ]
then
    echo one
fi
[root@bigdata01 ~]# sh if1.sh 
not found param
[root@bigdata01 ~]# sh if1.sh 1
one
[root@bigdata01 ~]# sh if1.sh 2
[root@bigdata01 ~]# 
```

##### 2、双分支

```shell
[root@bigdata01 ~]# cat if2.sh 
#!/bin/bash
if [ $# -lt 1 ]
then
    echo "not found param"
    exit 100
fi

flag=$1
if [ $flag -eq 1 ]
then
    echo one
else
    echo "not support"
fi
[root@bigdata01 ~]# sh if2.sh 
not found param
[root@bigdata01 ~]# sh if2.sh 1
one
[root@bigdata01 ~]# sh if2.sh 2
not support
```

##### 3、多分支

```shell
[root@bigdata01 ~]# cat if3.sh 
#!/bin/bash
if [ $# -lt 1 ]
then
    echo "not found param"
    exit 100
fi

flag=$1
if [ $flag -eq 1 ]
then
    echo one
elif [ $flag -eq 2 ]
then
    echo two
elif [ $flag -eq 3 ]
then
    echo three
else
    echo "not support"
fi
[root@bigdata01 ~]# sh if3.sh 1
one
[root@bigdata01 ~]# sh if3.sh 2
two
[root@bigdata01 ~]# sh if3.sh 3
three
[root@bigdata01 ~]# sh if3.sh 4
not support
[root@bigdata01 ~]# sh if3.sh
not found param
```

### 4.3. 定时器 crontab

crontab，Linux 中的定时器，可以去周期性地执行命令。

#### 1）查看服务状态

```shell
[root@bigdata01 ~]# systemctl status crond
● crond.service - Command Scheduler
   Loaded: loaded (/usr/lib/systemd/system/crond.service; enabled; vendor preset: enabled)
   Active: active (running) since 六 2022-08-13 11:57:31 CST; 2h 7min ago
 Main PID: 655 (crond)
   CGroup: /system.slice/crond.service
           └─655 /usr/sbin/crond -n

8月 13 11:57:31 bigdata01 systemd[1]: Started Command Scheduler.
8月 13 11:57:32 bigdata01 crond[655]: (CRON) INFO (RANDOM_DELAY will be scaled with factor 31% if used.)
8月 13 11:57:32 bigdata01 crond[655]: (CRON) INFO (running with inotify support)
```

#### 2）停止服务

```shell
[root@bigdata01 ~]# systemctl stop crond
[root@bigdata01 ~]# systemctl status crond
● crond.service - Command Scheduler
   Loaded: loaded (/usr/lib/systemd/system/crond.service; enabled; vendor preset: enabled)
   Active: inactive (dead) since 六 2022-08-13 14:05:53 CST; 3s ago
  Process: 655 ExecStart=/usr/sbin/crond -n $CRONDARGS (code=exited, status=0/SUCCESS)
 Main PID: 655 (code=exited, status=0/SUCCESS)

8月 13 11:57:31 bigdata01 systemd[1]: Started Command Scheduler.
8月 13 11:57:32 bigdata01 crond[655]: (CRON) INFO (RANDOM_DELAY will be scaled with factor 31% if used.)
8月 13 11:57:32 bigdata01 crond[655]: (CRON) INFO (running with inotify support)
8月 13 14:05:53 bigdata01 systemd[1]: Stopping Command Scheduler...
8月 13 14:05:53 bigdata01 systemd[1]: Stopped Command Scheduler.
```

#### 3）启动服务

```shell
[root@bigdata01 ~]# systemctl start crond
[root@bigdata01 ~]# systemctl status crond
● crond.service - Command Scheduler
   Loaded: loaded (/usr/lib/systemd/system/crond.service; enabled; vendor preset: enabled)
   Active: active (running) since 六 2022-08-13 14:06:19 CST; 1s ago
 Main PID: 5516 (crond)
   CGroup: /system.slice/crond.service
           └─5516 /usr/sbin/crond -n

8月 13 14:06:19 bigdata01 systemd[1]: Started Command Scheduler.
8月 13 14:06:19 bigdata01 crond[5516]: (CRON) INFO (RANDOM_DELAY will be scaled with factor 5% if used.)
8月 13 14:06:19 bigdata01 crond[5516]: (CRON) INFO (running with inotify support)
8月 13 14:06:19 bigdata01 crond[5516]: (CRON) INFO (@reboot jobs will be run at computer's startup.)
```

#### 4）crontab 表达式格式

- *号表示，匹配任何一个数字，即每分/每小时/每天/每月/一周内任意一天的意思。
- 可以看到，crontab 表达式最小单位为 min，不支持秒级的任务。

```shell
# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name  command to be executed
```

#### 5）添加定时任务

```shell
# 修改并保存后，任务就会直接加入定时管理中，无需额外命令去开启定时管理
[root@bigdata01 ~]# vi /etc/crontab 
[root@bigdata01 ~]# tail -1 /etc/crontab 
  *  *  *  *  * root       sh /root/showtime.sh >> /root/showTime.log
[root@bigdata01 ~]# tail -f showTime.log
2022-08-13 14:17:01
2022-08-13 14:18:01
^C
```

#### 6）查看 crontab 运行日志

```shell
[root@bigdata01 ~]# tail -f /var/log/cron 
Aug 13 14:01:01 bigdata01 run-parts(/etc/cron.hourly)[4194]: finished 0anacron
Aug 13 14:05:53 bigdata01 crond[655]: (CRON) INFO (Shutting down)
Aug 13 14:06:19 bigdata01 crond[5516]: (CRON) INFO (RANDOM_DELAY will be scaled with factor 5% if used.)
Aug 13 14:06:19 bigdata01 crond[5516]: (CRON) INFO (running with inotify support)
Aug 13 14:06:19 bigdata01 crond[5516]: (CRON) INFO (@reboot jobs will be run at computer's startup.)
Aug 13 14:16:01 bigdata01 crond[5516]: (*system*) RELOAD (/etc/crontab)
Aug 13 14:16:01 bigdata01 CROND[7896]: (root) CMD (      sh /root/showtime.sh )
Aug 13 14:17:01 bigdata01 crond[5516]: (*system*) RELOAD (/etc/crontab)
Aug 13 14:17:01 bigdata01 CROND[8147]: (root) CMD (      sh /root/showtime.sh >> /root/showTime.log)
Aug 13 14:18:01 bigdata01 CROND[8415]: (root) CMD (      sh /root/showtime.sh >> /root/showTime.log)
Aug 13 14:19:02 bigdata01 CROND[8661]: (root) CMD (      sh /root/showtime.sh >> /root/showTime.log)
```

