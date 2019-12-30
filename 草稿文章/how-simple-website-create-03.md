---
title: 简单的网站搭建-03
date: 2019-11-24 00:22:01
tags:
  - spring
  - java
---

### 针对上一篇文章的一些修改

#### 权限相关代码修改

为了更好地获取验证信息，我修改了上一章中关于权限的一部分代码。首先是新增了`Authentication`类与`Authorization`类，用来进一步封装验证信息：

Authentication:

``` java
package cn.gmwenterprise.presevere.config.security;
import com.fasterxml.jackson.annotation.JsonFormat;
import java.time.LocalDateTime;
// 登录凭据
public class Authentication {
    // 登陆时间
    private LocalDateTime loginDatetime;
    // 登录IP地址
    private String loginIp;
    // 用户信息主键
    private Integer userId;
    // 登录平台
    private Platform platform;
    // 过期时间[分钟]
    private Long timeout;
    @JsonFormat(shape = JsonFormat.Shape.STRING)
    public enum Platform { BROWSER, ANDROID, IOS }
    // 无参构造、有参构造、getter、setter
}
```

Authorization:

``` java
package cn.gmwenterprise.presevere.common;

import cn.gmwenterprise.presevere.domain.SysUser;
import cn.gmwenterprise.presevere.vo.Authentication;

public class Authorization {
    // 验证时解析token值后将其放在这里
    private String token;
    // 解析出userId后查询当前用户信息置于此
    private SysUser currentUser;
    // 凭据
    private Authentication authentication;
    // 无参构造、有参构造、getter、setter
}
```

(不要吐槽我的命名...)

然后修改了`AuthorizationHolder`类：

``` java
package cn.gmwenterprise.presevere.config.security;

public final class AuthorizationHolder {
    private static final ThreadLocal<Authorization> AUTHORIZATION = new ThreadLocal<>();

    public static void set(Authorization authorization) {
        if (AUTHORIZATION.get() == null) {
            AUTHORIZATION.set(authorization);
        }
    }

    public static Authorization get() {
        return AUTHORIZATION.get();
    }

    public static boolean exist() {
        return AUTHORIZATION.get() != null;
    }

    public static void remove() {
        AUTHORIZATION.remove();
    }
}
```

修改过后的验证信息相关类看起来信息更全面，在使用时也更加方便。最后将`AuthenticationInterceptor`拦截器也修改：

``` java
package cn.gmwenterprise.presevere.config.security;

import cn.gmwenterprise.presevere.common.TokenHelper;
import cn.gmwenterprise.presevere.domain.SysUser;
import cn.gmwenterprise.presevere.service.UserService;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Component;
import org.springframework.util.StringUtils;
import org.springframework.web.servlet.HandlerInterceptor;

import javax.annotation.Resource;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.time.LocalDateTime;

@Component
public class AuthenticationInterceptor implements HandlerInterceptor {
    private static final Logger log = LoggerFactory.getLogger(AuthenticationInterceptor.class);

    @Resource
    UserService userService;

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        String token = request.getHeader("Authorization");
        if (!StringUtils.isEmpty(token)) {
            log.info("Authorization: [{}]", token);
            // 解析的数据修改为了Authentication
            Authentication authentication = TokenHelper.parseToken(token, Authentication.class);
            if (validateAuthentication(authentication, request)) {
                // token验证通过后，AuthorizationHolder中也有了值
                Authorization authorization = AuthorizationHolder.get();
                authorization.setAuthentication(authentication);
                authorization.setToken(token);
            }
        }
        return true;
    }

    /**
     * 验证token
     */
    private boolean validateAuthentication(Authentication authentication, HttpServletRequest request) {
        // 没有凭据或无法解析
        if (authentication == null) {
            return false;
        }

        // 凭据生成ip地址与当前请求来源不相同
        if (!request.getRemoteHost().equals(authentication.getLoginIp())) {
            return false;
        }

        // TODO 判断客户端属于哪个Platform

        // 如果timeout值为0就不过期
        if (authentication.getTimeout() > 0L) {
            // 设置了过期时间
            LocalDateTime expireTime = authentication.getLoginDatetime().plusMinutes(authentication.getTimeout());
            if (!expireTime.isAfter(LocalDateTime.now())) {
                // 过期了
                return false;
            }
        }

        // 判断用户信息
        if (authentication.getUserId() != null) {
            SysUser currentUser = userService.getUserById(authentication.getUserId());
            if (currentUser == null) {
                return false;
            }
            // 代码走到这一行即通过了验证，生成凭据并存储到Holder中
            Authorization authorization = new Authorization();
            authorization.setCurrentUser(currentUser);
            AuthorizationHolder.set(authorization);
            return true;
        } else {
            return false;
        }
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        // 请求完成后移除凭据
        AuthorizationHolder.remove();
    }
}
```

