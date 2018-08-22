# 6. 安装配置glance

Glance组件服务在云端主要作用是给计算节点安装实例时，提供实例镜像文件。另外采用了NFS文件管理系统作为镜像文件的后端存储系统，方便三个控制节点共用同一个共享。

1）在controller1上创建glance数据库

```text
MariaDB [(none)]> CREATE DATABASE glance;
```

2）在controller1上创建数据库用户并赋予权限

```text
MariaDB [(none)]> GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'localhost' IDENTIFIED BY 'yjscloud';
MariaDB [(none)]> GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'%' IDENTIFIED BY 'yjscloud';
```

3）在controller1上创建glance用户及赋予admin权限

```text
source /root/admin-openrc
openstack user create --domain default glance --password yjscloud
openstack role add --project service --user glance admin
```

4）在controller1上创建image服务

```text
openstack service create --name glance --description "OpenStack Image service" image
```

5）在controller1上创建glance的endpoint

```text
openstack endpoint create --region RegionOne image public http://yjscloud.com:9292
openstack endpoint create --region RegionOne image internal http://yjscloud.com:9292
openstack endpoint create --region RegionOne image admin http://yjscloud.com:9292
```

6）在controller1、2、3上安装glance相关rpm包

```text
yum install openstack-glance -y
```

7）在controller1、2、3上修改glance配置文件`/etc/glance/glance-api.conf` 注意把配置文件的密码设置成你自己的

```text
cp /etc/glance/glance-api.conf /etc/glance/glance-api.conf.bak
>/etc/glance/glance-api.conf
openstack-config --set /etc/glance/glance-api.conf DEFAULT debug False
openstack-config --set /etc/glance/glance-api.conf DEFAULT verbose True
openstack-config --set /etc/glance/glance-api.conf DEFAULT bind_host controller1
openstack-config --set /etc/glance/glance-api.conf DEFAULT bind_port 9393
openstack-config --set /etc/glance/glance-api.conf DEFAULT registry_host controller1
openstack-config --set /etc/glance/glance-api.conf DEFAULT registry_port 9191
openstack-config --set /etc/glance/glance-api.conf DEFAULT auth_region RegionOne
openstack-config --set /etc/glance/glance-api.conf DEFAULT registry_client_protocol http
openstack-config --set /etc/glance/glance-api.conf DEFAULT show_image_direct_url False
openstack-config --set /etc/glance/glance-api.conf DEFAULT workers 4
openstack-config --set /etc/glance/glance-api.conf DEFAULT backlog 4096
openstack-config --set /etc/glance/glance-api.conf DEFAULT image_cache_dir /var/lib/glance/image-cache
openstack-config --set /etc/glance/glance-api.conf DEFAULT rpc_backend rabbit
openstack-config --set /etc/glance/glance-api.conf DEFAULT scrub_time 43200
openstack-config --set /etc/glance/glance-api.conf DEFAULT delayed_delete False
openstack-config --set /etc/glance/glance-api.conf DEFAULT enable_v1_api False
openstack-config --set /etc/glance/glance-api.conf DEFAULT enable_v2_api True
openstack-config --set /etc/glance/glance-api.conf oslo_messaging_rabbit rabbit_hosts controller1:5672,controller2:5672,controller3:5672
openstack-config --set /etc/glance/glance-api.conf oslo_messaging_rabbit rabbit_userid openstack
openstack-config --set /etc/glance/glance-api.conf oslo_messaging_rabbit rabbit_password yjscloud
openstack-config --set /etc/glance/glance-api.conf oslo_messaging_rabbit rabbit_use_ssl False
openstack-config --set /etc/glance/glance-api.conf oslo_messaging_rabbit rabbit_ha_queues True
openstack-config --set /etc/glance/glance-api.conf oslo_messaging_rabbit rabbit_retry_interval 1
openstack-config --set /etc/glance/glance-api.conf oslo_messaging_rabbit rabbit_retry_backoff 2
openstack-config --set /etc/glance/glance-api.conf oslo_messaging_rabbit rabbit_max_retries 0
openstack-config --set /etc/glance/glance-api.conf oslo_messaging_rabbit amqp_durable_queues False
openstack-config --set /etc/glance/glance-api.conf oslo_concurrency lock_path /var/lock/glance
openstack-config --set /etc/glance/glance-api.conf database connection mysql+pymysql://glance:yjscloud@yjscloud.com/glance
openstack-config --set /etc/glance/glance-api.conf database idle_timeout 3600
openstack-config --set /etc/glance/glance-api.conf database max_pool_size 30
openstack-config --set /etc/glance/glance-api.conf database max_retries -1
openstack-config --set /etc/glance/glance-api.conf database retry_interval 2
openstack-config --set /etc/glance/glance-api.conf database max_overflow 60
openstack-config --set /etc/glance/glance-api.conf keystone_authtoken auth_uri http://yjscloud.com:5000
openstack-config --set /etc/glance/glance-api.conf keystone_authtoken auth_url http://yjscloud.com:35357
openstack-config --set /etc/glance/glance-api.conf keystone_authtoken memcached_servers controller1:11211,controller2:11211,controller3:11211
openstack-config --set /etc/glance/glance-api.conf keystone_authtoken auth_type password
openstack-config --set /etc/glance/glance-api.conf keystone_authtoken project_domain_name default
openstack-config --set /etc/glance/glance-api.conf keystone_authtoken user_domain_name default
openstack-config --set /etc/glance/glance-api.conf keystone_authtoken username glance
openstack-config --set /etc/glance/glance-api.conf keystone_authtoken password yjscloud
openstack-config --set /etc/glance/glance-api.conf keystone_authtoken project_name service
openstack-config --set /etc/glance/glance-api.conf keystone_authtoken token_cache_time -1
openstack-config --set /etc/glance/glance-api.conf paste_deploy flavor keystone
openstack-config --set /etc/glance/glance-api.conf glance_store stores file,http
openstack-config --set /etc/glance/glance-api.conf glance_store default_store file
openstack-config --set /etc/glance/glance-api.conf glance_store filesystem_store_datadir /var/lib/glance/images/
```

