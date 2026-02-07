# JSON 转换冲突问题

> 由于在 Spring Boot 中，默认的 JSON 转换器是 Jackson。但是项目中又引入了 fastjson ，导致接口返回 JSON 中出现类似 {"$ref":"$.list[0]"} 这种“引用”现象

## 解决方案

增加 JsonConverterConfig 配置类，强制让 Spring MVC 使用且仅使用 Jackson 作为 HTTP 消息转换器

```java
@Configuration
public class JsonConverterConfig {

  @Bean
  @Primary
  public HttpMessageConverters jacksonOnly() {
    return new HttpMessageConverters(
        new MappingJackson2HttpMessageConverter()
    );
  }
}
```

但是上述代码会导致最终的 HttpMessageConverter 中会存在两个 MappingJackson2HttpMessageConverter，以下是优化后的代码

```java
@Configuration
public class JsonConverterConfig implements WebMvcConfigurer {

  @Override
  public void extendMessageConverters(List<HttpMessageConverter<?>> converters) {
    // 1. 彻底移除所有的 Fastjson 转换器
    // 不管它是通过什么手段挤进来的，直接按类名查杀
    converters.removeIf(converter ->
        converter.getClass().getName().toLowerCase().contains("fastjson"));

    // 2. 此时列表里剩下的就是 Spring 默认的 Jackson 和其他转换器（String, Byte等）
    // 如果你想确保 Jackson 优先级最高，可以把它移到第一位
    for (int i = 0; i < converters.size(); i++) {
      if (converters.get(i) instanceof MappingJackson2HttpMessageConverter) {
        HttpMessageConverter<?> jacksonConverter = converters.remove(i);
        converters.add(0, jacksonConverter);
        break;
      }
    }
  }
}
```

此外，对于使用 `fastjson @JSONField(format = "yyyy-MM-dd")` ，要替换成 `@JsonFormat(pattern = "yyyy-MM-dd", timezone = "GMT+8")` ，否则返回前端的日期类型不会被正确格式化，注意时区