#### 其他代码的修改

新增了一个返回状态的枚举`Res`：

``` java
package cn.gmwenterprise.presevere.vo;

public enum Res {

    /**
     * [1] 请求成功
     */
    OK(1, "请求成功"),

    /**
     * [2] 请求失败
     */
    ERROR(2, "请求失败"),

    /**
     * [3] 需要登录
     */
    LOGIN_REQUIRED(3, "需要登录"),

    /**
     * [4] 当前用户权限不足
     */
    UNAUTHORIZED(4, "权限不足");

    private int code;
    private String msg;

    Res(int code, String msg) {
        this.code = code;
        this.msg = msg;
    }

    public int getCode() {
        return code;
    }

    public String getMsg() {
        return msg;
    }
}
```

然后修改一下`AjaxResult`的几个静态方法：

``` java
public final class AjaxResult {
    public static AjaxResult ok(Object data) {
        return new AjaxResult(Res.OK.getCode(), Res.OK.getMsg(), data);
    }
    public static AjaxResult error(Object data) {
        return new AjaxResult(Res.ERROR.getCode(), Res.ERROR.getMsg(), data);
    }
    public static AjaxResult res(Res status) {
        return new AjaxResult(status.getCode(), status.getMsg(), null);
    }
    public static AjaxResult res(Res status, Object data) {
        return new AjaxResult(status.getCode(), status.getMsg(), data);
    }
}
```

同时对于`SecurityInterceptor`类也进行了相关修改：

``` java
    public boolean preHandle(...) throws Exception {
        ......省略
                return noAccess(response, "当前用户无访问权限", Res.UNAUTHORIZED);
            }
            return noAccess(response, "禁止访问，需要登陆", Res.LOGIN_REQUIRED);
        }
        return true;
    }

    private boolean noAccess(HttpServletResponse response, String errorMsg, Res status) throws IOException {
        ......
        writer.write(objectMapper.writeValueAsString(AjaxResult.res(status, errorMsg)));
        ......
    }
```

这样写起来更方便。

### 全局异常处理

新增一个异常类，用于在业务逻辑中显示抛出：

``` java
package cn.gmwenterprise.presevere.common;

/**
 * 业务异常，代码中显式抛出，全局处理器统一处理
 */
public class BusinessException extends RuntimeException {
    public BusinessException() { }
    public BusinessException(String message) {
        super(message);
    }
    public BusinessException(String message, Throwable cause) {
        super(message, cause);
    }
    public BusinessException(Throwable cause) {
        super(cause);
    }
    public BusinessException(String message, Throwable cause, boolean enableSuppression, boolean writableStackTrace) {
        super(message, cause, enableSuppression, writableStackTrace);
    }
}
```

然后创建SpringMVC全局异常处理器：

``` java
package cn.gmwenterprise.presevere.common;

import cn.gmwenterprise.presevere.vo.AjaxResult;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;

@RestControllerAdvice
public class ExceptionManagement {
    private static final Logger log = LoggerFactory.getLogger(ExceptionManagement.class);

    // 当运行时检测到了代码抛出了这个异常，则由此处进行捕获，并作出相应处理
    // 这里的处理十分简单，可以根据情况做更多复杂处理
    @ExceptionHandler(BusinessException.class)
    public AjaxResult businessExceptionHandler(BusinessException exp) {
        log.error(exp.getMessage());
        return AjaxResult.error(exp.getMessage());
    }
}
```

### 登录、注册功能，后台实现

#### 文字说明

前后端分离项目，登录信息实际上存储在客户端，也就是浏览器，服务器不保存登录信息，所以需要令牌这样一个东西，在登陆程序验证身份通过后授予客户端令牌，之后每一次客户端的请求都会携带令牌，服务器对其进行检测。也存在一些不需要令牌的请求，则忽略令牌。

登录、注册这两个功能分别作为两个接口，自然是不需要进行验证的。所以系统会新增两个请求接口：login(登录)、register(注册)。

登录会需要接收loginName、password作为验证信息，验证通过则生成令牌返回给客户端；注册同样会接收这两个参数，成功注册信息后同时也生成令牌返回给客户端，即注册成功自动登录。

在我的前后端程序中，登录、注册都会通过两个接口来实现。登录系统时，首先发送第一个接口，用于验证登录用户名[loginName/username]是否存在，若不存在则返回错误信息提示用户名不存在，若存在则返回成功信息，客户端接收到错误信息就直接弹出错误提示，若接收到成功信息就发送第二个请求，同时携带username和password，后端接口验证通过后返回令牌，客户端收到令牌后保存到客户端，然后执行登录成功后的操作；注册系统时，同样也需要发送第一个接口并只携带username，用于生成用户初始数据以及每个人都不同的密码盐，因为在登录时会判断用户是否有加密盐从而决定发送后台的密码是否加密，前台获取到生成的盐后将密码进行加盐加密，并发送第二个注册请求，并携带username、password，收到注册成功后返回的令牌执行与登录操作相同的事情。

