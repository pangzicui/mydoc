# 运行Minikube
### 下载
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
### 安装
sudo install minikube-linux-amd64 /usr/local/bin/minikube
### 创建k8s用户
useradd k8s
passwd k8s
### 把用户加入 docker组
usermod -aG docker k8s
### 切换k8s用户
su k8s
### 运行
minikube start

### 文档链接
https://minikube.sigs.k8s.io/docs/start/