---
title: springboot配置FastJsonHttpMessageConverter(实战篇)
tags:
  - springboot
  - fastjson
  - http
  - springMVC
categories:
  - spring
abbrlink: 2029409209
date: 2018-01-22 15:12:00
---
# 背景
## 老的方式:controller的参数与返回值
```
public JsonResult queryDrillDetail(@RequestBody String body) {
List<Map<String, Object>> detailMapList = new ArrayList<>();
.......
return new JsonResult(JsonResult.SUCCESSFUL, new PagedList<>(query.getPage() + 1, pageable.getPageSize(),
                query.getTotalCount(), detailMapList));
}
```
## 老的方式:问题
- 入参不明确
- 返回值类型不明确
- Map的key只有在运行时,才会知道,且不知道类型

## 期望的方式:使用vo

# 改造步骤:
1.在使用converters的最后一个优先级,添加FastJsonHttpMessageConverter

```
@Component
public class MyWebAppConfigurer extends WebMvcConfigurerAdapter {

    // 添加converter的第三种方式
    // 同一个WebMvcConfigurerAdapter中的configureMessageConverters方法先于extendMessageConverters方法执行
    // 可以理解为是三种方式中最后执行的一种，不过这里可以通过add指定顺序来调整优先级，也可以使用remove/clear来删除converter，功能强大
    // 使用converters.add(xxx)会放在最低优先级（List的尾部）
    // 使用converters.add(0,xxx)会放在最高优先级（List的头部）
    @Override
    public void extendMessageConverters(List<HttpMessageConverter<?>> converters) {
        converters.add(new FastJsonHttpMessageConverter());
    }

}
```

2.model遵循老的接口原则,转为下划线分割的

```
@JSONType(naming = PropertyNamingStrategy.SnakeCase)
public class PurchaseDetailDrillVo {

    private Long pmsPurchaseOrderId;
    private Long pmsPurchaseOrderDetailId;
    private String pmsPurchaseOrderNo;
    private Integer productType;
    private String deliverAddress;
    private Long provinceId;
    ....
}
```

3.使用@RequestBody注解,类型改为vo

测试,接受参数和返回参数都是vo,而接口的字段映射依然为下划线,


------
以下为项目本身的问题
4.pom文件中排除jackson的依赖
```
        <dependency>
            <groupId>com.ejlerp</groupId>
            <artifactId>ejlerp-common</artifactId>
            <version>3.2.0-SNAPSHOT</version>
            <exclusions>
                <exclusion>
                    <artifactId>jackson-core</artifactId>
                    <groupId>com.fasterxml.jackson.core</groupId>
                </exclusion>
            </exclusions>
        </dependency>
```

5.fastjason默认支持所有的contentType,及*, 与springmvc校验相冲突,so需要声明fastjason能支持的contentType,在初始化FastJsonHttpMessageConverter,注入参数
```
    @Override
    public void extendMessageConverters(List<HttpMessageConverter<?>> converters) {
        FastJsonHttpMessageConverter fastJsonHttpMessageConverter = new FastJsonHttpMessageConverter();
        ArrayList<MediaType> supportedMediaTypes = Lists.newArrayList();
        supportedMediaTypes.add(MediaType.APPLICATION_FORM_URLENCODED);
        supportedMediaTypes.add(MediaType.APPLICATION_JSON);
        supportedMediaTypes.add(MediaType.TEXT_HTML);
        supportedMediaTypes.add(MediaType.TEXT_PLAIN);
        fastJsonHttpMessageConverter.setSupportedMediaTypes(supportedMediaTypes);
        converters.add(fastJsonHttpMessageConverter);
    }
```





# fastjson官网
https://github.com/alibaba/fastjson/issues/1555
https://github.com/alibaba/fastjson/wiki/PropertyNamingStrategy_cn