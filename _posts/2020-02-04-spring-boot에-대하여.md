---
layout: post
title: spring boot에 대하여                            # Title of the page
hide_title: false                                  # Hide the title when displaying the post, but shown in lists of poststhumbnail: "assets/img/thumbnails/sample-th.png"  # Add 
color: brown                             # Add the specified color as feature image, and change link colors in post
author: "Mun Soo Kim"
tags: [기술, spring]
---

spring boot는 순수 spring으로만 개발할 때 개발자가 직접 해야 할 아주 많은 일들을 대신 해줌으로써 생산성을 많이 높여줍니다. 그럼 spring boot가 개발자들을 위해서 어떤 것들을 해주는지 살펴보겠습니다. 

### 의존성 관리
---
spring boot는 주요 프로젝트들의 의존성을 편리하게 관리해줍니다. 일반적으로 pom.xml파일에 dependency를 추가할 때는 버전 정보를 함께 명시해야 하는데요. spring boot의 dependency를 보시면 버전 정보가 없는 것을 확인하실 수 있습니다. 

* pom.xml
    ```xml
    ...
    <parent>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-parent</artifactId>
            <version>2.3.8.RELEASE</version>
            <relativePath/> <!-- lookup parent from repository -->
    </parent>
    ...
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    <dependencies>
    ```
    위에서 확인할 수 있듯이 버전에 대한 정보를 명시하지 않고 있습니다. spring boot가 알아서 적절한 버전을 가져와주기 때문입니다. 각 프로젝트들의 버전 정보는 parent태그에 명시된 spring-boot-starter-parent artifact에 정의되어 있습니다. 맥은 command를, 윈도우는 control키를 누르고 artifact명을 클릭하면 아래의 파일에 접근하게 됩니다.

<br/>

* spring-boot-starter-parent-2.3.8.RELEASE.pom
    ```xml
    ...
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-dependencies</artifactId>
        <version>2.3.8.RELEASE</version>
    </parent>
    ...
    ```
    여기서 다시 한 번 spring-boot-dependencies를 타고 들어가봅니다.

<br/>

* spring-boot-dependencies-2.3.8.RELEASE.pom
    ```xml
    ...
    <dependencyManagement>
        ...
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <version>2.3.8.RELEASE</version>
        </dependency>
        ...
    </dependencyManagement>
    ...
    ```
    네 드디어 도착입니다. 위 파일의 dependencyManagement태그에 각 프로젝트들에 대한 버전 정보를 명시해두고 있습니다. 저희가 pom.xml에 직접 추가하는 의존성들에 대한 버전은 이 파일에 명시된 버전으로 설정됩니다. 즉 ***pom.xml파일은 위의 부모 설정들을 상속받아서 버전을 정의하게 되는 겁니다.*** 

그리고 이렇게 버전을 대신 관리해주게 되면 의존성 간의 버전 호환성에 대한 걱정도 할 필요가 없어집니다. spring boot가 의존성을 추가할 때 알아서 설정된 버전과 호환되는 버전으로 추가를 해주기 때문입니다.

만약 pom.xml파일에 의존성을 추가할 때 개발자가 직접 버전을 명시하게 되면 부모 설정을 오버라이딩하여 개발자가 명시한 버전으로 의존성을 추가하게 됩니다.


### @SpringBootApplication
---
spring은 xml이나 자바 설정을 통해서 빈을 ioc컨테이너에 등록할 수 있습니다. 특히 자바 설정을 이용할 때는 자바 config파일을 만들어서 @Configuration, @ComponentScan과 같은 annotation을 사용해서 아래와 같이 만들 수 있습니다.

