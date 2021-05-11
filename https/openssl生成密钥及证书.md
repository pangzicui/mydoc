## 生成CA的私钥

```
openssl genrsa -des3 -out d:/https/ssl/cakey.pem 2048
```

## 生成CA的证书请求
```
openssl req -new -key d:/https/ssl/cakey.pem -out d:/https/ssl/ca.csr -subj "/C=CN/ST=myprovince/L=mycity/O=myorganization/OU=mygroup/CN=myname"
```

## 生成CA的证书
```
openssl x509 -req -days 3650 -sha256 -extensions v3_ca -signkey d:/https/ssl/cakey.pem -in d:/https/ssl/ca.csr -out d:/https/ssl/ca.cer
```

## 生成server端的私钥
```
openssl genrsa -des3 -out d:/https/ssl/server-key.pem 2048
```

## 生成server的证书请求
```
openssl req -new -key d:/https/ssl/server-key.pem -out d:/https/ssl/server.csr -subj "/C=CN/ST=myprovince/L=mycity/O=myorganization/OU=mygroup/CN=client1"
```

## 生成server的证书
```
openssl x509 -req -days 365 -sha256 -extensions v3_req -CA d:/https/ssl/ca.cer -CAkey d:/https/ssl/cakey.pem -CAcreateserial -in d:/https/ssl/server.csr -out d:/https/ssl/server.cer
```

## 生成客户端的私钥
```
openssl genrsa -des3 -out d:/https/ssl/client-key.pem 2048
```

## 生成客户端的证书请求
```
openssl req -new -key d:/https/ssl/client-key.pem -out d:/https/ssl/client.csr -subj "/C=CN/ST=myprovince/L=mycity/O=myorganization/OU=mygroup/CN=postgres"
```

## 生成客户端的证书
```
openssl x509 -req -days 365 -sha256 -extensions v3_req -CA d:/https/ssl/ca.cer -CAkey d:/https/ssl/cakey.pem -CAserial d:/https/ssl/ca.srl -in d:/https/ssl/client.csr -out d:/https/ssl/client.cer
```

## 将客户端的私钥和证书打包成keystore
```
openssl pkcs12 -export -clcerts -name myclient -inkey d:/https/ssl/client-key.pem -in d:/https/ssl/client.cer -out d:/https/ssl/client.keystore
```

## 将服务端的私钥和证书打包成keystore
```
openssl pkcs12 -export -clcerts -name myserver -inkey d:/https/ssl/server-key.pem -in d:/https/ssl/server.cer -out d:/https/ssl/server.keystore
```

## 将CA的证书加入可信列表
```
keytool -importcert -trustcacerts -alias www.mydomain.com -file d:/https/ssl/ca.cer -keystore d:/https/ssl/ca-trust.keystore
```

## p12转bks
```
keytool -importkeystore -srckeystore d:/https/ssl/client.keystore -srcstoretype pkcs12 -destkeystore d:/https/ssl/client.bks -deststoretype bks -provider org.bouncycastle.jce.provider.BouncyCastleProvider
```

## 生成bks的可信列表
```
keytool -import -alias ca -file D:/https/ssl/ca.cer -keystore D:/https/ssl/trust.bks -storetype BKS -provider org.bouncycastle.jce.provider.BouncyCastleProvider
```