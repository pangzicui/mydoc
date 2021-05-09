## 创建密钥
```
ssh-keygen -t rsa
```
## 拷贝秘钥
```
ssh-copy-id -i /root/.ssh/id_rsa.pub root@192.168.1.110 -p 22
```
## 查看连接日志
```
tail /var/log/secure -n 20
```
## 免密登录后仍然需要密码
```
vim /etc/ssh/sshd_config
```
```
#允许root认证登录
PermitRootLogin yes

#允许密钥认证
RSAAuthentication yes
PubkeyAuthentication yes

#默认公钥存放的位置
AuthorizedKeysFile  .ssh/authorized_key
```

