
## 本k8s 离线安装教程是基于  https://blog.csdn.net/u012375924/article/details/78987263 知道安装，可以参照这边文章，下面的是个人总结


1,安装是基于1.9.0 kubeadm 安装的. 链接: http://blog.51cto.com/cstsncv/2061943

链接：https://pan.baidu.com/s/1pMdK0Td 密码：zjja
## 或者 https://pan.baidu.com/s/1iP6AwY5fv-2muY8LDFcfhA 


环境要求 :centos7及以上

2, 保证 selinux ,firewalld 处于关闭状态,

3, 解压文件 ,如果不存在工具包,则安装  yum -y install bzip2 
   tar -jxvf k8s_images.tar.bz2

4, 安装docker ,如果之前安装docker,请卸载干净  rpm -qa |grep docker 

rpm -ivh docker-ce-selinux-17.03.2.ce-1.el7.centos.noarch.rpm
rpm -ivh docker-ce-17.03.2.ce-1.el7.centos.x86_64.rpm
  

5,  安装启动docker,关闭selinux,firewalld,后
配置系统路由参数,防止kubeadm报路由警告:

echo "
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
" >> /etc/sysctl.conf

执行 sysctl -p


6 导入镜像:
   cd k8s_images/docker_images/

   for i in `ls`;do docker load < $i ;done

cd ..

7 安装软件

rpm -ivh socat-1.7.3.2-2.el7.x86_64.rpm

rpm -ivh kubernetes-cni-0.6.0-0.x86_64.rpm  kubelet-1.9.9-9.x86_64.rpm  kubectl-1.9.0-0.x86_64.rpm

rpm -ivh kubectl-1.9.0-0.x86_64.rpm
rpm -ivh kubeadm-1.9.0-0.x86_64.rpm


8,  因为安装的是docker17.03 ,默认采用是 cgroupfs ,故需要更改相关参数
vi /etc/systemd/system/kubelet.service.d/10-kubeadm.conf

把 "Environment="KUBELET_CGROUP_ARGS=--cgroup-driver=systemd" 改成 "Environment="KUBELET_CGROUP_ARGS=--cgroup-driver=cgroupfs"

重启reload

systemctl daemon-reload && systemctl restart kubel


# 以上是master和slave节点都要进行的操作


# 9, master 进行特殊的操作

# 9.1     kubeadm init --kubernetes-version=v1.9.0 --pod-network-cidr=10.244.0.0/16
完成后,将kubeadm join xxx保存下来，等下node节点加入集群需要使用
如果忘记了，可以在master上通过kubeadmin token list得到.
如果没启动起来,执行第8步操作,修改文件,启动服务
# 9.2    kubeadm reset
# 9.3
在重新执行:
kubeadm init --kubernetes-version=v1.9.0 --pod-network-cidr=10.244.0.0/16

# 9.4
此时root用户还不能使用kubelet控制集群需要，按提示配置下环境变量
对于root用户
     export KUBECONFIG=/etc/kubernetes/admin.conf
也可以直接放到~/.bash_profile
    echo "export KUBECONFIG=/etc/kubernetes/admin.conf" >> ~/.bash_profile
source一下环境变量
# 9.5
source ~/.bash_profile

# 9.6
kubectl version测试:
[root@master k8s_images]# kubectl version


#  9.7
kubectl create -f kube-flannel.yml


## 10,  slave 要进行的特殊操作

在执行前八步之后,slave 只需要执行 集群加入操作即可.
   kubeadm join --token 5ca674.b6852c3519d873de 10.113.1.191:6443 --discovery-token-ca-cert-hash sha256:93e7ce2b7475585b4cddabeeb9e67eba1160583d3387ef518a4b8620f3258aac



11, 部署ui界面

https://blog.csdn.net/newcrane/article/details/78719349

https://blog.csdn.net/u012375924/article/details/78987263





https://10.113.1.191:32666/#!/login



# kubectl -n kube-system get secret | grep kubernetes-dashboard-admin
         kubernetes-dashboard-admin-token-hwwcs           kubernetes.io/service-account-token   3         2m

[root@php-server-191 k8s_images]# kubectl -n kube-system get secret | grep kubernetes-dashboard-token
kubernetes-dashboard-token-4dcxv                 kubernetes.io/service-account-token   3         32m

[root@php-server-191 k8s_images]# kubectl describe -n kube-system secret/kubernetes-dashboard-token-4dcxv



获取目前的服务信息.




k8s 部署nginx 容器服务.

http://www.cnblogs.com/puroc/p/5764330.html
nginx 的服务信息.
http://10.113.1.191:31740/


#### 相关的k8s操作命令



导入image包

docker load < kubernetes-dashboard_v1.8.1.tar

[root@php-server-191 k8s_images]# kubectl create -f kubernetes-dashboard.yaml
secret "kubernetes-dashboard-certs" created
serviceaccount "kubernetes-dashboard" created
role "kubernetes-dashboard-minimal" created
rolebinding "kubernetes-dashboard-minimal" created
deployment "kubernetes-dashboard" created
service "kubernetes-dashboard" created



http://10.113.1.191:32666/

kubectl get pods -n kube-system -o wide

kubectl get pods -n kube-system -o wide |grep kubernetes-dashboard

删除相关服务 
kubectl delete -f kubernetes-dashboard.yaml

kubectl  logs kubernetes-dashboard-7486b894c6-x2dtv --namespace=kube-system

kubectl get service
 

