#spring boot 搭建https单向认证
##生成密钥库
```
keytool -genkeypair -alias test -keypass 123456 -keyalg RSA -keysize 1024 -validity 365 -keystore d:/https/test.keystore -storepass 123456
```
将密钥库拷贝到项目resource目录下
##配置application.yml，开启ssl支持
```
server:
  ssl:
    enabled: true
    key-store: classpath:test.keystore
    key-store-type: JKS
    key-password: 123456
    key-store-password: 123456

```
#双向认证
##生成客户端证书
```
keytool -genkeypair -alias client -keypass 123456 -keyalg RSA -keysize 1024 -validity 365 -keystore d:/https/client.keystore -storepass 123456
```
##生成服务器证书
```
keytool -genkeypair -alias server -keypass 123456 -keyalg RSA -keysize 1024 -validity 365 -keystore d:/https/server.keystore -storepass 123456
```
##导出客户端公钥
```
keytool -keystore d:/https/client.keystore -export -alias client -file d:/https/client.cer
```
##导出服务器公钥
```
keytool -keystore d:/https/server.keystore -export -alias server -file d:/https/server.cer
```
##生成证书信任库
```
keytool -genkeypair -alias trust -keypass 123456 -keyalg RSA -keysize 1024 -validity 365 -keystore d:/https/trust.keystore -storepass 123456
```
##把公钥添加到信任证书库中

```
keytool -import -v -file d:/https/client.cer -keystore trust.keystore
```
##配置application.yml
```
server:
  ssl:
    enabled: true
    key-store: classpath:server.keystore
    key-store-type: JKS
    key-password: 123456
    key-store-password: 123456
    # trustStore信任库，存放了服务端信任的客户端证书的公钥文件
    protocol: TLS
    client-auth: need
    trust-store: classpath:trust.keystore
    trust-store-password: 123456
    trust-store-type: JKS
    trust-store-provider: SUN
```