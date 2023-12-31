# Docker手册
# 修改镜像地址
```
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://t5t8q6wn.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```
# 容器信息
### 查看docker容器版本
```
docker version
```
### 查看docker容器信息
```
docker info
```
### 查看docker容器帮助
```
docker --help
```
# 镜像操作
## 镜像查看
### 列出本地images
```
docker images
```
### 含中间映像层
```
docker images -a
```
## 镜像搜索
### 搜索仓库MySQL镜像
```
docker search mysql
```
### --filter=stars=600：只显示 starts>=600 的镜像
```
docker search --filter=stars=600 mysql
```
### --no-trunc 显示镜像完整 DESCRIPTION 描述
```
docker search --no-trunc mysql
```
### --automated ：只列出 AUTOMATED=OK 的镜像
```
docker search  --automated mysql
```
## 镜像下载
### 下载Redis官方最新镜像，相当于：docker pull redis:latest
```
docker pull redis
```
### 下载仓库所有Redis镜像
```
docker pull -a redis
```
### 下载私人仓库镜像
```
docker pull bitnami/redis
```
## 镜像删除
### 单个镜像删除，相当于：docker rmi redis:latest
```
docker rmi redis
```
### 强制删除(针对基于镜像有运行的容器进程)
```
docker rmi -f redis
```
### 多个镜像删除，不同镜像间以空格间隔
```
docker rmi -f redis tomcat nginx
```
### 删除本地全部镜像
```
docker rmi -f $(docker images -q)
```
## 镜像构建
### （1）编写dockerfile
```
cd /docker/dockerfile
```
###（2）构建docker镜像
```
docker build -f /docker/dockerfile/mycentos -t mycentos:1.1
```
## 镜像导出
### docker save
```
docker save -o /keycloak.tar jboss/keycloak
```
## 镜像导入
### docker load
```
docker load -i /keycloak.tar 
```
# 容器操作
## 容器启动
### 新建并启动容器，参数：-i  以交互模式运行容器；-t  为容器重新分配一个伪输入终端；--name  为容器指定一个名称
```
docker run -i -t --name mycentos
```
### 后台启动容器，参数：-d  已守护方式启动容器
```
docker run -d mycentos
```
### 启动一个或多个已经被停止的容器
```
docker start redis
```
### 重启容器
```
docker restart redis
```
## 容器进程
### top支持 ps 命令参数，格式：docker top [OPTIONS] CONTAINER [ps OPTIONS]
### 列出redis容器中运行进程
```
docker top redis
```
### 查看所有运行容器的进程信息
```
for i in  `docker ps |grep Up|awk '{print $1}'`;do echo \ &&docker top 
$i; done
```
## 容器日志
### 查看redis容器日志，默认参数
```
docker logs rabbitmq
```
### 查看redis容器日志，参数：-f  跟踪日志输出；-t   显示时间戳；--tail  仅列出最新N条容器日志；
```
docker logs -f -t --tail=20 redis
```
### 查看容器redis从2019年05月21日后的最新10条日志。
```
docker logs --since="2019-05-21" --tail=10 redis
```
## 容器的进入与退出
### 使用run方式在创建时进入
```
docker run -it centos /bin/bash
```
### 关闭容器并退出
```
exit
```
### 仅退出容器，不关闭
```
快捷键：Ctrl + P + Q
```
### 直接进入centos 容器启动命令的终端，不会启动新进程，多个attach连接共享容器屏幕，参数：--sig-proxy=false  确保CTRL-D或CTRL-C不会关闭容器
```
docker attach --sig-proxy=false centos 
```
### 在 centos 容器中打开新的交互模式终端，可以启动新进程，参数：-i  即使没有附加也保持STDIN 打开；-t  分配一个伪终端
```
docker exec -i -t  centos /bin/bash
```
### 以交互模式在容器中执行命令，结果返回到当前终端屏幕
```
docker exec -i -t centos ls -l /tmp
```
### 以分离模式在容器中执行命令，程序后台运行，结果不会反馈到当前终端
```
docker exec -d centos  touch cache.txt 
```
## 查看容器
### 查看正在运行的容器
```
docker ps
```
### 查看正在运行的容器的ID
```
docker ps -q
```
### 查看正在运行+历史运行过的容器
```
docker ps -a
```
### 显示运行容器总文件大小
```
docker ps -s
```
### 显示最近创建容器
```
docker ps -l
```
### 显示最近创建的3个容器
```
docker ps -n 3
```
### 不截断输出
```
docker ps --no-trunc 
```
### 获取镜像redis的元信息
```
docker inspect redis
```
### 获取正在运行的容器redis的 IP
```
docker inspect --format='{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' redis
```
## 容器的删除和停止
### 停止一个运行中的容器
```
docker stop redis
```
### 杀掉一个运行中的容器
```
docker kill redis
```
### 删除一个已停止的容器
```
docker rm redis
```
### 删除一个运行中的容器
```
docker rm -f redis
```
### 删除多个容器
```
docker rm -f $(docker ps -a -q)
docker ps -a -q | xargs docker rm
```
### -l 移除容器间的网络连接，连接名为 db
```
docker rm -l db 
```
### -v 删除容器，并删除容器挂载的数据卷
```
docker rm -v redis
```
## 生成镜像
### 基于当前redis容器创建一个新的镜像；参数：-a 提交的镜像作者；-c 使用Dockerfile指令来创建镜像；-m :提交时的说明文字；-p :在commit时，将容器暂停
```
docker commit -a="DeepInThought" -m="my redis" [redis容器ID]  myredis:v1.1
```
## 容器与主机间的数据拷贝
### 将rabbitmq容器中的文件copy至本地路径
```
docker cp rabbitmq:/[container_path] [local_path]
```
### 将主机文件copy至rabbitmq容器
```
docker cp [local_path] rabbitmq:/[container_path]/
```
### 将主机文件copy至rabbitmq容器，目录重命名为[container_path]（注意与非重命名copy的区别）
```
docker cp [local_path] rabbitmq:/[container_path]
```
## 容器的导出
### export
```
docker export -o nginx-test.tar nginx-test
```
## 容器的导入
### import
```
docker import nginx-test.tar nginx:imp
```