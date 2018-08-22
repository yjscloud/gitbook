# 8. 安装配置neutron

控制节点上neutron是网络的管理组件，提供网络、子网和路由器抽象功能，利用neutron可以很快搭建好私有云内部网络。本次搭建采用的是flat模式，使得内网的机子能访问外网。

1）在controller1上创建neutron数据库

```text
MariaDB [(none)]> CREATE DATABASE neutron;
```

2）在controller1创建数据库用户并赋予权限

```text
MariaDB [(none)]> GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'localhost' IDENTIFIED BY 'yjscloud';
MariaDB [(none)]> GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'%' IDENTIFIED BY 'yjscloud';
```

3）在controller1上创建neutron用户及赋予admin权限

```text
source /root/admin-openrc
openstack user create --domain default neutron --password yjscloud
openstack role add --project service --user neutron admin
```

4）在controller1上创建network服务

```text
openstack service create --name neutron --description "OpenStack Networking" network
```

5）在controller1上创建endpoint

```text
openstack endpoint create --region RegionOne network public http://yjscloud.com:9696
openstack endpoint create --region RegionOne network internal http://yjscloud.com:9696
openstack endpoint create --region RegionOne network admin http://yjscloud.com:9696
```

6）在controller1、2、3上安装neutron相关软件

```text
yum install openstack-neutron openstack-neutron-ml2 openstack-neutron-linuxbridge ebtables -y
```

7）在controller1、2、3上配置neutron配置文件`/etc/neutron/neutron.conf`

```text
cp /etc/neutron/neutron.conf /etc/neutron/neutron.conf.bak
>/etc/neutron/neutron.conf
openstack-config --set /etc/neutron/neutron.conf DEFAULT debug False
openstack-config --set /etc/neutron/neutron.conf DEFAULT verbose true
openstack-config --set /etc/neutron/neutron.conf DEFAULT bind_host controller1
openstack-config --set /etc/neutron/neutron.conf DEFAULT bind_port 9797
openstack-config --set /etc/neutron/neutron.conf DEFAULT core_plugin neutron.plugins.ml2.plugin.Ml2Plugin
openstack-config --set /etc/neutron/neutron.conf DEFAULT service_plugins neutron.services.l3_router.l3_router_plugin.L3RouterPlugin,neutron.services.metering.metering_plugin.MeteringPlugin
openstack-config --set /etc/neutron/neutron.conf DEFAULT auth_strategy keystone
openstack-config --set /etc/neutron/neutron.conf DEFAULT allow_overlapping_ips True
openstack-config --set /etc/neutron/neutron.conf DEFAULT notify_nova_on_port_status_changes True
openstack-config --set /etc/neutron/neutron.conf DEFAULT notify_nova_on_port_data_changes True
openstack-config --set /etc/neutron/neutron.conf DEFAULT advertise_mtu True
openstack-config --set /etc/neutron/neutron.conf DEFAULT rpc_response_timeout 180
openstack-config --set /etc/neutron/neutron.conf DEFAULT mac_generation_retries 32
openstack-config --set /etc/neutron/neutron.conf DEFAULT dhcp_lease_duration 600
openstack-config --set /etc/neutron/neutron.conf DEFAULT global_physnet_mtu 1500
openstack-config --set /etc/neutron/neutron.conf DEFAULT control_exchange neutron
openstack-config --set /etc/neutron/neutron.conf DEFAULT api_workers 4
openstack-config --set /etc/neutron/neutron.conf DEFAULT rpc_workers 4
openstack-config --set /etc/neutron/neutron.conf DEFAULT agent_down_time 75
openstack-config --set /etc/neutron/neutron.conf DEFAULT dhcp_agents_per_network 2
openstack-config --set /etc/neutron/neutron.conf DEFAULT router_distributed False
openstack-config --set /etc/neutron/neutron.conf DEFAULT router_scheduler_driver neutron.scheduler.l3_agent_scheduler.ChanceScheduler
openstack-config --set /etc/neutron/neutron.conf DEFAULT allow_automatic_l3agent_failover True
openstack-config --set /etc/neutron/neutron.conf DEFAULT l3_ha True
openstack-config --set /etc/neutron/neutron.conf DEFAULT max_l3_agents_per_router 0
openstack-config --set /etc/neutron/neutron.conf DEFAULT min_l3_agents_per_router 2
openstack-config --set /etc/neutron/neutron.conf database connection mysql+pymysql://neutron:yjscloud@yjscloud.com/neutron
openstack-config --set /etc/neutron/neutron.conf database idle_timeout 3600
openstack-config --set /etc/neutron/neutron.conf database max_pool_size 30
openstack-config --set /etc/neutron/neutron.conf database max_retries -1
openstack-config --set /etc/neutron/neutron.conf database retry_interval 2
openstack-config --set /etc/neutron/neutron.conf database max_overflow 60
openstack-config --set /etc/neutron/neutron.conf oslo_messaging_rabbit rabbit_hosts controller1:5672,controller2:5672,controller3:5672
openstack-config --set /etc/neutron/neutron.conf oslo_messaging_rabbit rabbit_userid openstack
openstack-config --set /etc/neutron/neutron.conf oslo_messaging_rabbit rabbit_password yjscloud
openstack-config --set /etc/neutron/neutron.conf oslo_messaging_rabbit rabbit_ha_queues True
openstack-config --set /etc/neutron/neutron.conf oslo_messaging_rabbit rabbit_retry_interval 1
openstack-config --set /etc/neutron/neutron.conf oslo_messaging_rabbit rabbit_retry_backoff 2
openstack-config --set /etc/neutron/neutron.conf oslo_messaging_rabbit rabbit_max_retries 0
openstack-config --set /etc/neutron/neutron.conf oslo_messaging_rabbit amqp_durable_queues False
openstack-config --set /etc/neutron/neutron.conf keystone_authtoken auth_uri http://yjscloud.com:5000
openstack-config --set /etc/neutron/neutron.conf keystone_authtoken auth_url http://yjscloud.com:35357
openstack-config --set /etc/neutron/neutron.conf keystone_authtoken memcached_servers controller1:11211,controller2:11211,controller3:11211
openstack-config --set /etc/neutron/neutron.conf keystone_authtoken auth_type password
openstack-config --set /etc/neutron/neutron.conf keystone_authtoken project_domain_name default
openstack-config --set /etc/neutron/neutron.conf keystone_authtoken user_domain_name default
openstack-config --set /etc/neutron/neutron.conf keystone_authtoken project_name service
openstack-config --set /etc/neutron/neutron.conf keystone_authtoken username neutron
openstack-config --set /etc/neutron/neutron.conf keystone_authtoken password yjscloud
openstack-config --set /etc/neutron/neutron.conf nova auth_url http://yjscloud.com:35357
openstack-config --set /etc/neutron/neutron.conf nova auth_type password
openstack-config --set /etc/neutron/neutron.conf nova project_domain_name default
openstack-config --set /etc/neutron/neutron.conf nova user_domain_name default
openstack-config --set /etc/neutron/neutron.conf nova region_name RegionOne
openstack-config --set /etc/neutron/neutron.conf nova project_name service
openstack-config --set /etc/neutron/neutron.conf nova username nova
openstack-config --set /etc/neutron/neutron.conf nova password yjscloud
openstack-config --set /etc/neutron/neutron.conf oslo_concurrency lock_path /var/lib/neutron/tmp
openstack-config --set /etc/neutron/neutron.conf agent report_interval 30
openstack-config --set /etc/neutron/neutron.conf agent root_helper sudo neutron-rootwrap /etc/neutron/rootwrap.conf
```

