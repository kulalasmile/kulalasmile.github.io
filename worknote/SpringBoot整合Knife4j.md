[TOC]
# SpringBoot整合Knife4j

## 概述

Knife4j 是springfox-swagger的增强UI实现，为Java开发者在使用Swagger的时候，能拥有一份简洁、强大的接口文档体验。

## 依赖

>  pom.xml：

```xml
<dependency>
	<groupId>com.github.xiaoymin</groupId>
	<artifactId>knife4j-spring-boot-starter</artifactId>
	<version>2.0.4</version>
</dependency>
```

注意：spring-boot-starter-parent的版本不要超过3.0，不然无法启动

## 配置

```java
@Configuration
@EnableSwagger2
@EnableKnife4j
@Import(BeanValidatorPluginsConfiguration.class)
public class Knife4jConfig {


    @Bean(value = "defaultApi2")
    public Docket defaultApi2() {
        Docket docket=new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                //分组名称
                .groupName("2.X版本")
                .select()
                //这里指定Controller扫描包路径
                .apis(RequestHandlerSelectors.basePackage("com.yvon.controller"))
                .paths(PathSelectors.any())
                .build();
        return docket;
    }

    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                .title("swagger-bootstrap-ui RESTful APIs")
                .description("swagger-bootstrap-ui")
                .termsOfServiceUrl("http://localhost:7777/")
                .version("1.0")
                .build();
    }
}
```

放行：

```yaml
      - /swagger-resources/**
      - /v2/api-docs
      - /v2/api-docs-ext
      - /doc.html
      - /webjars/**
```

## 运行

http://IP:PORT/doc.html 

![](https://gitee.com//kulalasmile/image/raw/master/img/20200714141821.png)