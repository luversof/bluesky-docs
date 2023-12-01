= Spring Boot 3.2 Release Notes

21 revisions 기준으로 작성함

== Upgrading from Spring Boot 3.1


=== Parameter Name Discovery
Spring Boot 3.2에서 사용하는 Spring Framework version은 더 이상 bytecode를 파싱하여 parameter name을 추론하지 않는다.
dependency injection 또는 property binding 관련된 issue가 발생하면 `-parameters` option을 사용하여 compile 하고 있는지 다시 한번 확인해야 한다.
자세한 내용은 https://github.com/spring-projects/spring-framework/wiki/Upgrading-to-Spring-Framework-6.x#parameter-name-retention["Upgrading to Spring Framework 6.x" wiki의 해당 section]을 잠조.


=== Logged Application Name
이제 `spring.application.name` property를 설정할 때마다 기본 log 출력에 application name이 포함된다.
이전 형식을 선호하는 경우 `logging.include-application-name` 를 `false` 로 설정하면 된다.


=== Auto-configured User Details Service
auto-configure 된 `InMemoryUserDetailsManager`  는 `spring-security-oauth2-client`, `spring-security-oauth2-resource-server`, 및 `spring-security-saml2-service-provider` 중 하나 이상이 classpath에 있을 때 back off 된다.
마찬가지로 reactive application에서 이제 auto-configure 된 `MapReactiveUserDetailsService` 는 `spring-security-oauth2-client` 및 `spring-security-oauth2-resource-server` 중 하나 이상이 classpath에 있는 경우 back off 된다.


위의 dependency 중 하나를 사용중이지만 application에 여전히 `InMemoryUserDetailsManager` 또는 `MapReactiveUserDetailsService` 가 필요한 경우, application에서 필요한 bean을 정의해야 한다.


=== OTLP Tracing Endpoint
`management.otlp.tracing.endpoint` 의 default value가 제거되었다.
이제 `management.otlp.tracing.endpoint` 가 값이 있는 경우에만  `OtlpHttpSpanExporter` bean이 auto-configure 된다.
이전 동작을 복원하려면 `management.otlp.tracing.endpoint=http://localhost:4318/v1/traces` 를 설정하면 된다.


=== H2 Version 2.2
이제 Spring Boot는 기본적으로 H2 version 2.2를 사용한다.
이전 버전의 H2에서 database를 계쏙 사용하려면 data migration을 수행해야 할 수 있다.
upgrade 하기 전에 `SCRIPT` command를 사용하여 database를 export한다.
새 version의 H2로 빈 database를 생성한 다음 `RUNSCRIPT` command를 사용하여 data를 가져온다.



=== Oracle UCP DataSource
Oracle UCP DataSource는 더 이상 기본적으로 `validateConnectionOnBorrow` 룰 `true` 로 설정하지 않는다.
이전 동작을 복원해야 하는 경우 `spring.datasource.oracleucp.validate-connection-on-borrow` application property 를 `true` 로 설정하면 된다.



=== Jetty 12
Spring Boot는 이제 Jetty 12를 지원한다.
Jetty 12는 Servlet 6.0 API를 지원하여 Tomcat 및 Undertow에 모두 부합한다.
이전에는 Spring Boot 3.x와 함께 Jetty를 사용하는 경우 Servlet API를 5.0으로 downgrade 해야 했다.
이제 더 이상 그럴 필요가 없다.
upgrade 할 때 Servlet API version에 대한 override를 제거한다.


=== Kotlin 1.9.0 and Gradle
1.9.0 version에는 additional resource directory가 손실될 수 있는 https://youtrack.jetbrains.com/issue/KT-60459/Gradle-Plugin-overwrites-resource-directories[a bug]가 있다.
이로 인해 AOT 처리에 의해 생성된 resource가 native image의 classpath에 포함되지 않으므로 native image compile이 중단된다.
이 문제를 해결하려면 Kotlin의 Gradle plugin을 적용한다.


=== Nested Jar Support
이제 더 이상 Java 8을 지원할 필요가 없으므로 Spring Boot의 "Uber Jar" loading을 지원하는 기본 코드가 다시 작성되었다.
update 된 코드는 JDK 기대치에 더 부합하는 새로운 URL format을 사용한다.
`jar:file:/dir/myjar.jar:BOOT-INF/lib/nested.jar!/com/example/MyClass.class` 의 이전 URL format은 `jar:nested:/dir/myjar.jar/!BOOT-INF/lib/nested.jar!/com/example/MyClass.class` 로 대체되었다.
또한 update 된 코드는 resource 관리를 위해 `java.lang.ref.Cleaner` (JDK 9의 일부)를 사용한다.