* AppConfig.java(제가 임의로 만든 파일입니다. spring boot프로젝트에는 존재하지 않습니다.)
    ```java
    @Configuration
    @ComponentScan(basePackageClass = SampleTicketApplication.class)
    public class AppConfig {
        
    }
    ```
    위의 파일을 패키지의 루트에 위치시키면 그 파일과 같은 depth에 위치한 자바 파일과 패키지들에 존재하는 자바 파일까지 스캔을 하면서 @Controller, @Service, @Configuration, @Repository와 같은 컴포넌트 annotation이 붙은 클래스들을 모두 빈으로 등록하게 됩니다. 

    하지만 spring boot프로젝트를 만들면 위와 같은 설정 파일이 없음에도 불구하고 빈들이 잘 등록됩니다. 그 역할을 @SpringBootApplication이 하게 됩니다. 이 annotation은 프로젝트를 처음 생성하면 루트 패키지의 Application 클래스에 붙게 됩니다. 

<br />

* SampleTicketApplication.java
    ```java
    @SpringBootApplication
    public class SampleTicketApplication {
        public static void main(String[] args) {
            SpringApplication.run(SampleTicketApplication.class, args);
        }
    }
    ```
    spring boot프로젝트를 처음 생성하면 패키지의 루트에 위치하는 Application 클래스 입니다. 이 클래스에서 애플리케이션을 실행하게 되죠. @SpringBootApplication을 타고 들어가보면 아래와 같은 파일이 나오게 됩니다.

<br />

* SpringBootApplication.java
    ```java
    @Target({ElementType.TYPE})
    @Retention(RetentionPolicy.RUNTIME)
    @Documented
    @Inherited
    @SpringBootConfiguration
    @EnableAutoConfiguration
    @ComponentScan(
        excludeFilters = {@Filter(
        type = FilterType.CUSTOM,
        classes = {TypeExcludeFilter.class}
    ), @Filter(
        type = FilterType.CUSTOM,
        classes = {AutoConfigurationExcludeFilter.class}
    )}
    )
    public @interface SpringBootApplication {
        ...
    }
    ```
    @SpringBootApplication에 @ComponentScan과 @SpringBootConfiguration이 붙은 것을 확인하실 수 있습니다. @SpringBootConfiguration을 타고 들어가보시면 @Configuration이 붙은 것을 확인하실 수 있습니다. 즉 ***@SpringBootApplication이 붙은 Application 클래스가 빈을 등록하기 위한 자바 설정 파일의 역할을 겸하고 있는 것입니다.*** 

### 자동 설정
---
@SpringBootApplication을 통해서 컴포넌트 annotation이 붙은 클래스들을 알아서 빈으로 등록해준다는 것을 확인했습니다. 그런데 이렇게 개발자가 직접 정의한 빈 이외에도 spring boot가 정의해둔 유용한 빈들이 ioc컨테이너에 추가됩니다. 이 기능도 @SpringBootApplication에 붙은 ***@EnableAutoConfiguration*** 때문에 동작합니다. 

그럼 이 자동으로 등록되는 빈은 어디에서 스캔되는 걸까요? 
<div>
    <img width="400" alt="스크린샷 2021-02-04 오후 11 44 40" src="https://user-images.githubusercontent.com/56672937/106908903-12c06300-6743-11eb-9ad0-f8400cbcf2c6.png">
</div>
<div>
    <img width="539" alt="스크린샷 2021-02-04 오후 11 47 14" src="https://user-images.githubusercontent.com/56672937/106909149-561ad180-6743-11eb-9b5b-4658129f9475.png">
</div>
IntelliJ의 좌측 프로잭트 구조에서 하단의 External Libraries에 여러 의존성들 중 위의 이미지에 보이는 spring-boot-autoconfigure프로젝트의 spriing.factories파일에 명시되어 있습니다.

* spring.factories

    ```tex
    # Auto Configure
    org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
    ...
    org.springframework.boot.autoconfigure.web.servlet.MultipartAutoConfiguration,\
    org.springframework.boot.autoconfigure.web.servlet.WebMvcAutoConfiguration,\
    org.springframework.boot.autoconfigure.websocket.reactive.WebSocketReactiveAutoConfiguration,\
    org.springframework.boot.autoconfigure.websocket.servlet.WebSocketServletAutoConfiguration,\
    org.springframework.boot.autoconfigure.websocket.servlet.WebSocketMessagingAutoConfiguration,\
    ...
    ```
    위와 같이 EnableAutoConfiguration에 명시된 Configuration파일들이 빈으로 등록됩니다. 대표적으로 WebMvcAutoConfiguration만 살펴보겠습니다.

