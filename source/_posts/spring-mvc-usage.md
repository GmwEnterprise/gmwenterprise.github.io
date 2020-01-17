---
title: spring-mvc常用方式总结
date: 2020-01-16 14:35:47
tags:
  - java
  - spring
---
> 本文用于记录自己在学习中、工作中所用到的、了解到的spring-mvc的常用用法、配置等。只记录spring-boot中的配置方式，不再记录xml配置方式。

使用spring-boot开发web项目，引入spring-boot-starter-web依赖后，项目便集成了默认的mvc功能，提供基本的api访问功能。

如果想要自定义配置，定义一个配置类并实现WebMvcConfigurer接口，配置@EnableWebMvc注解即可：
``` java
@Configuration
@EnableWebMvc
public class ConfigWeb implements WebMvcConfigurer {
  // 在这里可以实现多种配置
}
```

## 配置静态资源访问路径

默认的spring-boot配置是定义'/'作为dispatchServlet的拦截路径的，所有的访问都会被拦截（包括静态资源），交由spring-mvc来处理。可以通过添加资源处理器来单独配置静态资源的访问：
``` java
@Override
public void addResourceHandlers(ResourceHandlerRegistry reg) {
  // addResourceHandler -> 配置访问路径: http://{host}:{port}/{context-path}/resources/**
  // addResourceLocations -> 配置静态资源的存放路径
  reg.addResourceHandler("/resources/**").addResourceLocations("classpath:static/");
  // 当存在静态资源与控制器路径冲突时，配置该资源访问的优先顺序。默认为Integer.MAX_VALUE - 1即最低优先级
  // 冲突时优先访问控制器。值越小优先级越高
  reg.setOrder(Integer.MAX_VALUE - 1);
}
```

## 配置跨域

可以单独针对一个Controller或一个控制器方法进行配置，直接添加注解@CrossOrigin即可；这里提供全局配置的方式：
``` java
@Override
public void addCorsMappings(CorsRegistry registry) {
  registry
    .addMapping("/**")
    .allowedMethods("GET", "POST", "PATCH", "PUT", "DELETE", "OPTION");
    .allowCredentials(true);
}
```

## 处理消息转换

springmvc默认提供了基本的消息转换工具。对于想要自定义JSON格式的消息转换/序列化，只需要提供覆盖方法就能实现：
``` java
@Override
public void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
  // 这个方法执行时，converters是为空的
  // 如果这个方法返回之后converters依然为空，或者不覆盖该方法，springmvc就会默认提供一个消息转换队列
  converters.add(... /* 自定义消息转换器 */);
  log.info("configureMessageConverters completed: {}", converters);
}

@Override
public void extendMessageConverters(List<HttpMessageConverter<?>> converters) {
  // 这个方法是在configureMessageConverters执行之后执行的，converters里要么是自己配置的转换器，要么是系统默认提供的
  // 如果想保留系统默认提供的消息转换且额外增加一些规则，请使用该方法
  log.info("extendMessageConverters completed: {}", converters);
}
```

## 实现java8时间类的全局配置

### 响应数据

首先是响应数据。对于rest资源我们默认提供json转换器，但默认的消息转换器对于日期处理的格式不友好。所以此处进行自订处理：
``` java
@Override
public void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
  converters.add(new MappingJackson2HttpMessageConverter(provideObjectMapper()));
}

@Bean(name = "objectMapper")
ObjectMapper provideObjectMapper() {
  var mapper = new ObjectMapper();
  // 注册扩展模块
  mapper.registerModule(generateJavaTimeModule());
  return mapper;
}

// jsr310所提供的对于Java8时间处理的模块类
@Bean(name = "javaTimeModule")
JavaTimeModule generateJavaTimeModule() {
  var module = new JavaTimeModule();
  module.addSerializer(LocalDateTime.class, new LocalDateTimeSerializer(DATE_TIME_FORMATTER));
  module.addSerializer(LocalDate.class, new LocalDateSerializer(DATE_FORMATTER));
  module.addSerializer(LocalTime.class, new LocalTimeSerializer(TIME_FORMATTER));
  module.addDeserializer(LocalDateTime.class, new LocalDateTimeDeserializer(DATE_TIME_FORMATTER));
  module.addDeserializer(LocalDate.class, new LocalDateDeserializer(DATE_FORMATTER));
  module.addDeserializer(LocalTime.class, new LocalTimeDeserializer(TIME_FORMATTER));
  return module;
}

// 使用具有可读性的时间格式
public static final String DATE_TIME_PATTERN = "yyyy-MM-dd HH:mm:ss";
public static final String DATE_PATTERN = "yyyy-MM-dd";
public static final String TIME_PATTERN = "HH:mm:ss";
public static final DateTimeFormatter DATE_TIME_FORMATTER = DateTimeFormatter.ofPattern(DATE_TIME_PATTERN);
public static final DateTimeFormatter DATE_FORMATTER = DateTimeFormatter.ofPattern(DATE_PATTERN);
public static final DateTimeFormatter TIME_FORMATTER = DateTimeFormatter.ofPattern(TIME_PATTERN);
public static final SimpleDateFormat SIMPLE_DATE_FORMAT = new SimpleDateFormat(DATE_TIME_PATTERN);
```

### 请求数据

时间类型在请求中出现的类型无非为两种，一种是出现在请求体（@RequestBody），默认由消息转换处理；另一种就是表单参数。对于表单参数的时间类型也有多种处理模式，这里我选择使用全局处理的方式：
``` java
// 声明为bean是为了能在构造函数中注入FormattingConversionService
@Component
class ConvertersInitializer {
    @Autowired
    public ConvertersInitializer(FormattingConversionService fcs) {
        // 通过这个类来添加参数转换
        fcs.addConverter(new String2Date());
        fcs.addConverter(new String2LocalDate());
        fcs.addConverter(new String2LocalTime());
        fcs.addConverter(new String2LocalDateTime());
    }

    private static class String2LocalDateTime implements Converter<String, LocalDateTime> {
        @Override
        public LocalDateTime convert(String source) {
            return LocalDateTime.parse(source, DATE_TIME_FORMATTER);
        }
    }

    private static class String2LocalDate implements Converter<String, LocalDate> {
        @Override
        public LocalDate convert(String source) {
            return LocalDate.parse(source, DATE_FORMATTER);
        }
    }

    private static class String2LocalTime implements Converter<String, LocalTime> {
        @Override
        public LocalTime convert(String source) {
            return LocalTime.parse(source, TIME_FORMATTER);
        }
    }

    private static class String2Date implements Converter<String, Date> {
        @Override
        public Date convert(String source) {
            try {
                return SIMPLE_DATE_FORMAT.parse(DATE_TIME_PATTERN);
            } catch (ParseException e) {
                e.printStackTrace();
                return null;
            }
        }
    }
}
```

## 配置参数校验

## 返回出参拦截

实现接口ResponseBodyAdvice