우리는 가능한 한 새 코드가 이전 구현을 투명하게 대체할 수 있도록 최선을 다했다.
대부분의 사용자가 변경 사항을 눈치채지 못할 것으로 예상된다.
새로운 default launcher에서 새로운 name을 갖게 된 launcher class 중 하나를 직접 참조하는 경우 변경 사항을 알아차릴 수 있는 한 가지 영역이 있다:

[cols="1,1"]
|===
|New | Classic

| `org.springframework.boot.loader.launch.JarLauncher`
| `org.springframework.boot.loader.JarLauncher`

| `org.springframework.boot.loader.launch.PropertiesLauncher`
| `org.springframework.boot.loader.PropertiesLauncher`

| `org.springframework.boot.loader.launch.WarLauncher`
| `org.springframework.boot.loader.WarLauncher`
|===

그러나 새 구현에서 문제를 발견하는 경우 이전 코드를 사용할 수 있는 대체 option을 제공했다.


Gradle 사용자의 경우 `bootJar.loaderImplementation` 를 `org.springframework.boot.loader.tools.LoaderImplementation.CLASSIC` 으로 설정할 수 있다.
예를 들면 다음과 같다:

[source,gradle]
----
bootJar {
  loaderImplementation = org.springframework.boot.loader.tools.LoaderImplementation.CLASSIC
}
----

Maven 사용자의 경우 `spring-boot-plugin` configuration에서 `<loaderImplementation>` tag를 `CLASSIC` 으로 설정할 수 있다.
예를 들면 다음과 같다:

[source,xml]
----
<build>
  <plugins>
    <plugin>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-maven-plugin</artifactId>
      <executions>
        <execution>
          <goals>
            <goal>repackage</goal>
          </goals>
          <configuration>
            <loaderImplementation>CLASSIC</loaderImplementation>
          </configuration>
        </execution>
      </executions>
    </plugin>
  </plugins>
</build>
----

새 구현에서 예기치 않은 동작이 발견되면 Github issue를 제기해주면 된다.


=== Deprecations from Spring Boot 3.0

Spring Boot 3.0에서 더 이상 사용되지 않는 class, method 및 property가 이번 release에서 제거되었다.
upgrade 하기 전에 더 이상 사용되지 않는 method를 호출하고 있지 않은지 확인해야 한다.



=== Minimum Requirements Changes
None.



== New and Noteworthy
TIP: configuration의 변경 사항에 대한 overview는 https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-3.2.0-Configuration-Changelog[configuration changelog] 를 확인


=== Spring for Apache Pulsar Support
이제 Spring Boot에는 auto-configuration 지원과 https://github.com/spring-projects/spring-pulsar[Spring for Apache Pulsar project]를 위한 starter POM이 포함되어 있다.
자세한 내용은  https://docs.spring.io/spring-boot/docs/3.2.0/reference/html/messaging.html#messaging.pulsar[update된 reference documentation]을 참조.



=== Logging Correlation IDs
이제 Spring Boot는 Micrometer tracing을 사용할 때마다 correlation ID를 자동으로 log한다.
자세한 내용은 https://docs.spring.io/spring-boot/docs/3.2.0/reference/html//actuator.html#actuator.micrometer-tracing.logging[update된 documenation]을 참조.



=== RestClient Support
Spring Boot 3.2에는 Spring Framework 6.1에 도입된 새로운 `RestClient` interface에 대한 지원이 포함되어 있다.
이 interface는 `WebClient` 와 유사한 디자인으로 functional style blocking HTTP API를 제공한다.

기존 application에서 신규 application은 `RestTemplate` 대신 `RestClient` 를 사용하는 것을 고려할 수 있다. 

자세한 내용은 https://docs.spring.io/spring-boot/docs/3.2.0/reference/html//io.html#io.rest-client.restclient[update된 reference documentation] 참조.



=== RestTemplate HTTP Clients
Jetty의 `HttpClient` 가 classpath에 있는 경우, Spring Boot의 HTTP client auto-detection은 이제 Spring Framework 6.1에 도입된 새로운 `JettyClientHttpRequestFactory` 를 사용하도록 `RestTemplateBuilder` 를 구성한다.

`ClientHttpRequestFactories` 에 `JdkClientHttpRequestFactory` 에 대한 지원이 추가되었다.
`JettyClientHttpRequestFactory` 와 달리 auto-detection에 추가되지 않았다.
`JdkClientHttpRequestFactory` 을 사용하려면 opt in 해야 한다:


[source,java]
----
@Bean
RestTemplateBuilder restTemplateBuilder(RestTemplateBuilderConfigurer configurer) {
    return configurer.configure(new RestTemplateBuilder())
        .requestFactory(
                (settings) -> ClientHttpRequestFactories.get(JdkClientHttpRequestFactory.class, settings));
}
----



