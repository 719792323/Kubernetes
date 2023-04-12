# minikube的删除步骤
```text
# 停止虚拟机运行
minikube stop; 
# 卸载虚拟机命令
minikube delete
# 删除docker中的minikube容器
docker rm -f minikube的容器 && docker rmi minikube的镜像
# 删除相关文件
rm -rf ~/.kube ~/.minikube
# 如果minikube安装在/usr/local/bin/minikube
rm -rf /usr/local/bin/minikube
# 如果使用的是如rpm -ihv minikube-1.23.1-0.x86_64.rpm安装
rpm -e minikube-1.23.1-0.x86_64
# 删除kubelet，用which kubelet找到路径
rm -f kubelet的安装路径
# 删除相关环境配置
rm -rf /etc/kubernetes/
```