```text
scp -p /etc/neutron/neutron.conf controller2:/etc/neutron/neutron.conf
scp -p /etc/neutron/neutron.conf controller3:/etc/neutron/neutron.conf
```

注意更改节点controller编号

8）在controller1、2、3上配置配置`/etc/neutron/plugins/ml2/ml2_conf.ini`

```text
openstack-config --set /etc/neutron/plugins/ml2/ml2_conf.ini ml2 type_drivers flat,vlan,vxlan
openstack-config --set /etc/neutron/plugins/ml2/ml2_conf.ini ml2 mechanism_drivers linuxbridge,l2population
openstack-config --set /etc/neutron/plugins/ml2/ml2_conf.ini ml2 extension_drivers port_security
openstack-config --set /etc/neutron/plugins/ml2/ml2_conf.ini ml2 tenant_network_types vxlan
openstack-config --set /etc/neutron/plugins/ml2/ml2_conf.ini ml2 path_mtu 1500
openstack-config --set /etc/neutron/plugins/ml2/ml2_conf.ini ml2_type_flat flat_networks provider
openstack-config --set /etc/neutron/plugins/ml2/ml2_conf.ini ml2_type_vxlan vni_ranges 1:1000
openstack-config --set /etc/neutron/plugins/ml2/ml2_conf.ini securitygroup enable_ipset True
```

9）在controller1、2、3配置`/etc/neutron/plugins/ml2/linuxbridge_agent.ini`

```text
openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini DEFAULT debug false
openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini linux_bridge physical_interface_mappings provider:eth0
openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini vxlan enable_vxlan True
openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini vxlan local_ip 10.2.2.150   # 负责到其他2/3节点上是注意更改ip
openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini vxlan l2_population True
openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini agent prevent_arp_spoofing True
openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini securitygroup enable_security_group True
openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini securitygroup firewall_driver neutron.agent.linux.iptables_firewall.IptablesFirewallDriver
```

