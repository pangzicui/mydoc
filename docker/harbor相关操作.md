# Harbor操作
###cert目录
```
/etc/docker/
```
### 登录harbor
```
docker login 10.32.233.112
```
### 构建镜像
```
docker build -t 10.32.233.112/library/keycloak:11.0.2.1 .
```
### 推送镜像
```
docker push 10.32.233.112/library/keycloak:11.0.2.1
```