```text
scp -p /etc/glance/glance-api.conf controller2:/etc/glance/glance-api.conf
scp -p /etc/glance/glance-api.conf controller3:/etc/glance/glance-api.conf
#注意更改controller编号
```

8）在controller1、2、3上修改glance配置文件`/etc/glance/glance-registry.conf`：

```text
cp /etc/glance/glance-registry.conf /etc/glance/glance-registry.conf.bak
>/etc/glance/glance-registry.conf
openstack-config --set /etc/glance/glance-registry.conf DEFAULT debug False
openstack-config --set /etc/glance/glance-registry.conf DEFAULT verbose True
openstack-config --set /etc/glance/glance-registry.conf DEFAULT bind_host controller1
openstack-config --set /etc/glance/glance-registry.conf DEFAULT bind_port9191
openstack-config --set /etc/glance/glance-registry.conf DEFAULT workers 4
openstack-config --set /etc/glance/glance-registry.conf oslo_messaging_rabbit rabbit_hosts controller1:5672,controller2:5672,controller3:5672
openstack-config --set /etc/glance/glance-registry.conf oslo_messaging_rabbit rabbit_userid openstack
openstack-config --set /etc/glance/glance-registry.conf oslo_messaging_rabbit rabbit_password yjscloud
openstack-config --set /etc/glance/glance-registry.conf oslo_messaging_rabbit rabbit_use_ssl False
openstack-config --set /etc/glance/glance-registry.conf oslo_messaging_rabbit rabbit_ha_queues True
openstack-config --set /etc/glance/glance-registry.conf oslo_messaging_rabbit rabbit_retry_interval 1
openstack-config --set /etc/glance/glance-registry.conf oslo_messaging_rabbit rabbit_retry_backoff 2
openstack-config --set /etc/glance/glance-registry.conf oslo_messaging_rabbit rabbit_max_retries 0
openstack-config --set /etc/glance/glance-registry.conf oslo_messaging_rabbit amqp_durable_queues False
openstack-config --set /etc/glance/glance-registry.conf database connection mysql+pymysql://glance:yjscloud@yjscloud.com/glance
openstack-config --set /etc/glance/glance-registry.conf database idle_timeout 3600
openstack-config --set /etc/glance/glance-registry.conf database max_pool_size 30
openstack-config --set /etc/glance/glance-registry.conf database max_retries -1
openstack-config --set /etc/glance/glance-registry.conf database retry_interval 2
openstack-config --set /etc/glance/glance-registry.conf database max_overflow 60
openstack-config --set /etc/glance/glance-registry.conf keystone_authtoken auth_uri http://yjscloud.com:5000
openstack-config --set /etc/glance/glance-registry.conf keystone_authtoken auth_url http://yjscloud.com:35357
openstack-config --set /etc/glance/glance-registry.conf keystone_authtoken memcached_servers controller1:11211,controller2:11211,controller3:11211
openstack-config --set /etc/glance/glance-registry.conf keystone_authtoken auth_type password
openstack-config --set /etc/glance/glance-registry.conf keystone_authtoken project_domain_name default
openstack-config --set /etc/glance/glance-registry.conf keystone_authtoken user_domain_name default
openstack-config --set /etc/glance/glance-registry.conf keystone_authtoken project_name service
openstack-config --set /etc/glance/glance-registry.conf keystone_authtoken username glance
openstack-config --set /etc/glance/glance-registry.conf keystone_authtoken password yjscloud
openstack-config --set /etc/glance/glance-registry.conf paste_deploy flavor keystone
openstack-config --set /etc/glance/glance-registry.conf glance_store filesystem_store_datadir /var/lib/glance/images/
openstack-config --set /etc/glance/glance-registry.conf glance_store os_region_name RegionOne
```

