* [参考资料1](https://blog.csdn.net/gaofei0428/article/details/118650015)
# 1. 初始化主机
## 1.1 修改主机名
修改hostname，注意进行节点职责划分（谁是master、monitor、osd）
## 1.2 修改hosts文件
让hostname与ip进行关联
## 1.3 节点之前配置无密码登录
只需要配置主节点（master）到其它节点无密码登入即可
## 1.4 安装依赖软件
```shell
yum -y install deltarpm
```
## 1.5 关闭防火墙、iptables、selinux
```shell
systemctl stop firewalld && systemctl disable firewalld 
systemctl stop iptables && systemctl disable iptables
sed -i 's/enforcing/disabled/' /etc/selinux/config
```
## 1.6 机器时间同步
* chrony方式
```shell
yum install chrony -y
systemctl start chronyd
systemctl enable chronyd
```


# 2. 安装ceph-deploy
在**master**节点进行安装
## 2.1 配置epel
```shell
yum -y install epel-release
wget -O /etc/yum.repos.d/CentOS-Base.repo https://repo.huaweicloud.com/repository/conf/CentOS-7-reg.repo 
```
## 2.2 配置ceph的yum源
```shell
vim /etc/yum.repos.d/ceph.repo
[Ceph]
name=Ceph packages for $basearch
baseurl=http://download.ceph.com/rpm-jewel/el7/$basearch
enabled=1
gpgcheck=1
type=rpm-md
gpgkey=https://download.ceph.com/keys/release.asc
priority=1

[Ceph-noarch]
name=Ceph noarch packages
baseurl=http://download.ceph.com/rpm-jewel/el7/noarch
enabled=1
gpgcheck=1
type=rpm-md
gpgkey=https://download.ceph.com/keys/release.asc
priority=1

[ceph-source]
name=Ceph source packages
baseurl=http://download.ceph.com/rpm-jewel/el7/SRPMS
enabled=1
gpgcheck=1
type=rpm-md
gpgkey=https://download.ceph.com/keys/release.asc
priority=1


```
## 2.3 下载ceph-deploy
```shell
yum clean all
yum makecache
yum repolist all
yum install ceph-deploy -y
yum install yum-plugin-priorities -y
```

# 3.搭建集群
在**master**节点运行
## 3.1 创建目录用于保存ceph-deploy配置文件
```shell
mkdir /root/ceph-deploy && cd /root/ceph-deploy
```
## 3.2 创建集群配置文件
```shell
# 指定monitor节点
# 根据规划指定，假设bigdata1为admin、bigdata2为monitor
# 那么在就是执行ceph-deploy new bigdata2
ceph-deploy new monitor节点的hostname
# 如果运行成功会生成如下文件
# ceph.conf  ceph-deploy-ceph.log  ceph.mon.keyring
# ceph默认osd是3个，如果需要修改可以在文件中配置，如配置成2个
# osd_pool_default_size = 3
```
## 3.3 安装Ceph集群
在master节点上用ceph-deploy install 所有节点，进行软件安装
> ceph-deploy install 节点1 节点2 ... 节点n
>> 如果安装缓慢或者失败，建议将deploy上的ceph的yum文件同步到所有节点，如果在所有节点
>> 先yum -y install ceph-release 再执行yum -y install ceph ceph-radosgw
>> 运行完毕后建议再在master节点上运行一次ceph-deploy install 节点1 节点2 ... 节点n
## 3.4 初始化monitor
初始化指令，在master运行
* ceph-deploy mon create-initial
运行成功后，在规划的mon节点查看是否成功运行
* ps -ef | grep ceph
正常的话可以看到如下结果：\
ceph      6045     1  0 11:18 ?        00:00:00 /usr/bin/ceph-mon -f --cluster ceph --id bigdata2 --setuser ceph --setgroup ceph
## 3.5 初始化OSD
在osd节点上创建一个文件夹，并设置文件夹权限777
如：
* mkdir /var/local/osd
* chmod 777 /var/local/osd
在master节点prepare osd节点
* ceph-deploy osd prepare osd节点:/var/local/osd
在master节点active osd节点
* ceph-deploy osd activate osd节点:/var/local/osd
如果正常运行，在osd节点，可以看到如下内容
*  ps -ef | grep ceph
ceph     14859     1  0 14:13 ?        00:00:00 /usr/bin/ceph-osd -f --cluster ceph --id 0 --setuser ceph --setgroup ceph
## 3.6 拷贝配置到其余节点
在master节点上运行如下命令，将配置拷贝到非master的ceph节点
* ceph-deploy admin master节点 其余节点
在所有节点上，修改配置文件的执行权限
* chmod +r /etc/ceph/ceph.client.admin.keyring
执行完操作后，在所有节点执行健康检查操作，看节点是否运行正常
* ceph health
# 3.7 扩容osd
就是针对新的osd节点，基于3.5和3.6进行操作。
# 3.8 删除osd
首先在master节点用ceph osd tree看有哪些osd节点,如下发现有两个osd节点osd.0和osd.1
```text
root @bigdata1 14:28:59 ~/ceph-deploy # ceph osd tree
ID WEIGHT  TYPE NAME         UP/DOWN REWEIGHT PRIMARY-AFFINITY 
-1 0.39059 root default                                        
-2 0.19530     host bigdata3                                   
 0 0.19530         osd.0          up  1.00000          1.00000 
-3 0.19530     host bigdata4                                   
 1 0.19530         osd.1          up  1.00000          1.00000 
```
发现有osd.0和osd.1，假设要关闭osd.0和osd.1
* systemctl stop ceph-osd@0
* systemctl stop ceph-osd@1
将节点标记为out，down
* ceph osd out 0
* ceph osd out 1
* ceph osd down 0
* ceph osd down 1
进行移除节点
* ceph osd crush remove osd.0
* ceph osd crush remove osd.1
删除节点
* ceph osd rm 0
* ceph osd rm 1
