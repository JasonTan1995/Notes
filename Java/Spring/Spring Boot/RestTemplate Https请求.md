## RestTemplate 发送Https请求失败的问题

### 1. 前言
***
日常开发中很多时候都会调用到第三方的接口或者公司里别的项目的接口,现在大多数都是使用`httpClient`还有就是`Spring`提供的`RestTemplate`,这两个都适用于调用第三方的接口.
以下记录一下在工作中使用到`RestTemplate`遇到的一些问题.

### 2. `RestTemplate`

`Spring Boot`官方文档的介绍:

[![ZnQhsU.png](https://s2.ax1x.com/2019/06/27/ZnQhsU.png)](https://imgchr.com/i/ZnQhsU)

大概意思就是说: 当需要远程调用别的接口的时候,可以使用`Spring Boot`提供的`RestTemplate`,而`RestTemplate`是需要自己去制定的,`Spring Boot`中没有提供这个bean.

- 使用方法:需要先配置,装载bean,然后再使用.然后对应的请求方法会有(`getForObject,postForObject,delet(),put()`)

```java
    @Bean
    public RestTemplate getRestTemplate(RestTemplateBuilder restTemplateBuilder) {
        RestTemplate restTemplate = restTemplateBuilder.build();
        return restTemplate;
    }
    @Service
    public class TestService {
      
      private final RestTemplate restTemplate;
      
      public TestService(RestTemplate restTemplate) {
        this.restTemplate = restTemplate;
      }
      
      public User restCallUser(String name) {
          return this.restTemplate.getForObject("/{name}/details", User.class, name);
      }
```
与`httpClient`对比,`restTemplate`使用起来很简单,上手也很快.但以上的情况都是在HTTP请求下的,如果是HTTPS请求的话那就是另外一种情况了.

会出现以下的错误:

[![ZnGoCj.png](https://s2.ax1x.com/2019/06/27/ZnGoCj.png)](https://imgchr.com/i/ZnGoCj)

这个错误出现就是因为访问HTTPS的接口没有进行证书验证.听到证书验证就懵了,所以究竟要怎么解决呢?

#### 2.1 `RestTemplate`跳过证书验证

- 首先看看`RestTemplate`的构造方法是支持传入一个工厂对象的

```java
public RestTemplate(ClientHttpRequestFactory requestFactory) {
        this();
        this.setRequestFactory(requestFactory);
    }
```
- 而这个工厂一般是用于设置配置参数,譬如`ReadTimeout`,`ConnectTimeout`,`Proxy`...等等.考虑到篇幅的问题,只贴出小部分代码,具体代码可以自己详细研究

```java
public class SimpleClientHttpRequestFactory implements ClientHttpRequestFactory, AsyncClientHttpRequestFactory {
    private static final int DEFAULT_CHUNK_SIZE = 4096;
    @Nullable
    private Proxy proxy;
    private boolean bufferRequestBody = true;
    private int chunkSize = 4096;
    private int connectTimeout = -1;
    private int readTimeout = -1;
    private boolean outputStreaming = true;
    @Nullable
    private AsyncListenableTaskExecutor taskExecutor;

    public SimpleClientHttpRequestFactory() {
    }
```

- 因此,我们可以根据该类来重写一个新的类用于跳过证书验证
```java
public class SkipSslVerificationHttpRequestFactory extends SimpleClientHttpRequestFactory {

    @Override
    protected void prepareConnection(HttpURLConnection connection, String httpMethod)
            throws IOException {
        if (connection instanceof HttpsURLConnection) {
            prepareHttpsConnection((HttpsURLConnection) connection);
        }
        super.prepareConnection(connection, httpMethod);
    }

    private void prepareHttpsConnection(HttpsURLConnection connection) {
        connection.setHostnameVerifier(new SkipHostnameVerifier());
        try {
            connection.setSSLSocketFactory(createSslSocketFactory());
        } catch (Exception ex) {
            // Ignore
        }
    }

    private SSLSocketFactory createSslSocketFactory() throws Exception {
        SSLContext context = SSLContext.getInstance("TLS");
        context.init(null, new TrustManager[]{new SkipX509TrustManager()},
                new SecureRandom());
        return context.getSocketFactory();
    }

    private class SkipHostnameVerifier implements HostnameVerifier {

        @Override
        public boolean verify(String s, SSLSession sslSession) {
            return true;
        }

    }

    private static class SkipX509TrustManager implements X509TrustManager {

        @Override
        public void checkClientTrusted(X509Certificate[] chain, String authType) throws CertificateException {

        }

        @Override
        public X509Certificate[] getAcceptedIssuers() {
            return new X509Certificate[0];
        }

        @Override
        public void checkServerTrusted(X509Certificate[] chain, String authType) {
        }

    }
}
```

- 然后就可以正常请求了,不管是HTTP还是HTTPS都能正常请求.


#### 参考资料:

[Calling REST Services with RestTemplate](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-resttemplate.html#boot-features-resttemplate-customization)

[Springboot RestTemplate如何配置http和https](https://blog.csdn.net/weixin_41235296/article/details/86645363)