```text
scp -p /etc/glance/glance-registry.conf controller2:/etc/glance/glance-registry.conf
scp -p /etc/glance/glance-registry.conf controller3:/etc/glance/glance-registry.conf
```

注意更改controller编号

9）controller1上同步glance数据库

```text
su -s /bin/sh -c "glance-manage db_sync" glance
```

10）在controller1、2、3上启动glance机设置开机启动

```text
systemctl enable openstack-glance-api.service openstack-glance-registry.service
systemctl restart openstack-glance-api.service openstack-glance-registry.service
systemctl status openstack-glance-api.service openstack-glance-registry.service
```

11）在controller1、2、3上将glance版本号写入环境变量openrc文件

```text
echo " " >> /root/admin-openrc && \
echo " " >> /root/demo-openrc && \
echo "export OS_IMAGE_API_VERSION=2" | tee -a /root/admin-openrc /root/demo-openrc
```

12）搭建glance后端存储 因为是HA环境，3个控制节点必须要有一个共享的后端存储，不然request发起请求的时候不确定会去调用哪个控制节点的glance服务，如果没有共享存储池存镜像，那么会遇到创建VM时候image找不到的问题。这里我们采用NFS的方式把glance的后端存储建立起来，当然在实际的生产环境中一搬会用ceph、GlusterFS的方式，这里我们以NFS为例子来讲诉后端存储的搭建。

首先准备好一台物理机或者虚拟机，要求空间要大，网络最好是在万兆 这里我们用10.1.1.155这一台虚拟机 首先在这台机器上安装glance组件：

```text
yum -y install openstack-glance python-glance python-glanceclient
```

其次安装NFS服务：

```text
yum -y install -y nfs-utils rpcbind
```

创建glance image的存储路径并赋予glance用户相应权限：

```text
mkdir -p /var/lib/glance/images
chown -R glance:glance /var/lib/glance/images
```

配置NFS把/var/lib/glance目录共享出去

```text
vim /etc/exports
```

添加的内容：

```text
/var/lib/glance *(rw,sync,no_root_squash)
```

启动相关服务，并把nfs设置开机启动：

```text
systemctl enable rpcbind
systemctl enable nfs-server.service
systemctl start rpcbind
systemctl status nfs-server
```

让NFS共享目录生效:

```text
showmount -e
```

接着在3个controller节点上做如下操作：

```text
mount -t nfs 10.1.1.155:/var/lib/glance/images /var/lib/glance/images
echo "/usr/bin/mount -t nfs 10.1.1.155:/var/lib/glance/ /var/lib/glance/" >> /etc/rc.d/rc.local
chmod +x /etc/rc.d/rc.local
df -h
```

13）在controller1上下载测试镜像文件

```text
wget http://download.cirros-cloud.net/0.3.4/cirros-0.3.4-x86_64-disk.img
```

14）在controller1上传镜像到glance

```text
source /root/admin-openrc

glance image-create --name "cirros-0.3.4-x86_64" --file cirros-0.3.4-x86_64-disk.img --disk-format qcow2 --container-format bare --visibility public --progress
```

![8-1-24](http://pded8ke3e.bkt.clouddn.com/8-1-24.jpg)

如果你做好了一个CentOS7.1系统的镜像，也可以用这命令操作，例：

```text
glance image-create --name "CentOS7.1-x86_64" --file CentOS_7.1.qcow2 --disk-format qcow2 --container-format bare --visibility public --progress
```

查看镜像列表：

```text
glance image-list
openstack image-list
```

15）其他两个节点重复6、7、8、10、11步骤

