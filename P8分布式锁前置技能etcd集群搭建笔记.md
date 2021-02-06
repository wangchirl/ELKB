#13、Rancher Master节点(Etcd开源数据库）

序号	主机名称	角色	数量	主机内网IP规划	主机外网IP	主机配置	基础软件	系统
01	etcd-Master	Node01	1	10.0.0.30	123.57.59.137	2C 4G	wget &&yum install -y etcd	CentOS7x64 1810
02	etcd-Slave	Node02	1	10.0.0.31	123.57.59.138	2C 4G	wget && yum install -y etcd	CentOS7x64 1810
03	etcd-Slave	Node03	1	10.0.0.32	123.57.59.139	2C 4G	wget && yum install -y etcd	CentOS7x64 1810
###PS:这里要注意 etcd并不是很吃硬件，如果业务量不是很大的话给2C 4G就够了，如果业务量比较大的话4G 8G-16G或更大的硬件配置，自己灵活掌握。安装的时候需要注意7.2版本坑比较多，跟etcd的版本存在兼容性的问题，自己安装的过程中容易翻车。

CentOS7x64 1810版本下载链接 链接：https://pan.baidu.com/s/14PWga1We99tkZLJyS_q0gQ 提取码：glk4

XShell XFtp Typora软件下载链接 链接：https://pan.baidu.com/s/1NZWfWqKezXewsHpV0gc2-A 提取码：48xc

etcd官方给出的硬件配置 https://github.com/etcd-io/etcd/blob/master/Documentation/op-guide/hardware.md

##13.2 设置主机hostname

hostnamectl set-hostname etcd-node01
hostnamectl set-hostname etcd-node02
hostnamectl set-hostname etcd-node03
##13.3 修改IP地址(有必要的话就设置hosts解析)

vim /etc/sysconfig/network-scripte/ifcfg-ens33
image.png

##13.4 安装etcd搭建集群环境 ###etcd集群三种方式 静态集群 动态集群 DNS集群,我们这里搭建的是静态集群。

#三台机器上都需要安装
yum install  -y etcd
安装完毕效果图

Node01 image.png Node02 image.png Node03 image.png

效验安装

rpm -qa etcd
image.png

查看安装的etcd的版本号

etcdctl -v
或
etcdctl --version
##13.5修改配置文件搭建集群环境

#修改之前先备份
cp /etc/etcd/etcd.conf /etc/etcd/etcd.conf.bak
cd /etc
vim etcd.conf
image.png

image.png

image.png

保存退出

systemctl restart etcd
systemctl status etcd
##13.6 配置防火墙开放端口C7--firewalld

firewall-cmd --zone=public --add-port=2379/tcp --permanent
firewall-cmd --zone=public --add-port=2380/tcp --permanent
firewall-cmd --reload && firewall-cmd --list-ports
##13.7 查看集群列表

etcdctl member list
image.png 查看集群健康状态

etcdctl cluster-health
image.png

##13.8 容易出现的问题 定时任务/计划任务/定时同步时间

yum install -y ntp
crontab -e
0 1 * * * /usr/sbin/ntpdate ntp.sjtu.edu.cn >> /var/log/ntpdate.log 2>&1 &
各个节点配置crontab 定时同步时间，否则可能会出现node1 etcd[1657]: the clock difference against peer f63afbe816fb463d is too hig问题

##13.9 测试集群是否成功

Master节点  etcdctl set name liang

Node02-Slave  etcdctl get name

Node03-Slvae etcdctl get name
image.png