---
title: SpringBoot实现基于Token的web后台认证机制
date: 2019-03-10 15:38:48
tags: [Spring Boot,网络]
---
# 几种常用的认证机制
HTTP Basic Auth、OAuth、Cookie Auth、Token Auth

<!--more-->

## HTTP Basic Auth
HTTP Basic Auth简单点说明就是每次请求API时都提供用户的username和password，简言之，Basic Auth是配合RESTful API 使用的最简单的认证方式，只需提供用户名密码即可，但由于有把用户名密码暴露给第三方客户端的风险，在生产环境下被使用的越来越少。因此，在开发对外开放的RESTful API时，尽量避免采用HTTP Basic Auth
## OAuth2.0
OAuth（开放授权）是一个开放的授权标准，允许用户让第三方应用访问该用户在某一web服务上存储的私密的资源（如照片，视频，联系人列表），而无需将用户名和密码提供给第三方应用。
![image](http://490.github.io/images/20190310_160834.png)

## Cookie Auth
Cookie认证机制就是为一次请求认证在服务端创建一个Session对象，同时在客户端的浏览器端创建了一个Cookie对象；通过客户端带上来Cookie对象来与服务器端的session对象匹配来实现状态管理的。默认的，当我们关闭浏览器的时候，cookie会被删除。但可以通过修改cookie 的expire time使cookie在一定时间内有效；
## Token Auth
![image](http://490.github.io/images/20190310_154210.png)
Token机制相对于Cookie机制又有什么好处呢？
*   **支持跨域访问**: Cookie是不允许垮域访问的，这一点对Token机制是不存在的，前提是传输的用户认证信息通过HTTP头传输.
*   **无状态(也称：服务端可扩展行)**:Token机制在服务端不需要存储session信息，因为Token 自身包含了所有登录用户的信息，只需要在客户端的cookie或本地介质存储状态信息.
*   **更适用CDN**: 可以通过内容分发网络请求你服务端的所有资料（如：javascript，HTML,图片等），而你的服务端只要提供API即可.
*   **去耦**: 不需要绑定到一个特定的身份验证方案。Token可以在任何地方生成，只要在你的API被调用的时候，你可以进行Token生成调用即可.
*   **更适用于移动应用**: 当你的客户端是一个原生平台（iOS, Android，Windows 8等）时，Cookie是不被支持的（你需要通过Cookie容器进行处理），这时采用Token认证机制就会简单得多。
*   **CSRF**:因为不再依赖于Cookie，所以你就不需要考虑对CSRF（跨站请求伪造）的防范。
*   **性能**: 一次网络往返时间（通过数据库查询session信息）总比做一次HMACSHA256计算 的Token验证和解析要费时得多.
*   **不需要为登录页面做特殊处理**: 如果你使用Protractor 做功能测试的时候，不再需要为登录页面做特殊处理.
*   **基于标准化**:你的API可以采用标准化的 JSON Web Token (JWT). 这个标准已经存在多个后端库（.NET, Ruby, Java,Python, PHP）和多家公司的支持（如：Firebase,Google, Microsoft）.

**基于Token的身份验证流程如下**
* 客户端使用用户名跟密码请求登录
* 服务端收到请求，去验证用户名与密码
* 验证成功后，服务端会签发一个 Token，再把这个 Token 发送给客户端
* 客户端收到 Token 以后可以把它存储起来，比如放在 Cookie 里或者 Local Storage 里
* 客户端每次向服务端请求资源的时候需要带着服务端签发的 Token
* 服务端收到请求，然后去验证客户端请求里面带着的 Token，如果验证成功，就向客户端返回请求的数据

# Spring Boot 实现
[实现见UserService，LoginController，PassportInterceptor](https://github.com/490/wenda/tree/master/src/main/java/com/zhaole)
```java
public class LoginTicket
{
    private int id;
    private int userId;
    private Date expired;
    private int status;//0有效
    private String ticket;
}
```
用户先去请求注册或者是登陆，然后服务器去验证他的用户名和密码

验证成功后userservice会生成一个Token，这里是ticket，客户端收到ticket之后呢会把ticket存在Cookie中，登录成功之后会有一个与当前用户对应的ticket

每次访问服务器资源的时候需要带着这个ticket，然后怎么判断是否有呢？就要用拦截器来实现过滤，用拦截器去判断这个ticket当前的状态是什么样的？有没有过期？身份状态是不是有效的？然后根据这个来判断应该赋予什么样的权限？当验证成功之后就把ticket对应的用户的通过下面一段发送给freemaker的上下文，实现页面的正常的渲染

```java
package com.zhaole.interceptor;
import com.zhaole.dao.LoginTicketDAO;
import com.zhaole.dao.UserDAO;
import com.zhaole.model.HostHolder;
import com.zhaole.model.LoginTicket;
import com.zhaole.model.User;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;
import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;
import javax.servlet.http.Cookie;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.util.Date;
/**
 * 拦截器
 * @ 用来判断用户的
 *1. 当preHandle方法返回false时，从当前拦截器往回执行所有拦截器的afterCompletion方法，再退出拦截器链。也就是说，请求不继续往下传了，直接沿着来的链往回跑。
 2.当preHandle方法全为true时，执行下一个拦截器,直到所有拦截器执行完。再运行被拦截的Controller。然后进入拦截器链，运
 行所有拦截器的postHandle方法,完后从最后一个拦截器往回执行所有拦截器的afterCompletion方法.
 */
@Component
public class PasswordInterceptor implements HandlerInterceptor
{
    @Autowired
    private LoginTicketDAO loginTicketDAO;
    @Autowired
    private UserDAO userDAO;
    @Autowired
    private HostHolder hostHolder;
    @Override
    public boolean preHandle(HttpServletRequest httpServletRequest,
                            HttpServletResponse httpServletResponse,
                             Object o) throws Exception
    {
        String ticket = null;
        if(httpServletRequest.getCookies()!=null)
        {
            for(Cookie cookie:httpServletRequest.getCookies())
            {
                if(cookie.getName().equals("ticket"))
                {
                    ticket = cookie.getValue();
                    break;
                }
            }
        }
        if(ticket!=null)//说明cookie不空，且name是ticket
        {
            //去数据库里把ticket找出来看看是否有效
            LoginTicket loginTicket = loginTicketDAO.selectByTicket(ticket);
            if(loginTicket==null || loginTicket.getExpired().before(new Date()) || loginTicket.getStatus()!=0)
            {
                return true;
            }
            //有效的话，查这个ticket对应的用户，把这个用户设置到threadlocal里。
            User user = userDAO.selectById(loginTicket.getUserId());
            hostHolder.setUser(user);
        }
        return true;
    }
    @Override
    public void postHandle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, ModelAndView modelAndView) throws Exception {
        if (modelAndView != null && hostHolder.getUser() != null) {
            modelAndView.addObject("user", hostHolder.getUser());
        }
    }
    @Override
    public void afterCompletion(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, Exception e) throws Exception {
        hostHolder.clear();
    }
}
```
当用户登出的时候就把ticket的身份状态置位为无效状态即可

