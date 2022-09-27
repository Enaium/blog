---
title: "Token+Redis登录认证"
date: 2022-03-10T19:17:00+0800
categories: java
---

首先需要了解一下大概的步骤

1. 登录生成一个Token存入Redis有效期为30分钟,返回到前端
2. 之后前端每次请求,带上登录时返回的Token
3. 服务器判断前端带来的Token是否在Redis服务器中
4. 存在放行并且重置Token有效期,不存在拦截

一个简简单单的登录请求

```java
@RequestMapping("/login")
@ResponseBody
public Result<String> login(@RequestBody UserDTO userDTO) {
    var byUsernameAndPassword = mapper.getByUsernameAndPassword(userDTO.getUsername(), userDTO.getPassword());
    if (byUsernameAndPassword != null) {
        return new Result<>(true, "login success");
    }
    return new Result<>(false, "wrong username or password");
}
```

生成一个UUID存入Redis,值为用户的ID,并且设置有效期为30分钟

```java
var uuid = "user-token:" + UUID.randomUUID();
            redisTemplate.opsForValue().set(uuid, byUsernameAndPassword.getId().toString(), 30, TimeUnit.MINUTES);
            return new Result<>(true, uuid);
```

接下来直写拦截器,重写addInterceptors方法

```java
@Configuration
public class RequestInterceptor implements WebMvcConfigurer {
    @Override
    public void addInterceptors(InterceptorRegistry registry) {

    }
}
```

使用`HandlerInterceptor`重写preHandle方法,登录和注册不用拦截

```java
registry.addInterceptor(new HandlerInterceptor() {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        return false;
    }
}).excludePathPatterns("/user/login", "/user/register");
```

前端在请求头中放入Token,之后从请求头中获取Token,从Redis中获取token是否存在,存在返回为true并且重新设置有效期,不存在就返回false设置响应状态为401

```java
@Override
public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
    var token = request.getHeader("token");
    if (token != null && redisTemplate.opsForValue().get(token) != null) {
        redisTemplate.expire(token, 30, TimeUnit.MINUTES);
        return true;
    }
    response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
    return false;
}
```

到这里一个简简单单的Token登录认证就完成了,不过还有个小问题,那就是只有访问需要拦截的地址时,有效期才会被重置,用户一直访问不需要拦截的地址,Token有效期那就不会被重置,所以解决方法也很简单,那就是在登录认证的拦截器之前再加一个拦截器,用来刷新Token有效期
