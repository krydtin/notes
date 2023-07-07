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
public RestTemplate restTemplate(HttpTracing httpTracing) {
    return new RestTemplateBuilder()           
            .interceptors(TracingClientHttpRequestInterceptor.create(httpTracing))
        .build();
}
```
