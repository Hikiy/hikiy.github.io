---
layout: post
title:  "Spring 学习记录：拦截器"
date:   2019-08-29 11:42:00 +0200
categories: Spring
excerpt: 
tagg: Spring

---

# 拦截器
## 1.实现 `HandlerInterceptor`

这里用自己写过的一个权限拦截做例子：

```
public class LoginInterceptor implements HandlerInterceptor {
    /**
     * 目标方法执行之前执行
     *
     * @param request
     * @param response
     * @param handler
     * @return
     */
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) {
        Object uid = request.getSession().getAttribute("uid");
        //没有登录，返回错误页面
        if( uid == null || (int)uid < 1 ){
            try {
                response.sendRedirect("/auth_error");     //没有uid信息的话进行路由重定向
            } catch (IOException e) {
                e.printStackTrace();
            }
            return false;
        }
        return true;
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) {

    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) {

    }
}
```

## 2.配置拦截器

```
@Configuration
public class WebMvcConfig implements WebMvcConfigurer {
    /**
     * 自定义拦截规则
     *
     * @param registry
     */
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        // addPathPatterns - 用于添加拦截规则
        // excludePathPatterns - 用户排除拦截

        registry.addInterceptor(new LoginInterceptor())
                .addPathPatterns("/admin/**");

        //      .excludePathPatterns("/index.html", "/", "/user/login");
    }
}
```

<br /><br /><br /><br />
> github: https://github.com/Hikiy  
> 作者：Hiki  
> 创建日期：2019.8.29  
> 更新日期：2019.8.29
