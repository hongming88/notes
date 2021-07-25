# redis安装

## 第一步gcc

安装C 语言的编译环境

yum install gcc

测试 gcc版本 

yum --version

![安装第一步](安装第一步.jpg)

## 第二步

下载redis-6.2.1.tar.gz放/opt目录



tar -zxvf redis-6.2.1.tar.gz



解压完成后进入目录：cd redis-6.2.1



在redis-6.2.1目录下再次执行make命令（只是编译好）

成功后：

![安装第一步](安装第二步.jpg)



成功后查看默认安装目录： /usr/local/bin 


redis-benchmark:性能测试工具，可以在自己本子运行，看看自己本子性能如何
redis-check-aof：修复有问题的AOF文件，rdb和aof后面讲
redis-check-dump：修复有问题的dump.rdb文件
redis-sentinel：Redis集群使用
redis-server：Redis服务器启动命令
redis-cli：客户端，操作入口

## 修改配置文件

把redis.conf复习到/myredis下

修改

1. 后台启动设置daemonize no改成yes(支持后台启动)

2. 注视bind=127.0.0.1（变成所有人都能连，也能加ip地址）
3. protected-mode 将本机访问保护模式设置no
4. requirepass xxx(设置密码)



# 使用

## 第一种：redis-cli 

启动服务：redis-server /myredis/redis.conf

查看 ： ps -ef|grep redis

客户端连接（一定要先启动服务）：

1.  redis-cli -a xxx(xxx是密码)
2. 先redis-cli 再AUTH xxx

关闭：

1. 查看进程号后用: kill -9 进程号
2. redis-cli -a xxx shutdown（xxx是密码）

## 第二种（RDM）

打开端口：

`vi /etc/sysconfig/iptables`

加入防火墙规则：

`-A INPUT -m state –state NEW -m tcp -p tcp –dport 6379 -j ACCEPT`

重启防火墙：

`service iptables status`

查看端口是否开放：

netstat -aptn  



如果在某云服务器，注意增加安全组策略





附上几条防火墙命令：

service iptables status # 查看iptables状态
service iptables restart # iptables服务重启
service iptables stop  # iptables服务禁用



