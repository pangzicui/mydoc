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
keytool -genkeypair -alias client -keypass 123456 -keyalg RSA -storetype PKCS12 -keysize 1024 -validity 365 -keystore d:/https/client.p12 -storepass 123456
```
##生成服务器证书

```
keytool -genkeypair -alias server -keypass 123456 -keyalg RSA -storetype PKCS12 -keysize 1024 -validity 365 -keystore d:/https/server.p12 -storepass 123456
```
##导出客户端公钥

```
keytool -keystore d:/https/client.p12 -export -alias client -file d:/https/client.cer
```
##导出服务器公钥

```
keytool -keystore d:/https/server.p12 -export -alias server -file d:/https/server.cer
```
##把客户端公钥添加到服务端信任证书库中

```
keytool -import -alias client -v -file d:/https/client.cer -keystore d:/https/server.p12 -storepass 123456
```
##把服务端公钥添加到客户端信任证书库中

```
keytool -import -alias server -v -file d:/https/server.cer -keystore d:/https/client.p12 -storepass 123456
```
##将服务端的密钥导入JDK密钥库

```
keytool -import -alias server -file d:/https/server.cer -keystore "C:\Program Files\Java\jdk1.8.0_251\jre\lib\security\cacerts" –v -storepass changeit
```
##将客户端的密钥导入JDK密钥库

```
keytool -import -alias client -file d:/https/client.cer -keystore "C:\Program Files\Java\jdk1.8.0_251\jre\lib\security\cacerts" –v -storepass changeit
```
##如果需要删除JDK密钥库中的密钥的话，执行以下操作

```
keytool -delete -alias client -keystore "C:\Program Files\Java\jdk1.8.0_251\jre\lib\security\cacerts" -storepass changeit

keytool -delete -alias server -keystore "C:\Program Files\Java\jdk1.8.0_251\jre\lib\security\cacerts" -storepass changeit
```
##生成BKS密钥库(安卓)

```
keytool -genkey -alias client -keypass 123456 -keyalg RSA -keysize 1024 -validity 365 -keystore d:/https/client.bks -storepass 123456 -storetype BKS -provider org.bouncycastle.jce.provider.BouncyCastleProvider
```
##BKS导出crt 公钥
```
keytool -keystore d:/https/client.bks -export -alias client -file d:/https/client.crt -storetype BKS -provider org.bouncycastle.jce.provider.BouncyCastleProvider
```
## Cer转pem
```
openssl x509 -inform der -in d:/https/client.cer -out d:/https/client.pem
```
##JKS转PEM
```
openssl pkcs12 -nodes -in d:/https/client.p12 -out d:/https/client.pem
```
##p12转JKS
```
keytool -importkeystore -srckeystore d:/https/client.p12 -srcstoretype PKCS12 -deststoretype JKS -destkeystore d:/https/client.jks
```
##JKS转CERT
```
keytool -exportcert -alias client -file d:/https/client.cert -keystore d:/https/client.jks
```
##配置application.yml

```
server:
  ssl:
    enabled: true
    key-store: classpath:server.p12
    key-store-type: JKS
    key-password: 123456
    key-store-password: 123456
    # trustStore信任库，存放了服务端信任的客户端证书的公钥文件
    protocol: TLS
    client-auth: need
    trust-store: classpath:server.p12
    trust-store-password: 123456
    trust-store-type: JKS
    trust-store-provider: SUN
```