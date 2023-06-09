# 1.查看内核是否支持rbd
* modprobe rbd 
如果指令正常运行，不报错表示支持
# 2.创建Pool
pool是ceph的逻辑分区，类似K8s的NS，可以给不同的Pool设置不同的副本数，数据块大小等等 \
创建一个叫testpool的pool
* ceph osd pool create testpool 256
查看创建的pool
* ceph osd lspools
# 3.创建rbd
在testpool下创建一个叫myrbd的rbd
* rbd create testpool/myrbd --size 10240
# 4.映射块设备到机器
* rbd feature disable testpool/myrbd object-map fast-diff deep-flatten
* rbd map testpool/myrbd
成功运行后，会返回如下一个块设备名称，如下/dev/rbd0
```text
root @bigdata1 14:48:26 ~/ceph-deploy # rbd feature disable testpool/myrbd object-map fast-diff deep-flatten
root @bigdata1 14:48:36 ~/ceph-deploy # rbd map testpool/myrbd
/dev/rbd0
```
查看dev的设备信息，可以发现rbd0
```text
root @bigdata1 14:50:40 ~/ceph-deploy # ll /dev/ | grep rbd
drwxr-xr-x 3 root root          60 Apr 24 14:48 rbd
brw-rw---- 1 root disk    251,   0 Apr 24 14:48 rbd0
```
# 5.挂载使用rbd
```shell
# 创建一个待挂载的文件夹
mkdir /mnt/myrbd
# 设置rbd块的文件系统
mkfs.ext4 -m0 /dev/rbd0
# 挂载
mount /dev/rbd0 /mnt/myrbd
```
如果运行正常，则可以在/mnt/myrbd文件夹中自由读写文件