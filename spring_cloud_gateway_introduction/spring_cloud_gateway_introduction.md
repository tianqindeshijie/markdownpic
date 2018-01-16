## Spring Cloud Gateway学习
### 1. 如何使用Spring cloud Gateway
添加如下依赖在pom.xml文件中则可以使用Spring Cloud Gateway

```
    <dependency>
		<groupId>org.springframework.cloud</groupId>
		<artifactId>spring-cloud-starter-gateway</artifactId>
	</dependency>
```
版本号可以跟随主项目的版本号即可。

如果你把Spring cloud Gateway的添加到pom.xml中但是又不需要它启动，可以在配置文件中配置`spring.cloud.gateway.enabled=false`。
    
### 2. 术语
- Route:路由是网关的基本构建块。它由一个ID、一个目的URI、一组谓词和一组过滤器定义。如果聚合谓词为真，则路由匹配。  
- Predicate:这是一个java8函数谓词。输入类型是Spring框架`ServerWebExchange`。这允许开发人员从HTTP请求（如头文件或参数）匹配任何东西。
- Filter: 这些例子Spring框架`Gatewayfilter`构造与特定工厂。在这里，请求和响应可以在发送下游请求之前或之后进行修改。
### 3. 如何工作
![image](https://raw.githubusercontent.com/tianqindeshijie/markdownpic/master/spring_cloud_gateway_introduction/spring_cloud_gateway_diagram.png)

客户端向Spring cloud Gateway发出请求。如果网关处理程序映射确定请求与路由匹配，则将其发送到网关Web处理程序。该处理程序通过该请求的过滤器链来运行。过滤器被虚线分隔的原因是，过滤器可以在代理之前或之后执行逻辑。流程是前置过滤器执行，代理执行，后置过滤器执行。
### 4.路由谓词
Spring Cloud Gateway使用Spring WebFlux的`HandlerMapping`一部分功能来做路由匹配。 Spring Cloud Gateway包含许多内置路由谓词工厂. 所有的这些谓词都匹配不同情况的Http请求。多个路由谓词可以通过`and`来联合使用。 
#### 4.1 `After`路由谓词
`After`路由谓词有一个时间参数。该谓词匹配当前请求的日期后发生的。

```
spring:
  cloud:
    gateway:
      routes:
      # =====================================
      - id: after_route
        uri: http://example.org
        predicates:
        - After=2017-01-20T17:42:47.789-07:00[America/Denver]
```
上面的例子表示：会匹配2017年1月20日下午5点42分（美国/丹佛）以后的请求。
#### 4.2 `Before` 路由谓词

`Before`路由谓词有一个时间参数。该谓词匹配当前请求的日期之前发生的。

```
spring:
  cloud:
    gateway:
      routes:
      # =====================================
      - id: before_route
        uri: http://example.org
        predicates:
        - Before=2017-01-20T17:42:47.789-07:00[America/Denver]
```
上面的例子表示：会匹配2017年1月20日下午5点42分（丹佛）之前的请求。
#### 4.3 `Between` 路由谓词
`Between`路由谓词有两个时间参数，匹配在两个时间参数之间的请求，当然时间参数1必须早于时间参数2.

```
spring:
  cloud:
    gateway:
      routes:
      # =====================================
      - id: between_route
        uri: http://example.org
        predicates:
        - Betweeen=2017-01-20T17:42:47.789-07:00[America/Denver], 2017-01-21T17:42:47.789-07:00[America/Denver]
```
上面的例子表示：匹配所有2017年1月20日下午5点42分和2017年1月21日下午5点42分之间的请求。这可能对维护窗口有用。
#### 4.4 `Cookie` 路由谓词
`Cookie`路由谓词有两个参数，cookie的名称和一个正则表达式。此谓词匹配具有给定名称和值与正则表达式匹配的cookie。

```
spring:
  cloud:
    gateway:
      routes:
      # =====================================
      - id: cookie_route
        uri: http://example.org
        predicates:
        - Cookie=chocolate, ch.p
```
上面的例子表示匹配cookie名称为`chocolate `,它的值匹配正则表达式`ch.p`.
#### 4.5 `Header` 路由谓词
`Header`路由谓词有两个参数，Header的名称和一个正则表达式。此谓词匹配具有给定名称和值与正则表达式匹配的Header。

```
spring:
  cloud:
    gateway:
      routes:
      # =====================================
      - id: header_route
        uri: http://example.org
        predicates:
        - Header=X-Request-Id, \d+
```
上面的例子表示：匹配header的名字为`X-Request-Id`,值匹配正则表达式`\d+`（有价值的一个或多个数字）。
#### 4.6 `Host` 路由谓词
Host路由谓词有一个参数：主机名模式。模式是一种Ant样式的模式`.`作为分离器。此谓词匹配与模式匹配的主机名称。
```
spring:
  cloud:
    gateway:
      routes:
      - id: host_route
        uri: http://example.org
        predicates:
        - Host=**.somehost.org
```
上面的例子匹配像`www.somehost.org` or `beta.somehost.org`的主机名称。


#### 4.7 `Method` Route Predicate Factory
`Method` 有一个路由谓词匹配http方法。
```
spring:
  cloud:
    gateway:
      routes:
      - id: method_route
        uri: http://example.org
        predicates:
        - Method=GET
```

上面的例子表示只匹配`GET`方式类型的http请求。


#### 4.8 `Path` 路由谓词
`Path`路由谓词有一个参数`PathMatcher`

```
spring:
  cloud:
    gateway:
      routes:
      - id: host_route
        uri: http://example.org
        predicates:
        - Path=/foo/{segment}
```
上面的例子中route匹配路由为 `/foo/1`或者`/foo/bar`的请求

这个谓词提取URI模板变量 (像上面例子中的 `segment`)作为一个名称和值的映射，并且把它放在`ServerWebExchange.getAttributes()` 中，key为 `PathRoutePredicate.URL_PREDICATE_VARS_ATTR`.这些值可以在 Those values  <<gateway-route-filters,GatewayFilter Factories>>中使用。
#### 4.9 `Query`路由谓词
`Query` 路由谓词有两个参数：一个必填的`param`和一个选填的`regexp`.

```
spring:
  cloud:
    gateway:
      routes:
      - id: query_route
        uri: http://example.org
        predicates:
        - Query=baz
```
上面的例子匹配包含`baz`的查询参数。

```
spring:
  cloud:
    gateway:
      routes:
      # =====================================
      - id: query_route
        uri: http://example.org
        predicates:
        - Query=foo, ba.
```
上面的例子表示包含查询参数`foo`,并且它的值匹配`ba.`，比如`bar`,`baz`。
#### 4.10 `RemoteAddr` 路由谓词
`RemoteAddr`路由谓词需要一个ip列表，例如192.168.0.1/16 (192.168.0.1是一个IP 地址 而16是一个子网掩码）。
```
spring:
  cloud:
    gateway:
      routes:
      # =====================================
      - id: remoteaddr_route
        uri: http://example.org
        predicates:
        - RemoteAddr=192.168.1.1/24
```
上面的例子需要远端地址匹配才可以访问，比如`192.168.1.10`。
### 5. `GatewayFilter` 工厂
路由过滤器允许修改传入的Http请求或者传出的Http响应。路由过滤器的范围是特定的路线。春云网关包括许多内置的gatewayfilter工厂。路由过滤器覆盖的范围是特别的路由。 Spring Cloud Gateway 包含很多内置的路由过滤器。
#### 5.1 `AddRequestHeader` 网关过滤器
`AddRequestHeader`网关过滤器有两个参数：名称和参数值

```
spring:
  cloud:
    gateway:
      routes:
      # =====================================
      - id: add_request_header_route
        uri: http://example.org
        filters:
        - AddRequestHeader=X-Request-Foo, Bar
```
上面的例子表示，将会为所有符合条件的请求添加`X-Request-Foo:Bar`的http头。
#### 5.2 `AddRequestParameter`网关过滤器
`AddRequestParameter`网关过滤器有两个参数：名称和参数值

```
spring:
  cloud:
    gateway:
      routes:
      # =====================================
      - id: add_request_header_route
        uri: http://example.org
        filters:
        - AddResponseHeader=X-Response-Foo, Bar
```
上面的例子表示，将会为所有符合条件的响应添加`X-Request-Foo:Bar`的http头。
#### 5.4 `Hystrix` 网关过滤器
`Hystrix` 网关过滤器有一个名称参数，它表示`HystrixCommand`名称. (更多的选项可能在将来的版本中添加)。

```
spring:
  cloud:
    gateway:
      routes:
      # =====================================
      - id: hytstrix_route
        uri: http://example.org
        filters:
        - Hystrix=myCommandName
```
This wraps the remaining filters in a HystrixCommand with command name myCommandName.
#### 5.5 `PrefixPath`网关过滤器
`PrefixPath` 网关过滤器有一个前缀参数。
```
spring:
  cloud:
    gateway:
      routes:
      # =====================================
      - id: prefixpath_route
        uri: http://example.org
        filters:
        - PrefixPath=/mypath
```
上面的例子表示将为所有满足条件的请求添加一个`/mypath`的前缀。比如一个请求到`/hello`将会被发送到`/mypath/hello`
#### 5.6 `RequestRateLimiter` 网关过滤器
`RequestRateLimiter` 网关过滤器有3个参数: `replenishRate`, `burstCapacity`和`keyResolverName`。

- `replenishRate`表示允许用户每秒钟请求数量

- `burstCapacity` TODO

- `keyResolver` 是一个实现 `KeyResolver`接口的bean. 在配置上通过名字使用`SpEL`表达式.`#{@myKeyResolver}`是一个`SpEL`表达式指向`myKeyResolver`.

**KeyResolver.java**

```
public interface KeyResolver {
	Mono<String> resolve(ServerWebExchange exchange);
}
```
`KeyResolver`接口采取插件式策略来从请求中获取key。在将来的里程碑中将会有一些内置的实现类。

Redis的实现是基于脚本的。. 它需要使用Spring Boot下的`spring-boot-starter-data-redis-reactive`.

```
spring:
  cloud:
    gateway:
      routes:
      # =====================================
      - id: requestratelimiter_route
        uri: http://example.org
        filters:
        - RequestRateLimiter=10, 20, #{@userKeyResolver}
```
**Config.java**

```
@Bean
KeyResolver userKeyResolver() {
    return exchange -> Mono.just(exchange.getRequest().getQueryParams().getFirst("user"));
}
```
上面的例子表示每个用户每秒钟限制10个请求。`KeyResolver`是一个简单的获取请求中参数的实现。
#### 5.7 `RedirectTo` 路由过滤器
`RedirectTo`路由过滤器有两个参数：`status`和`url`. `status`应该是300系列的http码，`url`应该是有效的url。 url将会是http头中`Location`的值。

```
spring:
  cloud:
    gateway:
      routes:
      # =====================================
      - id: prefixpath_route
        uri: http://example.org
        filters:
        - RedirectTo=302, http://acme.org
```
上面的例子中：将会发送一个302的状态在http头部`Location:http://acme.org`用于执行跳转.
#### 5.8 `RemoveNonProxyHeaders` 路由过滤器
`RemoveNonProxyHeaders`路由过滤器将会移除转发请求header中的信息。删除的标头的默认列表来自`IETF`。 

**默认移除的header是:**

- Connection
- Keep-Alive
- Proxy-Authenticate
- Proxy-Authorization
- TE
- Trailer
- Transfer-Encoding
- Upgrade

如果要改变上面的列表，需要设置`spring.cloud.gateway.filter.remove-non-proxy-headers.headers`属性，这个列表中的header都将会被移除。
#### 5.9 `RemoveRequestHeader` 路由过滤器
`RemoveRequestHeader` 有一个参数:需要移除header的名字。

```
spring:
  cloud:
    gateway:
      routes:
      # =====================================
      - id: removerequestheader_route
        uri: http://example.org
        filters:
        - RemoveRequestHeader=X-Request-Foo
```
上面的例子中将移除`X-Request-Foo` header在向下发送的时候。
#### 5.10 `RemoveResponseHeader`路由过滤器
`RemoveResponseHeader`路由过滤器 有一个参数：移除header的名字。

```
spring:
  cloud:
    gateway:
      routes:
      # =====================================
      - id: removeresponseheader_route
        uri: http://example.org
        filters:
        - RemoveResponseHeader=X-Response-Foo
```
上面的例子中：将会移除响应上的`X-Response-Foo`header
#### 5.11 `RewritePath`路由过滤器
`RewritePath` 有两个参数：一个路由正则表达式参数`regexp`和一个`replacement`参数。 使用java正则表达式的一个灵活的方式来重写请求路径。

```
spring:
  cloud:
    gateway:
      routes:
      # =====================================
      - id: rewritepath_route
        uri: http://example.org
        predicates:
        - Path=/foo/**
        filters:
        - RewritePath=/foo/(?<segment>.*), /$\{segment}
```
上面的例子中：对于`/foo/bar`路径,在传递到下面之前将会设置路径为`/bar` 。注意：  `$\` 表示的是 `$`.
#### 5.12  `SecureHeaders`路由过滤器
`SecureHeaders`将会添加一定数量的header在响应的头部,这篇[文章](https://blog.appcanary.com/2017/http-security-headers.html)中推荐了哪些header是安全中常用的。

**下面header将被添加 (包含默认值):**
- X-Xss-Protection:1; mode=block
- Strict-Transport-Security:max-age=631138519
- X-Frame-Options:DENY
- X-Content-Type-Options:nosniff
- Referrer-Policy:no-referrer
- Content-Security-Policy:default-src 'self' https:; font-src 'self' https: data:; img-src 'self' https: data:; object-src 'none'; script-src https:; style-src 'self' https: 'unsafe-inline'
- X-Download-Options:noopen
- X-Permitted-Cross-Domain-Policies:none

要更改默认值在spring.cloud.gateway.filter.secure-headers命名空间设置适当的属性：

**更改属性：**

- xss-protection-header
- strict-transport-security
- frame-options
- content-type-options
- referrer-policy
- content-security-policy
- download-options
- permitted-cross-domain-policies
#### 5.13 `SetPath`路由过滤器
`SetPath`路由过滤器有一个路径模板参数：`template`.它提供了一种简单操作请求路径的方法，通过允许使用路径上的模板片段。这将使用Spring框架中的URI模板。允许多个匹配段。

```
spring:
  cloud:
    gateway:
      routes:
      # =====================================
      - id: setpath_route
        uri: http://example.org
        predicates:
        - Path=/foo/{segment}
        filters:
        - SetPath=/{segment}
```
上面的例子中：一个请求路径为`/foo/bar`,过滤器将在转发到下游之前设置路径为`/bar`.
#### 5.14 `SetResponseHeader`路由过滤器
`SetResponseHeader`路由过滤器有两个参数：`name`和`value`. 

```
spring:
  cloud:
    gateway:
      routes:
      # =====================================
      - id: setresponseheader_route
        uri: http://example.org
        filters:
        - SetResponseHeader=X-Response-Foo, Bar
```
上面的例子中：过滤器将会替换所有在header中名字为`X-Response-Foo`的值为`Bar`. 所以如果下游有一个响应中含有头部信息为`X-Response-Foo:1234`将会被替换为`X-Response-Foo:Bar`.
#### 5.15 `SetStatus` 路由过滤器
`SetStatus`路由过滤器 仅有一个`status`.它必须是一个有效的Spring Http状态码。它可能是整数404或枚举not_found的字符串表示形式。

```
spring:
  cloud:
    gateway:
      routes:
      # =====================================
      - id: setstatusstring_route
        uri: http://example.org
        filters:
        - SetStatus=BAD_REQUEST
      - id: setstatusint_route
        uri: http://example.org
        filters:
        - SetStatus=401
```
上面的例子中：在这两种情况下，响应的HTTP状态将被设置为401。
### 6. 全局过滤器
`GlobalFilter`接口基本和`GatewayFilter`是相同的。只不过这些过滤器特殊一下，可以有条件的用在所有的路由上。 
#### 6.1 Combined Global Filter and GatewayFilter Ordering
todo
#### 6.2 转发路由过滤器

`ForwardRoutingFilter`在交互列表中查找`ServerWebExchangeUtils.GATEWAY_REQUEST_URL_ATTR`属性为`forward` 标识 (ie `forward:///localendpoint`),将使用Spring中的`DispatcherHandler` 去处理这个请求。未被修改的原始url将会被添加到`ServerWebExchangeUtils.GATEWAY_ORIGINAL_REQUEST_URL_ATTR`属性的列表中。

#### 6.3 `LoadBalancerClient`过滤器
`ForwardRoutingFilter`在交互列表中查找`ServerWebExchangeUtils.GATEWAY_REQUEST_URL_ATTR`属性为`lb` 标识 (ie `lb://myservice`),
 它将使用Spring cloud `LoadBalancerClient` 去解析这个名称（`myservice`前面提到过的例子）为时间的主机名称和端口去替换相同的url中的名称。 未被修改的原始url将会被添加到`ServerWebExchangeUtils.GATEWAY_ORIGINAL_REQUEST_URL_ATTR`属性的列表中。
#### 6.4 Netty路由过滤器
`Netty` 路由过滤器运行当`ServerWebExchangeUtils.GATEWAY_REQUEST_URL_ATTR`中有 `http` 或者 `https` 标识.它将使用Netty的`HttpClient`做下游的代理请求。响应将被放入到 `ServerWebExchangeUtils.CLIENT_RESPONSE_ATTR`中 (有一个实验性的`WebClientHttpRoutingFilter`具有相同的作用，但是不是用Netty实现）。
#### 6.5 `Netty` 写响应过滤器

如果有一个Netty`HttpClientResponse`在`ServerWebExchangeUtils.CLIENT_RESPONSE_ATTR`交换列表中`NettyWriteResponseFilter`将会起作用。他在其它过滤器后面运行，它将重新设置返回的响应。(有一个实验性的`WebClientWriteResponseFilter`具有相同的作用，但是不是用Netty实现的)

#### 6.6 `RouteToRequestUrl` 过滤器
如果有一个`Route`对象在`ServerWebExchangeUtils.GATEWAY_ROUTE_ATTR`交互列表中，`RouteToRequestUrlFilter`将会起作用。它基于请求URI创建一个新URI，但使用`Route`对象的URI属性进行更新。It creates a new URI, based off of the request URI, but updated with the URI attribute of the `Route` object.新的URI会替换``ServerWebExchangeUtils.GATEWAY_REQUEST_URL_ATTR`中的交互属性.

#### 6.7 `Websocket`路由过滤器

`Websocket`路由过滤器，如果在 `ServerWebExchangeUtils.GATEWAY_REQUEST_URL_ATTR`列表中的url标识为`ws`或`wss`.它使用Spring Web Socket工具来转发websocket请求到下游。 

### 7. 配置
Spring Cloud Gateway的配置是由一系列`RouteDefinitionLocator`集合实现的。 

**RouteDefinitionLocator.java**

```
public interface RouteDefinitionLocator {
	Flux<RouteDefinition> getRouteDefinitions();
}
```
默认情况下,一个`PropertiesRouteDefinitionLocator` 使用Spring boot的`@ConfigurationPropertiesloads`机制载入属性配置。
上面的配置示例使用的是使用位置参数而不是命名参数的快捷符号。下面的两个例子是等价的：

```
spring:
  cloud:
    gateway:
      routes:
      # =====================================
      - id: setstatus_route
        uri: http://example.org
        filters:
        - name: SetStatus
          args:
            status: 401
      - id: setstatusshortcut_route
        uri: http://example.org
        filters:
        - SetStatus=401
```
对于网关的一些用法，属性将是足够的，但是一些生产用例将受益于从外部源（如数据库）加载配置。未来的里程碑版本将有`Routedefinitionlocator`实现基于数据库如：Redis、MongoDB和Cassandra。
#### 7.1 流畅的Java路由API
为了允许在java的简单配置，`Routes`提供了一种流畅的API使用方式
**GatewaySampleApplication.java**

```
// static imports from GatewayFilters and RoutePredicates
@Bean
public RouteLocator customRouteLocator(ThrottleGatewayFilterFactory throttle) {
    return Routes.locator()
            .route("test")
                .predicate(host("**.abc.org").and(path("/image/png")))
                .addResponseHeader("X-TestHeader", "foobar")
                .uri("http://httpbin.org:80")
            .route("test2")
                .predicate(path("/image/webp"))
                .add(addResponseHeader("X-AnotherHeader", "baz"))
                .uri("http://httpbin.org:80")
            .route("test3")
                .order(-1)
                .predicate(host("**.throttle.org").and(path("/get")))
                .add(throttle.apply(tuple().of("capacity", 1,
                     "refillTokens", 1,
                     "refillPeriod", 10,
                     "refillUnit", "SECONDS")))
                .uri("http://httpbin.org:80")
            .build();
}
```
这种样式还允许使用更多的自定义谓词断言。这些自定义的谓词可以通过`and`连接在一起。利用java的流畅API，你可以使用and()，or()和negate()等方法来连接谓词。
#### 7.2 `DiscoveryClient` Route Definition Locator
Spring cloud Gateway可以通过发现服务方式配置路由。想要使用这个功能，需要配置`spring.cloud.gateway.discovery.locator.enabled=true`并且确保`DiscoveryClient`在类路径上并且已经打开。