=== Support for `JdbcClient`
`NamedParameterJdbcTemplate` 의 존재 여부에 따라 https://docs.spring.io/spring-boot/docs/3.2.0/reference/html//data.html#data.sql.jdbc-client[`JdbcClient`]에 대한 auto-configuration이 추가되었다.
후자가 auto-configure 된 경우 `spring.jdbc.template.*` 의 property가 고려된다. 


=== Support for Virtual Threads
Spring Boot 3.2는 https://openjdk.org/jeps/444[virtual threads]를 지원한다.
virtual thread를 사용하려면 Java 21에서 실행하고, `spring.threads.virtual.enabled` property를 `true` 로 설정해야 한다.



==== Servlet Web Servers
virtual thread가 활성화 되면 Tomcat과 Jetty는 request 처리를 위해 virtual thread를 사용한다.
즉, controller의 method와 같이 web request를 처리하는 application code가 virtual thread에서 실행된다.


==== Blocking Execution with Spring WebFlux
Spring WebFlux의 block execution에 대한 지원은 `AsyncTaskExecutor` 인 경우 `applicationTaskExecutor` bean을 사용하도록 auto-configure 된다.
`applicationTaskExecutor` 는 default 및 virtual thread가 활성화 된 경우 모두 `AsyncTaskExecutor` 이다.


==== Task Execution
virtual thread가 활성화 된 경우  `applicationTaskExecutor` bean은 virtual thread를 사용하도록 구성된 `SimpleAsyncTaskExecutor` 가 된다.
이제 `@Async` method 호출 시 `@EnableAsync` 와 같은 application task executor, Spring MVC의 asynchronous request processing, Spring WebFlux의 blocking execution 지원 등 application task executor를 사용하는 모든 곳에서 virtual thread를 활용하게 된다.
이전과 마찬가지로 auto-configure된 executor 에 `TaskDecorator` bean이 적용되고 `spring.task.execution.thread-name-prefix` property가 적용된다.
다른 `spring.task.execution.*` property 들은 pool-based executor에만 해당되므로 무시된다.


이제 application context에서 `SimpleAsyncTaskExecutorBuilder` 를 사용할 수 있으며, 이를 사용하여 `SimpleAsyncTaskExecutor` 를 빌드할 수 있다.
`SimpleAsyncTaskExecutorCustomizer` bean을 사용하여 빌드된 `SimpleAsyncTaskExecutor` 를 customize 할 수 있다.
virtual thread가 활성화된 경우 builder는 virtual thread를 사용하도록 auto-configure 된다.



==== Task Scheduling
virtual thread를 사용하도록 설정하면 `taskScheduler` bean은 virtual thread를 사용하도록 구성된 `SimpleAsyncTaskScheduler` 가 된다.
`spring.task.scheduling.thread-name-prefix` property 및 `spring.task.scheduling.simple.*` property 들이 적용된다.
다른 `spring.task.scheduling.*` property 들은 pool-based scheduler에만 해당되므로 무시된다.

이제 application context에서 `SimpleAsyncTaskSchedulerBuilder` 를 사용할 수 있으며, 이를 사용하여 `SimpleAsyncTaskScheduler` 를 빌드할 수 있다.
`SimpleAsyncTaskSchedulerCustomizer` bean을 사용하여 빌드된 `SimpleAsyncTaskScheduler` 를 customize 할 수 있다.
virtual thread가 활성화된 경우 builder는 virtual thread를 사용하도록 auto-configure 된다.



==== Keeping the JVM Alive
`spring.main.keep-alive` 라는 새로운 property가 있다.
`true` 로 설정하면 다른 모든 thread가 virtual (또는 daemon) thread인 경우에도 JVM이 계속 활성화된다.



==== Technology Specific Integrations
virtual thread가 활성화되면 다음과 같은 기술별 통합이 적용된다:

* virtual thread executor는 RabbitMQ listener에 대해 auto-configure 된다.
* virtual thread executor는 Kafka listener에 대해 auto-configure 된다.
* Spring Data Redis의 `ClusterCommandExecutor` 는 virtual thread를 사용한다.
* Apache Pulsar 용 Spring은 auto-configure된 `ConcurrentPulsarListenerContainerFactory` 및 `DefaultPulsarReaderContainerFactory` 에 `VirtualThreadTaskExector` 를 사용한다.



