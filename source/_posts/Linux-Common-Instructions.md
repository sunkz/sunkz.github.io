title: Linux Common Instructions
author: Sunkz
tags:
  - linux
  - shell

photos:

  - https://s2.ax1x.com/2019/11/02/KbcJaD.png

categories: []
date: 2019-08-21 00:54:00

------

/bin  可执行命令文件
/boot 开机所需文件和系统核心文件
/dev 系统设备相关文件
/etc  主要配置文件
/home 除root用户外的其他用户目录
/lib 系统和程序运行的所需的库函数
/root root用户的目录
/sbin  root用户才能执行的命令文件
/srv 各个服务启动后所需要访问的数据 web服务的网页数据/srv/www
/tmp 临时文件
/opt 第三方软件建议安装目录
/media  光驱 u盘

------

磁盘格式化分区 : inode 区  data区  .  
文件的唯一标示inode_number存在inode区, 指向data区的文件 , 
不同分区中会存在相同的inode_number
ll -i 可以查看文件的inode_number
file_name —----n:1-------> inode_number ———1:1---—> file_data

------

UID 每一个用户都有一个UID 
root用户的为0
虚拟用户 bin daemon ftp mail nobody这些用户给系统中的程序使用  UID1-499
普通用户 UID 500 + 

GID 每个用户组都有一个GID  
用户------n:n--------用户组  但一个用户只能有一个主用户组

id user 查看用户信息 
finger user 查看用户信息
su - user 切换用户,且切换shell运行环境
sudo 默认只有root能使用, 可以修改配置文件,是普通用户分配特权指令
sudo -u root xxx  || sudo xxx  普通用户以root身份执行特权命令

/etc/password   /etc/shadow  用户配置文件
/etc/group   /etc/gshadow  用户组配置文件

------

d 目录文件 - 普通文件  c 字符设备文件 b 块设备文件 l 符号链接文件
r 可读  w 可操作  x 可执行
\-  rwx  |  r-x       |   r--       |           owner             group
owner|  group  | others |     owner_name   group_name

------

文本流通过stdin(键盘..)输入通过stdout stderr(屏幕...)输出
[~ @root]$  stdin
stdout  || stderr

stdin  文件描述符 0    也是文件
stdout 文件描述符 1  也是文件
stderr 文件描述符 2   也是文件

------

\> 将stdin 输出到指定文件 而不是stdout||stderr   输出重定向指令
cat hello.txt > a.txt 本来应该输出到stdout中的内容,被重定向到了a.txt中
cat hello.txt >> a.txt  不会覆盖a.txt原有内容,而是追加
tr 'skz' 'wxy' <  ./a.txt 将a.txt中的skz替换为wxy   输入重定向

| 管道 命令1的stdout作为命令2的stdin    管道不会传递stderr
命令2 必须是能接收文本输入流的命令  grep cut wc(文本统计) head tail  less more tr sort….

------

rpm -q xxx  查询
rpm -ivh xxxx.rpm  安装
rpm -e xxx  卸载
rpm -ql nginx 查询服务安装目录
yum search 查找包
yum list installed 查看已安装
yum install 安装   (是对rpm的封装, 使用的rpm文件)
yum list 查询已安装
yum remove 卸载yum安装软件 
wget url下载文件
curl -v | python -m json.tool

------

systemctl start docker 启动服务
systemctl enable docker 自启动
top 查看系统运行状态
ps -ef | grep redis 线程  带[]为内核态进程 tty为?的为后台运行
lsof -i:port 查看端口号
netstat -tnlp | grep :22 查看端口占用
netstat -nat | grep 8080  查看8080端口占用
ssh -o ServerAliveInterval=60 root@106.12.194.111
ssh  ./slb-devops.pem root@120.77.170.116
scp D:\\Docker\a.txt root@118.25.60.179:/root 上传文件
scp  -i  ./slb-devops.pem root@120.77.170.116:/root/wiki.log  ./wiki.log 用私钥下载文件

------

systemctl start firewalld 开启防火墙
firewall-cmd --query-port=6379/tcp  查看某个端口是否开放
firewall-cmd --zone=public --add-port=6379/tcp --permanent  永久开放端口 
firewall-cmd --reload  重启防火墙
iptables -F 关闭iptables
iptables -nv -L 详细查看
sestatus -v 查看SELinux 模块状态
setenforce 0 临时关闭SELinux

------

ln -s ./nginx-docker/html/index.html  ./index-link 创建符号链接,如果被指向文件的硬链接数目为1的话,删除被指向的file_name则会发生断链
ln ./docker/uac.yml ./uac-link   创建硬链接,本质让文件inode_number对应对个file_name.删除一个file_name通过其他file_name仍旧可以访问inode_number指向的file_data

------

通配字符串包括空
?  通配单个字母包括空
[0-8] 匹配0-8任意字符
[a,c,d] 匹配任意单一字符
{skz,wxy,sh} 匹配任意字符串
! 取反
[!a-f] 不包含a-f任意字符

------

find / -name fileName 查找文件
find ./ -iname "*.txt" -exec -rf {} \;  #i忽略大小写 \; 是转义;的意思
find ./ -name "*.txt" | xargs -I {} mv {} ./txtdir/ # 查找并移动
find / -size  +100k  找出size +大于 -小于  xx的文件
find / -type l 按类型查找 , 查找所有链接文件
find / -atime 3 -ll 找出三天内访问过的文件  -a(access) -m(modify)  并展示详细信息
find / \\(-size +1k -a -size -10m \\) 查找1k-10m之间的文件 -a 与  -o 或 ! 非

locate .jar  查找.jar文件 速度快 
which redis  查找redis路径
history 查看历史命令
! history_index  执行历史命令
dmidecode -s system-product-name  查看是物理机还是虚拟机

------

ps -ef | grep .. | less  显示进程信息
top 动态显示进程信息. 3秒刷新 

------

lsof -i:8080 | grep java | awk '{ print $2 }' | xargs kill -9 

------

\> info.log 清空info.log内容
echo hello >> a.txt 将hello追加到a.txt
du -h info.log 查看文件大小
du -sh * 当前目录文件大小
df -hl 查看磁盘容量

------

cat 适合小文件
less 支持上下键,pageDown pageUp键  
       :/skz 可以高亮当前屏中所有skz     n跳转下一个高亮 , N跳转上一个高亮  q退出
head -n 20  ./hello.txt  查看前20行
tail -f -n 2  a.txt  循环监视文件 a.txt 最后两行内容
echo sunkz > a.txt 全覆盖
echo sukz >> a.txt 追加文本

------

tar -zcvf ./skz.tar.gz ./skz 将skz目录打包并以gzip方式压缩为 skz.tar.gz
tar -zxvf xxx.tar.gz 解压
zip -r ./skz ./skz.zip
unzip AUTH.war -d AUTH 解压war
chmod 777 hello.txt 修改权限 
chmod +x auto.sh 给文件添加可执行权限

------

nohup java -jar uac.jar >log.out &  # 后台运行输出日志到指定文件
jobs #查看后台运行任务
lsof -i:8080 端口占用

-------

v2ray : debian

apt-get update -y && apt-get install curl -y

bash <(curl -s -L https://git.io/v2ray.sh)