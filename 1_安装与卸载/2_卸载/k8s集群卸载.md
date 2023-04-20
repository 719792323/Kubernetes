# 1.安装包卸载
## 1.1 基于yum卸载
* 卸载k8s
```shell
yum remove -y kubelet kubeadm kubectl
kubeadm reset -f
modprobe -r ipip
lsmod
rm -rf ~/.kube/
rm -rf /etc/kubernetes/
rm -rf /etc/systemd/system/kubelet.service.d
rm -rf /etc/systemd/system/kubelet.service
rm -rf /usr/bin/kube*
rm -rf /etc/cni
rm -rf /opt/cni
rm -rf /var/lib/etcd
rm -rf /var/etcd
yum clean all
yum makecache
```
# 2. 删除所有k8s相关容器
```shell
docker rm -f $(docker ps -a | grep k8s | awk '{print $1}')
docker rmi -f $(docker images -a | grep kube | awk '{print $3}')
```
# 3. 重启docker与damon
```shell
systemctl daemon-reload
systemctl restart docker
```