加密程序比较简单，暂时没有做复杂的工作，但是有了这个概念。前端使用的加密算法很简单，MD5(Password + Salt)，后台第二次加密核对数据库则是用的spring-security提供的BCryptPasswordEncoder工具进行bcrypt加密。前端的加密是为了防止http密码明文传输，后台再次加密是为了防止数据库泄露导致密码丢失。真正的安全系统使用的加密手段更强大，以后可做改进，flag先立好。

#### 后端代码

创建`LoginController`类。

``` java
package cn.gmwenterprise.presevere.controller;

import cn.gmwenterprise.presevere.config.security.AuthRequire;
import cn.gmwenterprise.presevere.domain.SysUser;
import cn.gmwenterprise.presevere.dto.DtoSign;
import cn.gmwenterprise.presevere.service.LoginService;
import cn.gmwenterprise.presevere.vo.AjaxResult;
import cn.gmwenterprise.presevere.vo.LoginSuccess;
import cn.gmwenterprise.presevere.vo.UsernameValidationResult;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.web.bind.annotation.*;

import javax.annotation.Resource;
import javax.servlet.http.HttpServletRequest;

@RestController
@RequestMapping("/sign")
public class LoginController {
    private static final Logger log = LoggerFactory.getLogger(LoginController.class);

    @Resource
    LoginService loginService;

    // 登录的第一个接口，验证用户名
    @GetMapping("/validUsername/{username}")
    public AjaxResult validUsername(@PathVariable String username) {
        UsernameValidationResult validationResult = loginService.validUsername(username);
        return AjaxResult.ok(validationResult);
    }

    // 登录
    @PostMapping("/login")
    public AjaxResult login(@RequestBody DtoSign body, HttpServletRequest request) {
        log.info("Login message: loginName[{}], password[{}]", body.getLoginName(), body.getPassword());
        LoginSuccess result = loginService.login(body, request);
        return AjaxResult.ok(result);
    }

    // 注册的第一个接口，生成密码盐
    @GetMapping("/randomSalt/{username}")
    public AjaxResult randomSalt(@PathVariable String username) {
        SysUser saltUser = loginService.randomSalt(username);
        return AjaxResult.ok(saltUser);
    }

    // 注册
    @PostMapping("/register")
    public AjaxResult register(@RequestBody DtoSign body, HttpServletRequest request) {
        log.info("Register message: loginName[{}], password[{}]", body.getLoginName(), body.getPassword());
        LoginSuccess result = loginService.register(request, body.getLoginName(), body.getPassword());
        return AjaxResult.ok(result);
    }
}
```

类中使用到了用于接收请求体参数而封装的dto对象，代码结构很简单：

``` java
package cn.gmwenterprise.presevere.dto;
public class DtoSign {
    private String loginName;
    private String password;
    // getter setter
}
```

Todo：**Dto参数校验**

创建其所需的服务接口`LoginService`。

``` java
package cn.gmwenterprise.presevere.service;
import cn.gmwenterprise.presevere.domain.SysUser;
import cn.gmwenterprise.presevere.dto.DtoSign;
import cn.gmwenterprise.presevere.vo.LoginSuccess;
import cn.gmwenterprise.presevere.vo.UsernameValidationResult;
import javax.servlet.http.HttpServletRequest;

public interface LoginService {
    LoginSuccess register(HttpServletRequest request, String username, String password);

    LoginSuccess login(DtoSign body, HttpServletRequest request);

    void logout();

    SysUser randomSalt(String username);

    UsernameValidationResult validUsername(String username);
}
```

同时创建这个接口又创建了`LoginSuccess`与`UsernameValidationResult`这两个用于接口返回数据封装的vo类。

``` java
package cn.gmwenterprise.presevere.vo;

public class UsernameValidationResult {
    private boolean frontEncoded;
    private String salt;
    // 构造、getter setter
}

public class LoginSuccess {
    // 就封装了一个token，也可以不封装，不过还是封装吧
    private String token;
}
```

现在实现`LoginService`接口：