注意eth0是public网卡，一般这里写的网卡名都是能访问外网的，如果不是外网网卡，那么VM就会与外界网络隔离。

10）在controller1、2、3配置 `/etc/neutron/l3_agent.ini`

```text
openstack-config --set /etc/neutron/l3_agent.ini DEFAULT interface_driver neutron.agent.linux.interface.BridgeInterfaceDriver
openstack-config --set /etc/neutron/l3_agent.ini DEFAULT external_network_bridge
openstack-config --set /etc/neutron/l3_agent.ini DEFAULT debug false
```

11）在controller1、2、3配置 配置`/etc/neutron/dhcp_agent.ini`

```text
openstack-config --set /etc/neutron/dhcp_agent.ini DEFAULT interface_driver neutron.agent.linux.interface.BridgeInterfaceDriver
openstack-config --set /etc/neutron/dhcp_agent.ini DEFAULT dhcp_driver neutron.agent.linux.dhcp.Dnsmasq
openstack-config --set /etc/neutron/dhcp_agent.ini DEFAULT enable_isolated_metadata True
openstack-config --set /etc/neutron/dhcp_agent.ini DEFAULT verbose True
openstack-config --set /etc/neutron/dhcp_agent.ini DEFAULT debug false
```

12）在controller1、2、3上重新配置`/etc/nova/nova.conf`，配置这步的目的是让compute节点能使用上neutron网络

```text
openstack-config --set /etc/nova/nova.conf neutron url http://yjscloud.com:9696
openstack-config --set /etc/nova/nova.conf neutron auth_url http://yjscloud.com:35357
openstack-config --set /etc/nova/nova.conf neutron auth_plugin password
openstack-config --set /etc/nova/nova.conf neutron project_domain_id default
openstack-config --set /etc/nova/nova.conf neutron user_domain_id default
openstack-config --set /etc/nova/nova.conf neutron region_name RegionOne
openstack-config --set /etc/nova/nova.conf neutron project_name service
openstack-config --set /etc/nova/nova.conf neutron username neutron
openstack-config --set /etc/nova/nova.conf neutron password yjscloud
openstack-config --set /etc/nova/nova.conf neutron service_metadata_proxy True
openstack-config --set /etc/nova/nova.conf neutron metadata_proxy_shared_secret yjscloud
```

13）在controller1、2、3上将dhcp-option-force=26,1450写入/etc/neutron/dnsmasq-neutron.conf echo "dhcp-option-force=26,1450" &gt;/etc/neutron/dnsmasq-neutron.conf

14）在controller1、2、3上配置`/etc/neutron/metadata_agent.ini`

```text
openstack-config --set /etc/neutron/metadata_agent.ini DEFAULT nova_metadata_ip yjscloud.com
openstack-config --set /etc/neutron/metadata_agent.ini DEFAULT metadata_proxy_shared_secret yjscloud
openstack-config --set /etc/neutron/metadata_agent.ini DEFAULT metadata_workers 4
openstack-config --set /etc/neutron/metadata_agent.ini DEFAULT verbose True
openstack-config --set /etc/neutron/metadata_agent.ini DEFAULT debug false
openstack-config --set /etc/neutron/metadata_agent.ini DEFAULT nova_metadata_protocol http
```

15）在controller1、2、3上创建软链接

```text
ln -s /etc/neutron/plugins/ml2/ml2_conf.ini /etc/neutron/plugin.ini
```

16）在controller1上同步数据库

```text
su -s /bin/sh -c "neutron-db-manage --config-file /etc/neutron/neutron.conf --config-file /etc/neutron/plugins/ml2/ml2_conf.ini upgrade head" neutron
```

17）在controller1、2、3上重启nova服务，因为刚才改了nova.conf

```text
systemctl restart openstack-nova-api.service
systemctl status openstack-nova-api.service
```

18）在controller1、2、3上重启neutron服务并设置开机启动

```text
systemctl enable neutron-server.service neutron-linuxbridge-agent.service neutron-dhcp-agent.service neutron-metadata-agent.service
systemctl restart neutron-server.service neutron-linuxbridge-agent.service neutron-dhcp-agent.service neutron-metadata-agent.service
systemctl status neutron-server.service neutron-linuxbridge-agent.service neutron-dhcp-agent.service neutron-metadata-agent.service
```

19）在controller1、2、3上启动neutron-l3-agent.service并设置开机启动

```text
systemctl enable neutron-l3-agent.service
systemctl start neutron-l3-agent.service
systemctl status neutron-l3-agent.service
```

20）随便一节点上执行验证

```text
source /root/admin-openrc
neutron agent-list
```

![8-1-25](http://pded8ke3e.bkt.clouddn.com/8-1-25.jpg)

