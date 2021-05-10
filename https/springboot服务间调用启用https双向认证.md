# SpringBoot服务间调用启用HTTPS双向认证

## 网关配置
### 配置网关server证书
```
ribbon:
  IsSecure: true
```
配置网关认可的证书信息，只有证书配对的请求才允许进入网关。
```
server:
  ssl:
    enabled: true
    key-store: classpath:ssl/server.keystore
    key-store-type: PKCS12
    key-password: 123456
    key-store-password: 123456
    key-alias: myserver
    client-auth: need
    trust-store: classpath:ssl/ca-trust.keystore
    trust-store-password: 123456
    trust-store-type: JKS
    trust-store-provider: SUN
```
### 配置网关转发给微服务的证书
配置网关转发请求给微服务时携带的证书
```
spring:
  cloud:
    gateway:
      httpclient:
        ssl:
          useInsecureTrustManager: true
          key-store: classpath:ssl/client.keystore
          key-store-type: PKCS12
          key-store-password: 123456
```
## 注册中心开启ssl
```
server:
  ssl:
    enabled: true
    key-store: classpath:ssl/server.keystore
    key-store-type: PKCS12
    key-password: 123456
    key-store-password: 123456
    key-alias: myserver
    client-auth: need
    trust-store: classpath:ssl/ca-trust.keystore
    trust-store-password: 123456
    trust-store-type: JKS
    trust-store-provider: SUN
```
```
eureka.client.serviceUrl.defaultZone=https://${eureka.instance.hostname}:${server.port}/eureka/
```
## 微服务开启ssl认证
```
ribbon:
  IsSecure: true
```
```
server:
  ssl:
    enabled: true
    key-store: classpath:ssl/server.keystore
    key-store-type: PKCS12
    key-password: 123456
    key-store-password: 123456
    key-alias: myserver
    client-auth: need
    trust-store: classpath:ssl/ca-trust.keystore
    trust-store-password: 123456
    trust-store-type: JKS
    trust-store-provider: SUN
```
## Feign调用时携带SSL配置
新增配置文件，重写Feign的Client
```
import com.netflix.discovery.DiscoveryClient;
import com.netflix.discovery.shared.transport.jersey.EurekaJerseyClientImpl;
import feign.Client;
import lombok.extern.slf4j.Slf4j;
import org.apache.http.conn.ssl.NoopHostnameVerifier;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.autoconfigure.condition.ConditionalOnProperty;
import org.springframework.cloud.netflix.ribbon.SpringClientFactory;
import org.springframework.cloud.openfeign.ribbon.CachingSpringLoadBalancerFactory;
import org.springframework.cloud.openfeign.ribbon.LoadBalancerFeignClient;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.util.ResourceUtils;
import org.springframework.util.StringUtils;

import javax.net.ssl.KeyManagerFactory;
import javax.net.ssl.SSLContext;
import javax.net.ssl.TrustManagerFactory;
import javax.net.ssl.X509TrustManager;
import java.io.FileInputStream;
import java.io.IOException;
import java.io.InputStream;
import java.security.KeyStore;
import java.security.KeyStoreException;
import java.security.NoSuchAlgorithmException;
import java.security.cert.CertificateException;
import java.security.cert.X509Certificate;
import java.util.UUID;

/**
 * @author HongCui
 */
@Configuration
@Slf4j
public class HttpsConfiguration {

    @Value("${server.ssl.key-store-password:}")
    private String keyStorePsw;
    @Value("${server.ssl.trust-store-password:}")
    private String trustStorePsw;
    @Value("${server.ssl.key-store:}")
    private String keyStorePath;
    @Value("${server.ssl.trust-store:}")
    private String trustStorePath;
    @Value("${server.ssl.enabled:false}")
    private Boolean isEnableSSL;
    @Value("${server.ssl.key-store-type:}")
    private String keyStoreType;
    @Value("${server.ssl.trust-store-type:}")
    private String trustStoreType;

    /**
     * Feign调用设置SSLContext
     *
     * @param cachingFactory
     * @param clientFactory
     * @return
     * @throws NoSuchAlgorithmException
     */
    @Bean
    @ConditionalOnProperty(name = "server.ssl.enabled", havingValue = "true")
    public Client feignClient(CachingSpringLoadBalancerFactory cachingFactory,
                              SpringClientFactory clientFactory) throws NoSuchAlgorithmException {
        try {
            SSLContext sslContext = getSSLContext();
            return new LoadBalancerFeignClient(new Client.Default(sslContext.getSocketFactory(), new NoopHostnameVerifier()), cachingFactory, clientFactory);
        } catch (Exception e) {
            log.error(e.getMessage(), e);
        }
        return new LoadBalancerFeignClient(new Client.Default(null, null), cachingFactory, clientFactory);
    }

    /**
     * Eureka注册参数设置SSLContext
     *
     * @return
     */
    @Bean
    @ConditionalOnProperty(name = "server.ssl.enabled", havingValue = "true")
    public DiscoveryClient.DiscoveryClientOptionalArgs discoveryClientOptionalArgs() {
        try {
            SSLContext sslContext = getSSLContext();
            EurekaJerseyClientImpl.EurekaJerseyClientBuilder builder = new EurekaJerseyClientImpl.EurekaJerseyClientBuilder();
            builder.withCustomSSL(sslContext);
            builder.withClientName(UUID.randomUUID().toString());
            builder.withMaxTotalConnections(10);
            builder.withMaxConnectionsPerHost(10);
            DiscoveryClient.DiscoveryClientOptionalArgs args = new DiscoveryClient.DiscoveryClientOptionalArgs();
            args.setEurekaJerseyClient(builder.build());
            return args;
        } catch (Exception e) {
            log.error(e.getMessage(), e);
        }
        return null;
    }

    /**
     * 获取SSLContext
     *
     * @return
     * @throws Exception
     */
    private SSLContext getSSLContext() throws Exception {
        //加载客户端的keystore
        KeyStore keyStore = loadKeyStore(keyStoreType, keyStorePath, keyStorePsw);
        KeyManagerFactory keyManagerFactory = KeyManagerFactory.getInstance(KeyManagerFactory.getDefaultAlgorithm());
        keyManagerFactory.init(keyStore, keyStorePsw.toCharArray());
        //加载信任证书的keystore
        KeyStore trustStore = loadKeyStore(trustStoreType, trustStorePath, trustStorePsw);
        TrustManagerFactory trustManagerFactory = TrustManagerFactory.getInstance(TrustManagerFactory.getDefaultAlgorithm());
        trustManagerFactory.init(trustStore);
        //创建SSLContext
        SSLContext sslContext = SSLContext.getInstance("TLS");
        //信任所有，单向认证时可以无脑信任对方的证书
//                sslContext.init(keyManagerFactory.getKeyManagers(), new TrustManager[]{new UnSafeTrustManager()}, null);
        sslContext.init(keyManagerFactory.getKeyManagers(), trustManagerFactory.getTrustManagers(), null);
        return sslContext;
    }

    /**
     * 加载keyStore
     *
     * @param keyStoreType
     * @param keyStorePath
     * @param keyStorePsw
     * @return
     * @throws CertificateException
     * @throws NoSuchAlgorithmException
     * @throws IOException
     * @throws KeyStoreException
     */
    public KeyStore loadKeyStore(String keyStoreType, String keyStorePath, String keyStorePsw) throws CertificateException, NoSuchAlgorithmException, IOException, KeyStoreException {
        if (!StringUtils.isEmpty(keyStorePath)) {
            KeyStore keyStore = KeyStore.getInstance(keyStoreType);
            InputStream keyStoreInputStream = new FileInputStream(ResourceUtils.getFile(keyStorePath));
            keyStore.load(keyStoreInputStream, keyStorePsw.toCharArray());
            keyStoreInputStream.close();
            return keyStore;
        }
        KeyStore keyStore = KeyStore.getInstance(KeyStore.getDefaultType());
        keyStore.load(null, null);
        return keyStore;
    }

    private static class UnSafeTrustManager implements X509TrustManager {

        @Override
        public void checkClientTrusted(X509Certificate[] x509Certificates, String s) throws CertificateException {

        }

        @Override
        public void checkServerTrusted(X509Certificate[] x509Certificates, String s) throws CertificateException {

        }

        @Override
        public X509Certificate[] getAcceptedIssuers() {
            return new X509Certificate[]{};
        }
    }

}
```