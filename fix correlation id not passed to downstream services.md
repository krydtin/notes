### Correlation id 
Fix trace ID and span ID are not passed to downstream services when making API calls using RestTemplate.


#### Add the library
**pom.xml**
add `brave-instrumentation-spring-web`
```xml
<dependency>
    <groupId>io.zipkin.brave</groupId>
    <artifactId>brave-instrumentation-spring-web</artifactId>
</dependency>
```

#### Add interceptor 

Import the `brave` package instead of the `sleuth` package.
```java
import brave.Tracing;
import brave.http.HttpTracing;
import brave.spring.web.TracingClientHttpRequestInterceptor;
```

Add `httpTracing`
```java
@Bean
public HttpTracing create(Tracing tracing) {
    return HttpTracing
            .newBuilder(tracing)
            .build();
}
```

Add `TracingClientHttpRequestInterceptor`
```java
@Bean
@Primary
public RestTemplate restTemplate(HttpTracing httpTracing) {

    final PoolingHttpClientConnectionManager connectionManager = new PoolingHttpClientConnectionManager();
    connectionManager.setDefaultConnectionConfig(
            ConnectionConfig.custom()
                    .setConnectTimeout(Timeout.ofSeconds(20))
                    .setSocketTimeout(Timeout.ofSeconds(60))
                    .build()
    );

    final CloseableHttpClient httpClient = HttpClientBuilder.create()
            .setConnectionManager(connectionManager)
            .build();

    return new RestTemplateBuilder()
            .requestFactory(() -> new HttpComponentsClientHttpRequestFactory(httpClient))
            .messageConverters(new StringHttpMessageConverter(), new MappingJackson2HttpMessageConverter())
            .interceptors(List.of(
                    TracingClientHttpRequestInterceptor.create(httpTracing)
            ))
            .errorHandler(new RestTemplateErrorHandler())
            .build();
}
```
