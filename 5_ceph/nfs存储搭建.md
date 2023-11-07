# 服务端
```text
1.nfs 安装
yum install -y nfs-utils

2.创建共享目录文件夹
mkdir /share
chmod 777 /share

3.修改配置文件传输规则 
vi /ect/exports
/share *(rw,sync,no_subtree_check,no_root_squash,insecure)
 
4.开启nfs和rpcbind服务
# 重启服务
systemctl restart rpcbind
systemctl restart nfs-server
# 设置开机自启
systemctl  enable  rpcbind
systemctl enable nfs-server
 
5.检查 挂载
showmount -e localhost
 
6.查询NFS的状态
查询服务状态  systemctl status nfs
停止服务  systemctl stop nfs
开启服务  systemctl start nfs
重启服务  systemctl restrart nfs
```

# 客户端
```text

1.安装nfs-utils
yum install nfs-utils -y

2.创建目录，赋予权限
mkdir  /client-share
chmod 777 /client-share

3.执行nfs挂载 
mount -t nfs 服务端ip:服务端共享目录全路径 客户端要挂载的目录
 
4.查看客户端挂载信息
df -h
```

# 定义PVC
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: nfs-static-pvc
spec:
  storageClassName: nfs
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
```

# 定义PV
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: nfs-1g-pv
spec:
  storageClassName: nfs
  accessModes:
    - ReadWriteMany
  capacity:
    storage: 1Gi
  nfs:
    path: /tmp/nfs/1g-pv
    server: 192.168.64.131
```

# Pod中绑定PVC
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nfs-static-pod
spec:
  volumes:
  - name: nfs-pvc-vol
    persistentVolumeClaim:
      claimName: nfs-static-pvc
  containers:
    - name: nfs-pvc-test
      image: nginx:alpine
      ports:
      - containerPort: 80
      volumeMounts:
        - name: nfs-pvc-vol
          mountPath: /tmp
```