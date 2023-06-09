# 1.安装ceph-common
k8s要使用ceph，需要在每个k8s节点都安装ceph-common，并把ceph的配置文件拷贝到k8s各节点上
```shell
# 把ceph节点上的ceph.repo拷贝到k8s节点上，以便yum源下载
yum install -y ceph-common 
scp /etc/ceph k8s节点:/etc
```
# 2.创建一个rdb
```shell
ceph osd pool create k8srbd 256
rbd create rbda -s 1024 -p k8srbd
rbd feature disable k8srbd/rbda object-map fast-diff deep-flatten
```
# 3.测试pod挂载创建的rbd
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: testrbd
spec:
  containers: 
    - image: nginx:1.7.9
      name: nginx
      volumeMounts: 
        - name: testrbd
          mountPath: /mnt
  volumes: 
    - name: testrbd
      rbd: 
        monitors:
          - '192.168.0.18:6789'
        pool: k8srbd
        image: rbda
        # 选择块设备使用哪种文件系统
        fsType: xfs
        readOnly: false
        user: admin
        keyring: /etc/ceph/ceph.client.admin.keyring
```

# 4.关联PV与PVC
## 4.1 创建一个k8s secret对象
获取client.admin的keyring值，并用base64编码
* ceph auth get-key client.admin | base64
QVFEcDlFVmthRWRLT1JBQWtpUzZ0QW5XSkVYQnU0MHcvRDBWL3c9PQ== 
创建ceph的secret对象
```yaml
apiVersion: v1
kind: Secret
metadata: 
 name: ceph-secret
data:
 key: QVFEcDlFVmthRWRLT1JBQWtpUzZ0QW5XSkVYQnU0MHcvRDBWL3c9PQ==
```
## 4.2 创建rbd pool池
```shell
ceph osd pool create pvcrbd 256
rbd create rbda -s 1024 -p pvcrbd
rbd feature disable pvcrbd/rbda object-map fast-diff deep-flatten
```
## 4.3 创建PV
```yaml
apiVersion: v1
kind: PersistentVolume
metadata: 
  name: ceph-pv
spec: 
 capacity: 
  storage: 1Gi
 accessModes:
   - ReadWriteMany
 rbd:
  monitors:
    - 192.168.0.18:6789
  pool: pvcrbd
  image: rbda
  user: admin
  secretRef: 
    name: ceph-secret
  fsType: xfs
  readOnly: false
 persistentVolumeReclaimPolicy: Recycle
```
## 4.4 创建PVC
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
 name: ceph-pvc
spec:
 accessModes:
   - ReadWriteMany
 resources:
   requests:
    storage: 1Gi
```
## 4.5创建pod绑定PVC
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations: {}
  labels:
    k8s.kuboard.cn/name: nginx-deploy
  name: nginx-deploy
  namespace: default
spec:
  replicas: 2
  selector:
    matchLabels:
      k8s.kuboard.cn/name: nginx-deploy
  template:
    metadata:
      labels:
        k8s.kuboard.cn/name: nginx-deploy
    spec:
      containers:
        - image: 'nginx:1.7.9'
          name: nginx
          volumeMounts:
            - mountPath: /mnt
              name: ceph-data
      volumes:
        - name: ceph-data
          persistentVolumeClaim:
            claimName: ceph-pvc
```