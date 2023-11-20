_이 page 에서는 Spring Framework 6.x.로 upgrade 하는 방법에 대한 안내를 제공한다._

## Upgrading to Version 6.1

70 revisions 기준으로 작성됨

### Baseline upgrades

Spring Framework 6.1은 다음 library를 통해 최소 요구 사항을 높였다:

* SnakeYAML 2.0
* Jackson 2.14
* Kotlin Coroutines 1.7
* Kotlin Serialization 1.5 

### Removed APIs

더 이상 사용되지 않는 여러 class, constructor 및 method가 code base 전체에서 제거되었다. [29449](https://github.com/spring-projects/spring-framework/issues/29449) 및 [30604](https://github.com/spring-projects/spring-framework/issues/30604) 참조.

몇 년 동안 공식적으로 또는 사실상 더 이상 사용되지 않는 RPC-style remote가 제거되었다.
이는 Hessian, HTTP Invoker, JMS Invoker 및 JAX-WS 지원에 영향을 미친다. [27422](https://github.com/spring-projects/spring-framework/issues/27422) 참조.

이러한 노력의 일환으로 EJB access도 제거되었다.
EJB를 lookup 해야 하는 경우, `JndiObjectFactoryBean` 또는 `<jee:jndi-lookup>` 을 통해 직접 JNDI를 사용함.

### Core Container

JDK 20에서 `java.net.URL` constructor가 더 이상 사용되지 않음에 따라 이제 상대 경로 처리를 포함하여 `URL` 확인이 `URI` 를 통해 일관되게 수행된다.
여기에는 전체 URL을 상대 경로로 지정할 때와 같이 흔하지 않은 경우에 대한 동작 변경이 포함된다. [29481](https://github.com/spring-projects/spring-framework/issues/29481) 및 [28522](https://github.com/spring-projects/spring-framework/issues/28522) 참조.

6.1에서 `LocalVariableTableParameterNameDiscoverer` 가 제거되었다.
`StandardReflectionParameterNameDiscoverer` 와 호환되도록 하려면 (`-debug` compiler flag를 사용하는 대신) parameter name 유지를 위해 일반적인 Java 8+ `-parameters` flag를 사용하여 Java source를 컴파일해야 한다.
Kotlin compiler에서는 `-java-parameters` flag를 사용하는 것이 좋다.

`AutowireCapableBeanFactory.createBean(Class, int, boolean)` 는 이제 더 이상 사용되지 않으며, 규칙에 기반한 `createBean(Class)` 를 선호한다.
후자는 6.1에서도 내부적으로 일관되게 사용된다. – 예, Quartz에서 `SpringBeanJobFactory` 및 Hibernate에서 `SpringBeanContainer` .

Array-to-collection 변환은 선언된 대상 `Collection` type에 대해 `Set` 이 아닌 `List` 결과를 선호한다.

application context가 닫히기 시작하면 `ThreadPoolTaskExecutor` 와 `ThreadPoolTaskScheduler` 가 graceful shutdown phase로 진입한다.
따라서 다른 component에서 callback을 stop하거나 destroy 하는 동안에는 더 이상 추가 task submission이 허용되지 않는다.
후자가 필요한 경우, shutdown phase가 길어지는 대신 executor/scheduler의 `acceptTasksAfterContextClose` flag 를 `true` 로 전환해야 한다.

`ApplicationContext` 를 통해 message 확인(internal `MessageSource` 에 access)은 context가 활성화 되어 있는 동안 허용된다.
context가 닫힌 후 `getMessage` 를 시도하면 이제 `IllegalStateException` 이 발생한다.

native image를 build 할 때 사전 계산된 field에 대한 자세한 logging은 기본적으로 제거되었으며, 관련 상세 로그를 표시하려면 `native-image` compiler build argument로 `-Dspring.native.precompute.log=verbose` 를 전달하여 복원할 수 있다.

### Data Access and Transactions

`@TransactionalEventListener` 는 동일한 method에서 잘못된 `@Transactional` 사용을 거부한다:  `REQUIRES_NEW` 로만 허용된다.(`@Async` 와 함께 사용할 수 있음).

이제 Hibernate Validator 설정이 불완전한 경우 (예: EL provider가 없는 경우) JPA bootstrapping이 실패하므로 이러한 경우를 더 쉽게 디버깅할 수 있다.

`HibernateJpaDialect` 를 사용하는 경우 `JpaTransactionManager` 는 가능한 경우 commit/rollback exceptions을 `DataAccessException` subclass로 변환하므로, 이전에는 generic `JpaSystemException` 으로 전파되던 Hibernate transaction exception이 이제는 예를 들어 `CannotAcquireLockException` 으로 표시될 수 있다.
번역할 수 없는 fallback exception의 경우, 이제 commit/rollback에서 전파되던 이전의 `TransactionSystemException` 대신 `JpaSystemException` 이 일관되게 발생한다.

### Web Applications

Spring MVC 와 WebFlux는 이제 `@Constraint` annotation이 있는 controller method parameter에 대한 method validation을 기본으로 지원한다.
이 기능을 사용하려면 1) controller class level에서 `@Validated` 를 제거하여 AOP 기반 method validation을 opt out 하고, 2) `mvcValidator` 또는 `webFluxValidator` bean이 `jakarta.validation.Validator` type (예: `LocalValidatorFactoryBean` ) 인지 확인하고, 3) method parameter에 직접 constraint annotation을 추가해야 한다.
method validation이 필요한 경우 (즉, constraint annotation이 있는 경우) `@Valid` 가 있는 model attribute 및 request body argument 도 method level에서 validation을 수행하며, 이 경우 argument resolver level에서 더 이상 validation을 수행하지 않으므로 이중 validation 수행을 피할 수 있다.
`BindingResult` argument는 여전히 존중되지만, 존재하지 않거나 다른 parameter에서 method validation이 실패하면 `MethodValidationException` 이 발생한다.
아직 6.1 M1에서는 처리되지 않지만 M2에서는 [30644](https://github.com/spring-projects/spring-framework/issues/30644)와 함께 처리될 예정이다.
M1 지원에 대한 자세한 내용은 [29825](https://github.com/spring-projects/spring-framework/issues/29825) 를 참조하고, 기타 모든 관련 작업 및 feedback 제공에 대해서는 포괄적인 issue [30645](https://github.com/spring-projects/spring-framework/issues/30645) 를 참조.

`MethodArgumentNotValidException` 및 `WebExchangeBindException` message argument가 변경되었다.
이제 오류는 작은 따옴표와 괄호 없이 `", 및 "` 로 연결된다.
Field error는 field name을 추가하는 등의 추가 작업없이 `MessageSource` 를 통해 해결된다.
이를 통해 application은 개별 error code를 사용자 지정하여 error 형식을 완벽하게 제어할 수 있다.
[30198](https://github.com/spring-projects/spring-framework/issues/30198) 및 계획된 문서 개선 [30653](https://github.com/spring-projects/spring-framework/issues/30653) 참조.

Spring MVC에서 `RouterFunctionMapping` order를 `3` 에서 `-1` 로 변경하여 mapping의 default order를 보다 일관성 있게 개선했다.
즉, 이제 Spring MVC 와 Spring WebFlux 모두에서 `RouterFunctionMapping` 이 항상 `RequestMappingHandlerMapping` 보다 먼저 정렬된다.
자세한 내용은 [30278](https://github.com/spring-projects/spring-framework/issues/30278) 참조.

`DispatcherHandler` 의 `throwExceptionIfNoHandlerFound` property는 이제 기본적으로 `true` 로 설정되며 더 이상 사용되지 않는다.
resulting exception은 기본적으로 404 error로 처리되므로 동일한 결과를 얻을 수 있다.
마찬가지로, `ResourceHttpRequestHandler` 는 이제 기본적으로 404로 처리되며 대부분의 application에서 동일한 결과를 가져올 수 있는 `NoResourceFoundException` 을 발생시킨다.
[29491](https://github.com/spring-projects/spring-framework/issues/29491) 참조.

`@RequestParam`, `@RequestHeader` 및 기타 controller method argument annotation은 이제 input이 text가 없는 non-empty String인 경우 defaultValue를 사용한다.

`ResponseBodyEmitter` 는 이제 exception이 `IOException` 이 아닌 경우 response를 complete 한다.
issue [30687](https://github.com/spring-projects/spring-framework/issues/30687) 참조.

Preflight check는 이제 `HandlerInteceptor` chain의 끝이 아닌 시작 지점에 실행된다.

[HTTP interface client](https://docs.spring.io/spring-framework/reference/integration/rest-clients.html#rest-http-interface)는 더 이상 blocking signature가 있는 method에 5초의 default timeout을 적용하지 않으며, 대신 기본 HTTP client의 default timeout 및 configuration 설정에 의존한다.
[30248](https://github.com/spring-projects/spring-framework/issues/30248) 참조.

WebFlux의 HTTP server Observability는 제한적이며 error를 제대로 관찰하지 못했다.
그 결과, `WebHttpHandlerBuilder` 에서 직접 계측하는 것을 선호하여 WebFlux `ServerHttpObservationFilter` 는 이제 더 이상 사용되지 않는다.
[30013](https://github.com/spring-projects/spring-framework/issues/30013) 참조.

`ReactorResourceFactory` class가 `org.springframework.http.client.reactive` package에서 `org.springframework.http.client` package로 이동되었다.

### Messaging Applications

[RSocket interface client](https://docs.spring.io/spring-framework/reference/rsocket.html#rsocket-interface)는 더 이상 blocking signature가 있는 method에 대해 5초의 default timeout을 적용하지 않으며, 대신 RSocket client의 default timeout 및 configuration 설정과 RSocket transport에 의존한다.
[30248](https://github.com/spring-projects/spring-framework/issues/30248) 참조.

Spring Expression Language (SpEL)의 보안 취약점이 Spring application에 악영향을 미칠 가능성을 줄이기 위해 팀은 신뢰할 수 없는 source의 SpEL expression 평가에 대한 지원을 기본적으로 비활성화 하기로 결정했다.
core Spring Framework 내에서 이것은 WebSocket messaging의 SpEL 기반 `selector` header 지원, 특히 `DefaultSubscriptionRegistry` 에 적용된다.
`selector` header 지원은 그대로 유지되지만 Spring Framework 6.1부터 명시적으로 사용하도록 설젛해야 한다. ([30550](https://github.com/spring-projects/spring-framework/issues/30550) 참조).
예를 들어 `WebSocketMessageBrokerConfigurer` 의 custom implementation은  `configureMessageBroker()` method를 재정의하고 다음과 같이 selector header name을 구성할 수 있다: `registry.enableSimpleBroker().setSelectorHeaderName("selector");` .


## Upgrading to Version 6.0

### Core Container

The JSR-330 based `@Inject` annotation is to be found in `jakarta.inject` now. The corresponding JSR-250 based annotations `@PostConstruct` and `@PreDestroy` are to be found in `jakarta.annotation`. For the time being, Spring keeps detecting their `javax` equivalents as well, covering common use in pre-compiled binaries.

The core container performs basic bean property determination without `java.beans.Introspector` by default. For full backwards compatibility with 5.3.x in case of sophisticated JavaBeans usage, specify the following content in a `META-INF/spring.factories` file which enables 5.3-style full `java.beans.Introspector` usage: `org.springframework.beans.BeanInfoFactory=org.springframework.beans.ExtendedBeanInfoFactory`

When staying on 5.3.x for the time being, you may enforce forward compatibility with 6.0-style property determination (and better introspection performance!) through a custom `META-INF/spring.factories` file: `org.springframework.beans.BeanInfoFactory=org.springframework.beans.SimpleBeanInfoFactory`

`LocalVariableTableParameterNameDiscoverer` is deprecated now and logs a warning for each successful resolution attempt (it only kicks in when `StandardReflectionParameterNameDiscoverer` has not found names). Compile your Java sources with the common Java 8+ `-parameters` flag for parameter name retention (instead of relying on the `-debug` compiler flag) in order to avoid that warning, or report it to the maintainers of the affected code. With the Kotlin compiler, we recommend the `-java-parameters` flag for completeness.

`LocalValidatorFactoryBean` relies on standard parameter name resolution in Bean Validation 3.0 now, just configuring additional Kotlin reflection if Kotlin is present. If you refer to parameter names in your Bean Validation setup, make sure to compile your Java sources with the Java 8+ `-parameters` flag.

`ListenableFuture` has been deprecated in favor of `CompletableFuture`. See [27780](https://github.com/spring-projects/spring-framework/issues/27780).

Methods annotated with `@Async` must return either `Future` or `void`. This has long been documented but is now also actively checked and enforced, with an exception thrown for any other return type. See [27734](https://github.com/spring-projects/spring-framework/issues/27734).

`SimpleEvaluationContext` disables array allocations now, aligned with regular constructor resolution.

### Caching

The `org.springframework.cache.ehcache` package has been removed as it was providing support for ehcache 2.x - with this version, `net.sf.ehcache` is using JavaEE APIs and [is about to be End Of Life'd](https://github.com/ehcache/ehcache2). Ehcache3 is the direct replacement. You should revisit your dependency management to use `org.ehcache:ehcache` (with the `jakarta` classifier) instead and look [into the official migration guide or reach out to the ehcache community for assistance](https://www.ehcache.org/documentation/3.10/migration-guide.html). We did not replace `org.springframework.cache.ehcache` with an updated version, as using ehcache through the JCache API or its new native API is preferred.

### Data Access and Transactions

Due to the Jakarta EE migration, make sure to upgrade to Hibernate ORM 5.6.x with the `hibernate-core-jakarta` artifact, alongside switching your `javax.persistence` imports to `jakarta.persistence` (Jakarta EE 9). Alternatively, consider migrating to Hibernate ORM 6.1 right away (exclusively based on `jakarta.persistence`, compatible with EE 9 as well as EE 10) which is the Hibernate version that Spring Boot 3.0 comes with.

The corresponding Hibernate Validator generation is 7.0.x, based on `jakarta.validation` (Jakarta EE 9). You may also choose to upgrade to Hibernate Validator 8.0 right away (aligned with Jakarta EE 10).

For EclipseLink as the persistence provider of choice, the reference version is 3.0.x (Jakarta EE 9), with EclipseLink 4.0 as the most recent supported version (Jakarta EE 10).

Spring's default JDBC exception translator is the JDBC 4 based `SQLExceptionSubclassTranslator` now, detecting JDBC driver subclasses as well as common SQL state indications (without database product name resolution at runtime). As of 6.0.3, this includes a common SQL state check for `DuplicateKeyException`, addressing a long-standing difference between SQL state mappings and legacy default error code mappings.

`CannotSerializeTransactionException` and `DeadlockLoserDataAccessException` are deprecated as of 6.0.3 due to their inconsistent JDBC semantics, in favor of the `PessimisticLockingFailureException` base class and consistent semantics of its common `CannotAcquireLockException` subclass (aligned with JPA/Hibernate) in all default exception translation scenarios.

For full backwards compatibility with database-specific error codes, consider re-enabling the legacy `SQLErrorCodeSQLExceptionTranslator`. This translator kicks in for user-provided `sql-error-codes.xml` files. It can simply pick up Spring's legacy default error code mappings as well when triggered by an empty user-provided file in the root of the classpath.

### Web Applications

Due to the Jakarta EE migration, make sure to upgrade to Tomcat 10, Jetty 11, or Undertow 2.2.19 with the `undertow-servlet-jakarta` artifact, alongside switching your `javax.servlet` imports to `jakarta.servlet` (Jakarta EE 9). For the latest server generations, consider Tomcat 10.1 and Undertow 2.3 (Jakarta EE 10).

Several outdated Servlet-based integrations have been dropped: e.g. Apache Commons FileUpload (`org.springframework.web.multipart.commons.CommonsMultipartResolver`), and Apache Tiles as well as FreeMarker JSP support in the corresponding `org.springframework.web.servlet.view` subpackages. We recommend `org.springframework.web.multipart.support.StandardServletMultipartResolver` for multipart file uploads and regular FreeMarker template views if needed, and a general focus on REST-oriented web architectures.

Spring MVC and Spring WebFlux no longer detect controllers based solely on a type-level `@RequestMapping` annotation. That means interface-based AOP proxying for web controllers may no longer work. Please, enable class-based proxying for such controllers; otherwise the interface must also be annotated with `@Controller`. See [22154](https://github.com/spring-projects/spring-framework/issues/22154).

`HttpMethod` is now a class and no longer an enum. Though the public API has been maintained, some  migration might be necessary (i.e. change from `EnumSet<HttpMethod>` to `Set<HttpMethod>`, use `if  else` instead of `switch`). For the rationale behind this decision, see [27697](https://github.com/spring-projects/spring-framework/issues/27697).

The Kotlin extension function to `WebTestClient.ResponseSpec::expectBody` now returns the Java `BodySpec` type and no longer uses the workaround type `KotlinBodySpec`. Spring 6.0 uses Kotlin 1.6, which fixed the bug that needed this workaround ([KT-5464](https://youtrack.jetbrains.com/issue/KT-5464)). This means that `consumeWith` is no longer available.

`RestTemplate`, or rather the `HttpComponentsClientHttpRequestFactory`, now requires Apache HttpClient 5.

The Spring-provided Servlet mocks (`MockHttpServletRequest`, `MockHttpSession`) require Servlet 6.0 now, due to a breaking change between the Servlet 5.0 and 6.0 API jars. They can be used for testing Servlet 5.0 based code but need to run against the Servlet 6.0 API (or newer) on the test classpath. Note that your production code may still compile against Servlet 5.0 and get integration-tested with Servlet 5.0 based containers; just mock-based tests need to run against the Servlet 6.0 API jar.

`SourceHttpMessageConverter` is not configured by default anymore in Spring MVC and `RestTemplate`. As a consequence, Spring web applications using `javax.xml.transform.Source` now need to configure `SourceHttpMessageConverter` explicitly. Note that the order of converter registration is important, and `SourceHttpMessageConverter` should typically be registered before "catch-all" converters like `MappingJackson2HttpMessageConverter` for example.