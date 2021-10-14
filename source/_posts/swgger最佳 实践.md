---
title: swagger最佳实践
top: false
cover: false
toc: true
summary: swagger最佳实践
categories: 技术实践
tags: swagger
abbrlink: 10004
date: 2021-10-14 20:10:10
img:
password:
---

## 基础配置
- 在pom.xml引入如下依赖，1.9.6是swagger-bootstrap-ui的最后一个版本，适配了springfox-swagger2:2.9.2版本。可参照下面配置

```xml
<dependency>
     <groupId>io.springfox</groupId>
     <artifactId>springfox-swagger2</artifactId>
     <version>2.9.2</version>
</dependency>
<dependency>
     <groupId>com.github.xiaoymin</groupId>
     <artifactId>swagger-bootstrap-ui</artifactId>
     <version>1.9.6</version>
</dependency>
```

- Swagger2Config文件配置，可以在项目的conf包下新建Swagger2Config.java文件，配置参考如下

```java
@Configuration
@EnableSwagger2
@EnableSwaggerBootstrapUI
public class Swagger2Config {

    @Bean
    public Docket createRestApi() {
        return new Docket(DocumentationType.SWAGGER_2)
                .groupName("1.0.0")
                .apiInfo(apiInfo())
                .select()
                // 扫描所有有注解的api，用这种方式更灵活
                .apis(RequestHandlerSelectors.withMethodAnnotation(ApiOperation.class))
                .paths(PathSelectors.any())
                .build();
    }

    @Bean
    public Docket createRestApiV2(){
        return new Docket(DocumentationType.SWAGGER_2)
                .groupName("2.0.0")
                .apiInfo(apiInfoV2())
                .select()
                .apis(RequestHandlerSelectors.withMethodAnnotation(ApiV2Version.class))
                .paths(PathSelectors.any())
                .build();
    }

    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                .title("接口文档")
                .description("接口文档")
                .version("1.0.0")
                .build();
    }

    private ApiInfo apiInfoV2() {
        return new ApiInfoBuilder()
                .title("V2.0接口文档")
                .description("V2.0接口文档")
                .version("2.0.0")
                .build();
    }

}
```

- 配置一个API：在您的controller层的API使用`@ApiOperation`注解标识即可，参照文件如下：

```java
@RestController
@RequestMapping("user")
@Api(tags = {"用户信息API接口"})
@ApiSort(value =1)
public class UserController extends BaseController {
    @Autowired
    KeyPairRedis keyPairRedis;
    @Autowired
    IUserService userService;

    /**
     * 获取用户信息
     * <p>这里我暴露了一个获取用户信息的接口
     *
     * @return
     */
    @GetMapping("getInfo")
    @ApiOperation("获取用户信息")
    public ResultDTO<UserInfoDTO> getInfo() {
        Long userId = SsoHelper.getUserId(redisTemplate, request);
        UserInfoDTO userInfoDTO = userService.findByUserId(userId);
        return ResultDTO.requstSuccess(userInfoDTO);
    }
}
```

## 如何访问
访问您的应用下的/doc.html即可。如：http://127.0.0.1:8080/doc.html
如果访问不到，检查一下拦截器, 可能是资源权限未开放，请放行对应资源，参照如下：

```java
@Configuration
public class InterceptorConf implements WebMvcConfigurer {

	@Bean
    public LoginInterceptor loginInterceptor() {
        return new LoginInterceptor();
    }
   
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(loginInterceptor())
                .addPathPatterns("/**")
                 // swagger-ui 升级版, 线上时关闭
                 .excludePathPatterns("/doc.html")
                 .excludePathPatterns("/**/swagger-resources/**")
                 .excludePathPatterns("/v2/**");
     }
}
```

好的，一般情况下到这儿已经可以了。