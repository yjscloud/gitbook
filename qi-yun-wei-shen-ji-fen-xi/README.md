# 七、运维审计分析

## 一、监控的理解

### 1. 监控对象

（1）监控对象的理解：我们要监控谁？假设我们要监控CPU，那么CPU我们就必须知道他是怎么工作，必须了解它的工作原理。

（2）监控对象的指标：我们监控CPU的那些参数？例如我们可以监控CPU的使用率、CPU负载 、CPU个数。

（3）确定性能基准线：怎么才算故障？CPU负载多少才算高？我们要制定一个标准来评判故障，这个标准要根据服务器跑的不同业务来制定。

### 2. 监控范围

#### （1）硬件监控

通常情况下对硬件的监控我们一般会使用IPMI工具加之人工的机房巡检。不同的服务器也有各自的远程控制卡 例如： DELL服务器：iDRAC HP服务器：ILO IBM服务器：IMM linux就可以使用IPMI BMC控制器

ipmitool工具的使用需要硬件支持，操作系统通常为Linux

在Linux下安装：

```text
yum -y install OpenIPMI ipmitool
```

使用IPMI有两种方式：

* 本地调用
* 远程调用 ip地址 

路由器和交换机通常使用SNMP监控，Linux也支持SNMP监控，开启SNMP监控通常需要安装如下几个rpm包

```text
 yum -y install net-snmp net-snmp-utils
```

#### （2）操作系统监控

对与操作系统我们通常会监控CPU、内存、IO 进程（网络、磁盘）

监控的基准：

* 确定服务类型：IO密集型、数据库、cpu密集型、web mail
* 确定性能基准线：运行队列数、cpu使用率、上下文切换

最常用的监控命令：`top`、`vmstat`、`mpstat`

```text
    top命令的两个参数
        P：cpu使用率排序
        M：内存使用率排序
```

CPU三个重要的概念：

* 上下文切换：cpu调度器实施的进程切换过程，上下文切换
* 运行队列（负载）
* 使用率

查看内存使用率的命令：`free` `vmtstat` 对于内存我们要知晓页的大小（通常情况是4KB），关心内存如何寻址和所剩空间

硬盘我们常用 `iotop`（了解`dd`命令）、`iostat`来测试其性能，当然对于硬盘我们首要关心的是它的顺序IO和随机IO

查看网络的性能通常使用`iftop`命令，测试网络IO推荐使用IBM的测试工具：nmon（二进制文件）

#### （3）应用服务监控

这里举例监控nginx

安装nginx：

```text
yum -y install gcc glibc gcc--c++ pcre-devel openssl-devel #安装必要rpm包
cd /usr/local/src
wget http://nginx.org/download/nginx-1.10.1.tar.gz
tar zxvf nginx-1.10.1.tar.gz
# configure shell脚本，执行它生成makefile
useradd -s /sbin/nologin -M www   #创建nginx用户
./configure --prefix=/usr/local/nginx-1.10.1 \
--user=www --group=www \
--with-http_ssl_module \
--with-http_stub_status_module

make && make install
ln -s /usr/local/nginx-1.10.1 /usr/local/nginx  #创建软连接
```

修改配置文件： 在配置文件添加

```text
  location /nginx-status {
      stub_status on;
      access_log off;
       allow all;
   }
```

![7-1](http://pded8ke3e.bkt.clouddn.com/7-1.png)

启动nginx：

```text
/usr/local/nginx/sbin/nginx -t   #检查配置文件
/usr/local/nginx/sbin/nginx  #启动nginx
```

监控的效果：

```text
[root@host-10-197-22-12 ~]# curl -s http://10.197.22.12:8080/nginx-status
Active connections: 1 
server accepts handled requests
 1152438 1152438 1152488 
Reading: 0 Writing: 1 Waiting: 0
```

#### （4）业务监控

我所处岗位暂不涉及监控业务的内容，所以这里就简单扯一下。 监控业务我的理解是我们要知道那个时间端我们网站什么的访问量最大、用户在线最大、请求最频繁，那个时间端最空闲。在业务繁忙的时间端我们要如何确保业务正常这是我们运维要思考的问题。

如何做好监控？要做那些监控？这里我按照我工作中所遇到的场景进行简单的介绍！