``` java
package cn.gmwenterprise.presevere.service.impl;

import cn.gmwenterprise.presevere.common.BusinessException;
import cn.gmwenterprise.presevere.common.Role;
import cn.gmwenterprise.presevere.common.TokenHelper;
import cn.gmwenterprise.presevere.config.security.Authentication;
import cn.gmwenterprise.presevere.dao.SysUserMapper;
import cn.gmwenterprise.presevere.domain.SysUser;
import cn.gmwenterprise.presevere.dto.DtoSign;
import cn.gmwenterprise.presevere.service.LoginService;
import cn.gmwenterprise.presevere.service.RoleService;
import cn.gmwenterprise.presevere.vo.LoginSuccess;
import cn.gmwenterprise.presevere.vo.UsernameValidationResult;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;
import org.springframework.util.StringUtils;

import javax.annotation.Resource;
import javax.servlet.http.HttpServletRequest;
import java.security.SecureRandom;
import java.time.LocalDateTime;
import java.util.Base64;

@Service
public class LoginServiceImpl implements LoginService {
    @Resource
    SysUserMapper sysUserMapper;
    @Resource
    RoleService roleService;

    private BCryptPasswordEncoder passwordEncoder = new BCryptPasswordEncoder();

    @Transactional(rollbackFor = Exception.class)
    @Override
    public LoginSuccess register(HttpServletRequest request, String username, String password) {
        SysUser user = sysUserMapper.selectByUsername(username);
        user.setPassword(passwordEncoder.encode(password));

        SysUser tobeUpdated = new SysUser();
        tobeUpdated.setId(user.getId());
        tobeUpdated.setPassword(user.getPassword());
        sysUserMapper.updateByPrimaryKeySelective(tobeUpdated);

        // 初始化用户角色信息, 给定一个userId，对其增加一系列指定角色字符串
        roleService.setRoles(user.getId(), Role.USER.getRole());

        // 注册成功
        Authentication payload = new Authentication(
            LocalDateTime.now(), request.getRemoteHost(),
            user.getId(), Authentication.Platform.BROWSER, 0L
        );
        return new LoginSuccess(TokenHelper.generateToken(payload));
    }

    @Override
    public LoginSuccess login(DtoSign body, HttpServletRequest request) {
        String username = body.getLoginName();
        String password = body.getPassword();

        SysUser sysUser = sysUserMapper.selectByUsername(username);
        if (sysUser == null) {
            throw new BusinessException("用户不存在，请重新输入！");
        }
        boolean passwordEquals;
        if (!StringUtils.isEmpty(sysUser.getSalt())) {
            passwordEquals = passwordEncoder.matches(password, sysUser.getPassword());
        } else {
            passwordEquals = password.equals(sysUser.getPassword());
        }
        if (passwordEquals) {
            // 登录成功
            Authentication payload = new Authentication(
                LocalDateTime.now(), request.getRemoteHost(),
                sysUser.getId(), Authentication.Platform.BROWSER, 0L
            );
            return new LoginSuccess(TokenHelper.generateToken(payload));
        }
        throw new BusinessException("密码错误，请重新输入！");
    }

    // 这里简单地使用了系统自带地base64来生成对于每个账户都不同的、差别较大的密码盐。
    // 我并没有专门研究加密相关的知识，所以这些目前来说，，，将就吧
    @Override
    public SysUser randomSalt(String username) {
        if (sysUserMapper.countByUsername(username) > 0) {
            throw new BusinessException("该用户名已经被使用！");
        }
        byte[] bytes = new byte[12];
        new SecureRandom(username.getBytes()).nextBytes(bytes);
        String salt = Base64.getEncoder().encodeToString(bytes);
        SysUser user = new SysUser();
        user.setUsername(username);
        user.setSalt(salt);
        sysUserMapper.insertSelective(user);
        return user;
    }

    // 如果用户不存在，直接抛业务异常；存在，则返回验证结果，如果密码盐有值则加密密码
    // 其实通过系统注册的所有账户都是有密码盐的，这里的这个设置，仅仅是为了我在开发时能有自己设置的管理员账户，且数据库能够查看密码
    // 因为我记性不好，数据库看到的密码都是加密后的，且无法反向解密，要是以后我用自己的网站发文章突然忘了密码，，嗯~ o(*￣▽￣*)o
    // 可以针对管理员账户设置更多的登录限制，收集更多的客户端信息用于验证
    @Override
    public UsernameValidationResult validUsername(String username) {
        SysUser sysUser = sysUserMapper.selectByUsername(username);
        if (sysUser == null) {
            throw new BusinessException("用户不存在，请重新输入！");
        }
        UsernameValidationResult validationResult = new UsernameValidationResult();
        validationResult.setFrontEncoded(!StringUtils.isEmpty(sysUser.getSalt()));
        validationResult.setSalt(sysUser.getSalt());
        return validationResult;
    }
}
```

对于mapper类中的一些更新并没有给出代码，mapper的方法命名都是可以很直观地映射出sql结构地，故无需多言。复杂sql我会给出代码。（应该暂时写不到复杂sql，总共就这么几张表）

未完待续...
