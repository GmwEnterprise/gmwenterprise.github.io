---
title: SpringMvc处理全局异常
date: 2019-11-04 10:16:01
tags: java
---
处理全局异常的方法主要为两种：

- 实现`HandlerExceptionResolver`接口
- `@ControllerAdvice`注解

第一种方法的一个局限性是，必须返回`ModelAndView`实例，如果有返回Json数据需求的话，就力不从心了;

第二种方法我使用的比较多，通过给处理类加上`@ControllerAdvice`注解或`@RestControllerAdvice`注解，然后给类中的方法加上`@ExceptionHandler`注解并指定异常类型，便使得其中的方法成为了指定异常处理器。返回的类型类似于Controller，添加了`@ResponseBody`注解便会返回REST资源，不添加则返回视图。

假定这样的需求：以.json结尾的请求出现异常则返回Json数据；以.page结尾的请求出现异常则返回视图，那么只需要这么写：

```java
@RestControllerAdvice
public class SpringMvcExceptionAdvice {
  @ExceptionHandler(Throwable.class)
  public Object throwableHandler(HttpServletRequest request, Throwable exp) {
    if (request.getRequestURL().toString().endsWith(".json")) {
      return AjaxResponse.fail("请求JSON数据异常");
    }
    return new ModelAndView("errorPage");
  }
}
```

异常处理器的参数可以注入request，response，exception，类似于controller却有不同。