=== Initial support for JVM Checkpoint Restore
Spring Boot 3.2는 JVM Checkpoint 복원 (https://openjdk.org/projects/crac/[Project CRaC])에 대한 초기 지원을 제공한다.
자세한 내용은 https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#deployment.efficient.checkpoint-restore[관련 documentation] 참조.


=== SSL Bundle Reloading
이제 trust material이 변경되면 SSL bundle을 자동으로 reload 할 수 있다.
bundle은 `reload-on-update` property 를 `true` 로 설정하여 이 기능을 사용하도록 선택해야 한다.
bundle의 consumer도 reloading을 지원해야 한다.

reloading을 지원하는 consumer는 다음과 같다:

- Netty web server
- Tomcat web server

SSL bundle reloading에 대한 자세한 내용은 https://docs.spring.io/spring-boot/docs/3.2.0/reference/html/features.html#features.ssl.reloading[reference documentation] 참조.


=== Observability Improvements
이제 Micrometer의 https://micrometer.io/docs/concepts#_the_timed_annotation[`@Timed`], `@Counted`, `@NewSpan`, `@ContinueSpan` 및  https://micrometer.io/docs/observation#_using_annotations_with_observed[`@Observed`] annotation을 사용할 수 있다.
이제 classpath에 AspectJ가 있는 경우 이들에 대한 aspect가 auto-configure 된다.

Micrometer Tracing의 `ObservationHandler` bean은 `ObservationConfig` 에 자동으로 등록된다.
Spring Boot 3.2.0 이전에는 uncategorized handler가 categorized handler 보다 먼저 등록되었다.
이제 categorized handler가 uncategorized handler보다 먼저 등록된다.
자세한 내용은 https://github.com/spring-projects/spring-boot/issues/34399[#34399] 참조.

B3 trace propagation의 default format이 single-no-parent에서 https://github.com/openzipkin/b3-propagation#single-header[single]로 변경되었다.

`@Scheduled` method는 이제 observability를 위해 계측된다.

R2DBC에 대한 Observability가 추가되었다.
이 기능을 사용하려면 프로젝트에 `io.r2dbc:r2dbc-proxy` dependency를 포함한다.



==== Properties
reactive pipeline에서 context propagation을 제어하는 `spring.reactor.context-propagation` 라는 새로운 configuration property가 있다.
reactive pipeline에서 observations, trace ids 및 span ids를 자동으로 전파하려면 이 property를 `auto` 로 설정하면 된다. 

이제 property 들을 통해 prefix로 시작하는 Observation을 비활성화 할 수 있다.
예를 들어 Spring Security가 observation을 report하지 못하도록 하려면 `management.observations.enable.spring.security=false` 로 설정하면 된다.

`management.observations.key-values.*` property를 사용하여 모든 observation에 low-cardinality key-value들을 자동으로 적용할 수 있다.
예를 들어 `management.observations.key-values.region=us-west` 를 설정하면 모든 관측값에 `us-west` 라는 값을 가진 key `region` 이 추가된다.



==== OpenTelemetry
OpenTelemetry `MeterProvider` bean이 발견되면 이 bean은 자동으로 `BatchSpanProcessor` 에 등록된다.

OpenTelemetry의 auto-configuration이 개선되엇다.
context에 `SdkLoggerProvider` 또는 `SdkMeterProvider` type의 bean이 있는 경우 해당 bean이 `OpenTelemetry` bean에 자동으로 등록된다.
또한 OpenTelemetry의 `Resource` 가 이제 bean으로 노출되며, resource attribute를 구성하는 새로운 configuration property `management.opentelemetry.resource-attributes`  가 있다.

OpenTelemetry를 사용 중이고 적용된 `SpanProcessor` 를 보다 세밀하게 제어하고 싶다면 이제 `SpanProcessors` type의 bean을 정의할 수 있다.
default로 사용 가능한 모든 `SpanProcessor` bean이 적용된다.
default르 override 하려면 `SpanExporters` bean을 사용하여 OpenTelemetry의 `SpanExporter` 와 동일하게 동작한다.
default로 사용 가능한 모든 `SpanExporter` bean이 적용된다.


==== Broader Exemplar Support in Micrometer 1.12
Micrometer 1.12에는 https://github.com/prometheus/prometheus/pull/11984[requires Prometheus 2.43 이상]이상이 필요한 https://github.com/micrometer-metrics/micrometer/pull/3996[exemplar support를 확대하는] 기능이 포함되어 있다.
2.43.0 이전 버전을 사용 중이고 Micrometer Tracing을 사용 중인 경우, Prometheus >= 2.43.0으로 upgrade 하지 않으면 metric이 더 이상 표시되지 않는다.




==== Observability in Tests
Spring Boot 3.2 이전에는 integration test를 실행할 때 전체 Micrometer Tracing. Brave 및 OpenTelemetry infrastructure가 비활성화되었다.
이제 최소한의 bean만 비활성화되어 backend로 span이 전송되지 않도록 개선되었다.
(비활성화 되는 bean 목록은 https://github.com/spring-projects/spring-boot/issues/35354[#35354] 참조)
custom Brave `SpanHandler` 또는 OpenTelemetry `SpanExporter` bean이 있는 경우, observability이 꺼진 상태에서 integration test를 실행할 때 생성되지 않도록 `@ConditionalOnEnabledTracing` 으로 annotation을 달아야 한다.

integration test를 observability를 활성화한 상태로 실행하려면 test class에서 https://docs.spring.io/spring-boot/docs/3.2.0/reference/html//features.html#features.testing.spring-boot-applications.tracing[`@AutoConfigureObservability` annotation을 사용]하면 된다.



=== Docker Image Building


==== Default CNB Builders Upgraded
Maven 및 Gradle plugin으로 image를 빌드할 때 사용되는 기본 CNB builder가 변경되었다.
빌드에 GraalVM plugin을 적용하면 새로운 default builder는 `paketobuildpacks:builder-jammy-tiny` 이다.
그렇지 않으면 새로운 default builder는 `paketobuildpacks:builder-jammy-base` 이다.
이러한 builder에 대한 자세한 내용은 https://paketo.io/docs/reference/builders-reference/[Paketo documentation] 을 참조.

이전 default builder에는 Ubuntu 18.04 기반 run image가 포함되어 있었지만, 새로운 default builder에는 Ubuntu 22.04 기반 run image가 포함되어 있다.
즉, 새로운 default builder로 빌드된 모든 image는 Ubuntu 22.04를 기반으로 한다.


==== Docker Host Configuration
`spring-boot:build-image` Maven goal 및 `bootBuildImage` Gradle task는 이제 default로 사용해야 하는 Docker daemon의 host address 및 기타 connection details를 결정하기 위해 Docker CLI configuration file을 사용한다.
자세한 내용은 https://docs.spring.io/spring-boot/docs/3.2.0/gradle-plugin/reference/htmlsingle/#build-image.docker-daemon[Gradle] 및 https://docs.spring.io/spring-boot/docs/3.2.0/maven-plugin/reference/htmlsingle/#build-image.docker-daemon[Maven] plugin documentation을 참조.



==== Bind Mounts for Caches
이제 CNB builder 및 buildpack에서 사용하는 build 및 launch cache를 named volume 대신 bind moount를 사용하도록 구성할 수 있다.
이 기능은 CI pipeline에서 volume access를 허용하지 않는 BitBucket CI 사용자들이 요청해온 기능이다.
자세한 정보 및 예제는 https://docs.spring.io/spring-boot/docs/3.2.0/maven-plugin/reference/htmlsingle/#build-image.examples.caches[Maven] 및 https://docs.spring.io/spring-boot/docs/3.2.0/gradle-plugin/reference/htmlsingle/#build-image.examples.caches[Gradle] documentation 참조.


==== Build Workspace Configuration
이제 CNB builder 및 buildpack에서 temporary build workspace는 bind mount 또는 custom named volume을 사용하도록 구성할 수 있다.
자세한 내용과 예제는 https://docs.spring.io/spring-boot/docs/3.2.0/maven-plugin/reference/htmlsingle/#build-image.examples.caches[Maven] 및 https://docs.spring.io/spring-boot/docs/3.2.0/gradle-plugin/reference/htmlsingle/#build-image.examples.caches[Gradle] documentation 참조.


==== Security Options Configuration
이제 CNB builder container에 적용되는 security option을 customize 하여 default Linux security option `label=disable` 을 사용할 수 있는 Docker 환경을 지원할 수 있다.
자세한 내용은 https://docs.spring.io/spring-boot/docs/3.2.0/maven-plugin/reference/htmlsingle/#build-image.customization[Maven] 및 https://docs.spring.io/spring-boot/docs/3.2.0/gradle-plugin/reference/htmlsingle/#build-image.customization[Gradle] documentation 참조.


=== Spring for GraphQL's Callable Support
이제 GraphQL 용 Spring은 `applicationTaskExecutor` 를 사용하도록 auto-configure된다.
따라서 `Callable` 을 반환하는 controller method를 즉시 지원할 수 있다.



=== Additional OAuth2 Token Validators
이제 auto-configure된 `JwtDecoder` 또는 `ReactiveJwtDecoder` 는 token validation을 위해 `OAuth2TokenValidator<Jwt>` bean을 사용한다.
이 bean은 decoder의 validator로 구성된 `DelegatingOAuth2TokenValidator` 에 포함되어 있다.



=== Service Connection Support for ActiveMQ
Testcontainers 와 Docker Compose 모두에 대한 통합과 함께 `ServiceConnection` 에 대한 지원이 ActiveMQ에 추가되었다.
통합은 `symptoma/activemq` image를 사용한다.



=== Docker Compose Support for Neo4j
Spring Boot의 Docker Compose 통합은 이제 Neo4j를 지원한다.
인증을 비활성화(`none` value)하거나 `neo4j` 사용자에 대한 password를 설정(`neo4j/your-password` value)하려면 YAML에서 `NEO4J_AUTH` environment variable을 구성해야 한다.


=== WebSocketServerSpec Configuration
auto-configuration에 사용되는 `WebSocketServerSpec` 은 `spring.rsocket.server.spec` namespace의 property들을 사용하여 customize 할 수 있다.



=== Neo4j `AuthTokenManager`
`AuthTokenManager` bean이 정의된 경우, 이 bean은 Neo4j를 사용한 인증에 사용된다.
이러한 bean은 `spring.neo4j.authentication.*` property들 보다 우선한다.
예를 들어 Testcontainers 또는 Docker Compose managed database에 대한 service connection에 대해 custom `Neo4jConnectionDetails` 가 정의되어 있는 경우 `AuthTokenManager` bean은 무시된다.


=== RabbitMQ


==== SSL Bundle Support
이제 `spring.rabbitmq.ssl.bundle` property를 사용하여 SSL bundle의 SSL trust material를 사용하도록 RabbitMQ connection을 구성할 수 있다.
이는 기존 `spring.rabbitmq.ssl` property를 사용하여 trust material를 Java keystore file로 제공하는 대신 사용할 수 있는 대안이다.



==== Limited Message Body Size
최신 버전의 RabbitMQ용 Java client는 default로 inbound message의 최대 크기를 64MB로 제한한다.
이 제한을 customize 하기 위해 `spring.rabbitmq.max-inbound-message-body-size` configuration property가 도입되었다.


==== Virtual Host Support for RabbitMQ Stream
RabbitMQ Stream에 대한 virtual host 지원이 추가되었다.
명시적으로 설정하지 않으면 RabbitMQ Stream의 virtual host는 자동으로 RabbitMQ에 대해 구성된 virtual host를 사용한다.
특정 virtual host를 RabbitMQ Stream에서 사용하려면 `spring.rabbitmq.stream.virtual-host` 를 설정하면 된다.


=== Kafka
==== SSL Bundle Support
이제 `spring.kafka.ssl.bundle` property를 사용하여 SSL bundle의 SSL trust material을 사용하도록 Kafka connection을 구성할 수 있다.
이는 기존 `spring.kafka.ssl` property를 사용하여 trust material을 Java keystore file로 제공하는 대신 사용할 수 있는 대안이 된다.



=== Jms Sessions
auto-configure된 `JmsTemplate` 에 의해 생성된 session을 구성하기 위한 새로운 property들이 도입되었다:

- `spring.jms.template.session.acknowledge-mode`
- `spring.jms.template.session.transacted`

마찬가지로, auto-configure된 `JmsMessageListenerContainer` 에 대해 `spring.jms.listener.session.transacted` property 가 도입되었다.

이러한 새로운 property에 맞춰 기존 `spring.jms.listener.acknowledge-mode` property는 더 이상 사용되지 않으며 `spring.jms.listener.session.acknowledge-mode` 가 대체 property로 도입되었다.



=== Connection validation on Oracle UCP datasources
Oracle UCP datasource의 connection validation에 대한 기본값이 제거되었다.
3.2.1-RC1 이전에는 connection validation이 기본적으로 활성화되었지만 더 이상 그렇지 않다.
connection validation이 피룡한 경우 configuration property `spring.datasource.oracleucp.validate-connection-on-borrow` 를 `true` 로 설정하면 된다.



=== testAndDevelopmentOnly Gradle Configuration
Spring Boot의 Gradle plugin은 이제 `developmentOnly` 외에도 `testAndDevelopmentOnly` configuration도 생성한다.
`developmentOnly` 와 달리 이 새로운 configuration의 dependency들은 test compile 및 runtime classpath에 포함된다.
이 configuration은 주로 https://docs.spring.io/spring-boot/docs/3.2.0/reference/html/features.html#features.testing.testcontainers.at-development-time[development 시 Testcontainers]를 사용하는 application을 위한 것이다.



=== Miscellaneous
위에 나열된 변경 사항 외에도 다음과 같은 많은 사소한 조정과 개선이 이루어졌다:

* Jackson의 `EnumFeature` 및 `JsonNodeFeature` 에 선언된 기능은 이제 각각 `spring.jackson.datatype.enum.*` 및 `spring.jackson.datatype.jsonnode.*` configuration property 들을 사용하여 활성화 및 비활성화 할 수 있다.
* 이제 additional build info property 들은 `Provider` 를 사용하여 lazy value 들을 가질 수 있다.
* Transaction manager customization은 이제 `PlatformTransactionManager` 뿐만 아니라 모든 type의 `TransactionManager` 에 적용된다.
* 이제 모든 `TransactionExecutionListener` bean이 auto-configure 된 transaction manager에 추가된다.
* embedded WebServer가 시작될 때 기록되는 모든 port information이 개선되어 더욱 일관성이 높아졌다.
* 새로운 property `spring.servlet.multipart.strict-servlet-compliance` 는 multipart handing이 `multipart/form-data` request에만 사용되는지 여부를 설정한다.
* welcome page의 handler가 잘못된 `Accept` header를 수신할 때 logging이 줄어들어 모든 MIME types을 허용하는 것으로 되돌아갔다.
* Spring Boot의 기본값을 `RestClient.Builder` 에 적용하는데 사용할 수 있는 `RestClientBuilderConfigurer` 가 추가되었다.
* `restTemplateBuilderConfigurer` bean은 더 이상 사용자 정의 bean에 대해 back off 하지 않는다. 자체 `restTemplateBuilderConfigurer` bean이 있는 경우 해당 bean을 제거해야 한다.
* Jetty server의 connection maximum amount를 구성하는 property가 추가되었다.
* PEM SSL bundle을 사용할 때 key를 확인하는데 사용할 수 있는 새로운 porperty가 추가되었다.
* 이제 프로그래밍 방식으로  `PemSslStoreBundle` 을 만들 때 key store password를 제공할 수 있다.
* 이제 `service.name` 이 명시적으로 설정되지 않은 경우 `spring.application.name` 이 OpenTelemetry의 `service.name` 에 사용된다.
* exported metric의 base `TimeUnit` OTLP registry에서 구성할 수 있는 새로운 property가 추가되었다.
* 이제 OTLP metric 및 trace에 대한 connection details가 지원된다. connection details bean은 Testcontainer 또는 Docker Compose를 `otel/opentelemetry-collector-contrib` image와 함께 사용하는 경우 자동으로 생성된다.
* Wavefront 사용 시 CSP 인증에 대한 지원이 추가되었다.
* 새로운 property 인 `flyway.postgresql.transactional-lock` 를 사용하여 Flyway의 transaction lock 사용을 PostgreSQL로 구성할 수 있다.
* Kafka MessageListenerContainer `changeConsumerThreadName` property에 대한 지원이 추가되었다.
* `Function<MessageListenerContainer, String>` bean을 Kafka MessageListenerContainer's `threadNameSupplier` 로 auto-configure 한다.
* virtual thread 문제를 auto-configure 하는데 도움이 되는 새로운 `@ConditionalOnThreading` annotation이 도입되었다.
* RabbitMQ container `forceStop` property에 대한 지원이 추가되었다.
* `WebClient` based Zipkin sender는 이제 configuration property들을 통해 설정된 timeout을 준수한다. 자세한 내용은 https://github.com/spring-projects/spring-boot/issues/36264[#36264] 참조.
* AOT 모드가 활성화 된 상태에서 application을 시작했지만 빌드에서 AOT 처리가 완료되지 않은 경우, 이제 error message가 더 명확해졌다.
* 이제 GraalVM을 사용할 때 `messages.properties` 및 `messages_*.properties` 에 대한 resource hint가 자동으로 제공된다.
* 이제 Kotlin Serialization을 위한 dependency management가 제공된다.
* https://github.com/awaitility/awaitility[Awaitility] (`org.awaitility:awaitility`) 가 이제 `spring-boot-starter-test` 의 일부가 되었다.
* 이제 `@JdbcTest` 및 `@DataJpaTest` 를 사용하여 테스트에서 auto-configure 된 `JdbcClient` bean을 사용할 수 있다.
* `MockMvc` 을 auto-configure 할 때 이제 filter는 등록 bean의 dispatcher type 및 init parameter를 사용하여 등록된다.
* 이제 Testcontainers를 병렬로 초기화할 수 있다. 이를 위해 `spring.testcontainers.beans.startup` 를 `parallel` 로 설정하면 된다.
* Micrometer observation을 지원하기 위한 `spring.kafka.template.observation-enabled` property 지원



=== Dependency Upgrades
Spring Boot 3.2.0 moves to new versions of the following Spring projects:

* Spring AMQP 3.1
* Spring Authorization Server 1.2
* Spring Batch 5.1
* Spring Data 2023.1
* Spring Framework 6.1
* Spring HATEOAS 2.2
* Spring Integration 6.2
* Spring Kafka 3.1
* Spring LDAP 3.2
* Spring Pulsar 1.0
* Spring Retry 2.0
* Spring Security 6.2
* Spring Session 3.2

Third-party dependencies have also been updated, the more noteworthy of which are the following:

* Artemis 2.29
* Brave 5.16
* Elasticsearch Client 8.10
* Flyway 9.22
* GraphQL Java 21.1
* Hibernate 6.3
* JUnit 5.10
* Jedis 5.0
* Kafka 3.6
* Kotlin 1.9
* Liquibase 4.24
* Log4j 2.21
* MariaDB 3.2
* Micrometer 1.12
* Micrometer Tracing 1.2
* Mockito 5.4
* Mongo Java Driver 4.11
* MySQL 8.1
* Neo4j Java Driver 5.10
* OkHttp 4.12
* OpenTelemetry 1.28
* Oracle UCP 23.3
* Rabbit AMQP Client 5.18.0
* Rabbit Stream Client 0.11
* Reactor 2023.0
* Selenium 4.14
* SnakeYAML 2.2



== Deprecations in Spring Boot 3.2.0

* OkHttp 3 지원은 더 이상 사용되지 않고 OkHttp 4로 대체되었다.
* `spring-boot:run`, `spring-boot:start` 및 `spring-boot:test-run` Maven goal의 `directories` property는 `additionalClasspathElements` 를 위해 더 이상 사용되지 않는다.
* 더 이상 사용되지 않는  `management.metrics.tags.*` 대신 `management.observations.key-values.*` 를 사용해야 한다.
* `LoggingSystemProperties` 및 `LogbackLoggingSystemProperties` 에 정의된 대부분의 constant는 enum value를 위해 더 이상 사용되지 않는다.
* `ClientHttpRequestFactorySettings` 및 `RestTemplateBuilder` 에서 활성화된 request buffering에 대한 지원은 더 이상 사용되지 않는다. 이 API는 더 이상 사용되지 않는 형태로 유지되지만 Spring Framework 6.1의 유사한 변경 사항에 따라 구성해도 아무런 영향을 미치지 않는다.
* 각 deletage를 프로그래밍 방식으로 spring.factories에 등록하는 것을 선호하기 때문에 `context.initializer.classes` environment property를 사용하여 추가 `ApplicationContextInitializer` 를 등록하는 것은 더 이상 사용되지 않는다.
* 각 delegate를 프로그래밍 방식으로 spring.factories에 등록하는 것을 선호하기 때문에 `context.listener.classes` environment property를 사용하여 추가 `ApplicationListener` 를 등록하는 것은 더 이상 사용되지 않는다.
* 확장 프로그램에서 관리되는 Flyway property들이 전용 namespace로 이동했다. 그 결과 `flyway.oracle*` property 들이 `flyway.oracle.*` 로 이동했다. 마찬가지로 `spring.flyway.sql-server-kerberos-login-file` 은 `spring.flyway.sqlserver.kerberos-login-file` 로 이동했다.
* https://github.com/influxdata/influxdb-client-java[새로운 InfluxDB Java client] 와 https://github.com/influxdata/influxdb-client-java/tree/master/spring[자체 Spring Boot 통합]을 위해 InfluxDB에 대한 지원은 더 이상 제공되지 않는다.
* `management.otlp.metrics.export.resource-attributes` configuration property가 더 이상 사용되지 않고 새로운 `management.opentelemetry.resource-attributes` 가 사용된다.
* `TaskExecutorBuilder` 가 `ThreadPoolTaskExecutorBuilder` 를 위해 더 이상 사용되지 않는다.
* `TaskSchedulerBuilder` 가 `ThreadPoolTaskSchedulerBuilder` 를 위해 더 이상 사용되지 않는다.
* Configuration property `spring.jms.listener.concurrency` 를 `spring.jms.listener.min-concurrency` 로 대체한다.
* Configuration property `spring.jms.listener.acknowledge-mode` 를 `spring.jms.listener.session.acknowledge-mode` 로 대체한다.
* `PlatformTransactionManagerCustomizer` 를 `TransactionManagerCustomizer` 로 대체한다.
* `TransactionManagerCustomizers(Collection<? extends PlatformTransactionManagerCustomizer<?>>)` 를 `TransactionManagerCustomizers#of(Collection<? extends TransactionManagerCustomizer<?>>)` 로 대체한다.
* propery 기반 initialization으로 `DelegatingApplicationContextInitializer` 및 `DelegatingApplicationListener` 를 사용하는 것은 더 이상 권장되지 않는다.
* `PemSslStoreBundle` 의 일부 오래된 생성자 및 `PemSslStoreDetails` 의 `certificate` accessor
* 
* `TaskExecutorCustomizer` 를 `ThreadPoolTaskExecutorCustomizer` 로 대체한다.
* `TaskSchedulerBuilder` 를 `ThreadPoolTaskSchedulerBuilder` 로 대체한다.
* `TaskSchedulerCustomizer` 를 `ThreadPoolTaskSchedulerCustomizer` 로 대체한다.
* `NettyWebServer` 의 일부 오래된 생성자
