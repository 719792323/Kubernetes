[minikube官网](https://minikube.sigs.k8s.io/docs/handbook/controls/)
# 1.安装minikube
注意：要先安装好Docker
## 1.1 下载并安装minikube
```shell
wget https://github.com/kubernetes/minikube/releases/download/v1.23.1/minikube-1.23.1-0.x86_64.rpm
rpm -ihv minikube-1.23.1-0.x86_64.rpm
```
## 1.2 启动minikube
注意：如果需要用python-client或者其它方式远程访问minikube提供的apiserver
则一定需要指定--apiserver-ips参数，该参数指定本机ip地址即可。
```shell
 minikube start \
 --force --driver=docker \
 --image-repository=registry.cn-hangzhou.aliyuncs.com/google_containers \ 
 --apiserver-ips=192.168.56.128
```
## 1.3 安装kubectl
```shell
curl -Lo kubectl    http://kubernetes.oss-cn-hangzhou.aliyuncs.com/kubernetes-release/release/v1.22.1/bin/linux/amd64/kubectl
mv kubectl /usr/bin
chmod a+x /usr/bin/kubectl
```
## 2.minikube常用指令
* minikube pause:挂起虚拟机
* minikube stop:停止虚拟机
* minikube config set memory xx:修改虚拟机内存配置
* minikube dashboard:启动 dashboard 控制台，curl 127.0.0.1:23341
* minikube delete:删除虚拟机，如果值创建了一个虚拟器用这个指令即可
* minikube delete --all:删除所有虚拟机
* minikube start:如果已经创建了虚拟机那么会重启如果没创建则会创建
* minikube start --force --driver=docker:重启指令
* minikube start -p cluster2:创建另外一个minikube集群

# 3.外部访问minikube的apiserver方法
## 3.1 拷贝minikube相关文件到本地
拷贝config、client.key、client.crt、ca.crt等文件，并参考如下对config进行修改
```yaml
apiVersion: v1
clusters:
- cluster:
    # 修改ca.crt的本地路径
    certificate-authority: ca.crt
    extensions:
    - extension:
        last-update: Wed, 12 Apr 2023 10:26:52 CST
        provider: minikube.sigs.k8s.io
        version: v1.23.1
      name: cluster_info
    # 修改server到nginx代理的地址
    server: https://192.168.56.128:8443
  name: minikube
contexts:
- context:
    cluster: minikube
    extensions:
    - extension:
        last-update: Wed, 12 Apr 2023 10:26:52 CST
        provider: minikube.sigs.k8s.io
        version: v1.23.1
      name: context_info
    namespace: default
    user: minikube
  name: minikube
current-context: minikube
kind: Config
preferences: {}
users:
- name: minikube
  user:
    # 如下两个文件修改到本地路径
    client-certificate: client.crt
    client-key: client.key
```
## 3.2 使用nginx进行代理
* 首先要注意在创建minikube集群时要开启--apiserver-ips=192.168.56.128参数
> 提供如下指令查看minikube对应的虚拟访问地址，假如结果是192.168.49.2:8443
* kubectl describe pods kube-apiserver-minikube -n kube-system | grep advertise-address.endpoint
* 下载nginx
```shell
# 添加ngixn的yum源
rpm -Uvh http://nginx.org/packages/centos/7/noarch/RPMS/nginx-release-centos-7-0.el7.ngx.noarch.rpm
# 安装nginx
yum install -y nginx
# 开启
systemctl start nginx
# 开机自启动
# systemctl enable nginx
```
* 配置nginx
```text
# vim /etc/nginx/nginx.conf
# 加入如下内容，即nginx对192.168.56.128:8443的请求代理到192.168.49.2:8443
stream {
  server {
      listen 192.168.56.128:8443;  
      proxy_pass 192.168.49.2:8443;
  }
}
#重新加载nginx配置文件并重启nginx
nginx -s reload 
systemctl restart nginx
```