<br />

* WebMvcAutoConfiguration.java
    ```java
    @Configuration(
        proxyBeanMethods = false
    )
    @ConditionalOnWebApplication(
        type = Type.SERVLET
    )
    @ConditionalOnClass({Servlet.class, DispatcherServlet.class, WebMvcConfigurer.class})
    @ConditionalOnMissingBean({WebMvcConfigurationSupport.class})
    @AutoConfigureOrder(-2147483638)
    @AutoConfigureAfter({DispatcherServletAutoConfiguration.class, TaskExecutionAutoConfiguration.class, ValidationAutoConfiguration.class})
    public class WebMvcAutoConfiguration {

        ...

        @Configuration(
            proxyBeanMethods = false
        )
        public static class EnableWebMvcConfiguration extends DelegatingWebMvcConfiguration implements ResourceLoaderAware {

            ...

            @Bean
            public RequestMappingHandlerAdapter requestMappingHandlerAdapter(@Qualifier("mvcContentNegotiationManager") ContentNegotiationManager contentNegotiationManager, @Qualifier("mvcConversionService") FormattingConversionService conversionService, @Qualifier("mvcValidator") Validator validator) {
                RequestMappingHandlerAdapter adapter = super.requestMappingHandlerAdapter(contentNegotiationManager, conversionService, validator);
                adapter.setIgnoreDefaultModelOnRedirect(this.mvcProperties == null || this.mvcProperties.isIgnoreDefaultModelOnRedirect());
                return adapter;
            }

            ...

            @Bean
            @Primary
            public RequestMappingHandlerMapping requestMappingHandlerMapping(@Qualifier("mvcContentNegotiationManager") ContentNegotiationManager contentNegotiationManager, @Qualifier("mvcConversionService") FormattingConversionService conversionService, @Qualifier("mvcResourceUrlProvider") ResourceUrlProvider resourceUrlProvider) {
                return super.requestMappingHandlerMapping(contentNegotiationManager, conversionService, resourceUrlProvider);
            }

            ...
        }
    }
    ```
    위 설정 파일을 보면 @Configuration이 붙어있습니다. 즉 spring.factories에 명시된 설정 파일을 빈으로 등록하고 그 설정 파일에 있는 빈들이 등록되는 것입니다. 여러 빈들 중 HandlerMapping, HandlerAdapter와 같이 spring mvc가 제공하는 dispatcher servlet의 전략 인터페이스들이 빈으로 등록되는 것을 확인할 수 있습니다. 

정리해보면 spring boot는 사용자가 정의한 빈을 등록하고 자동 설정에 등록된 빈을 등록하는 두 단계를 거치게 되는 겁니다.

### 내장 was
spring boot는 was가 프로젝트에 내장되어 있습니다. 즉 jar로 패키징을 해서 실행을 하는 것만으로 서버에 애플리케이션을 바로 띄울 수 있게 됩니다. 이 기능도 앞서 살펴본 자동 설정과 관련됩니다. 

* spring.factories

    ```tex
    # Auto Configure
    org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
    ...
    org.springframework.boot.autoconfigure.web.servlet.ServletWebServerFactoryAutoConfiguration,\
    ...
    ```
    앞서 살펴본 spring.factories파일에 명시된 ServletWebServerFactoryAutoConfiguration.java에 내장 was와 관련된 빈들이 등록되어 있습니다. 

### 마치며
지금까지 spring boot가 어떤 역할을 하는지 몇 가지 살펴봤는데요. 저의 글이 조금이나마 도움이 됐으면 좋겠습니다. 이외에도 spring boot가 우리에게 제공해주는 효용은 더 많습니다. 저도 공부를 하는 입장이다보니 다 알지 못 하고 틀린 내용이 있을 수 있습니다. 틀린 내용이 있거나 다른 좋은 기능이 있다면 댓글로 알려주시면 굉장히 감사할 것 같습니다. 긴 글 읽어주셔서 감사합니다~!