---
layout: post_layout
title: Spring MVC学习遇到的问题：JSP未渲染，直接显示源码
time: 2017年5月31日
location: 北京
pulished: true
excerpt_separator: "##"
---

## 一.背景
今天在学习Spring MVC的过程中遇到了直接显示JSP源码的问题，如图：
![图片](http://7xlv11.com1.z0.glb.clouddn.com/Spring%20MVC%20JSP.png)
源码和配置文件如下：
HomeController.java片段:
```java

@Controller
@RequestMapping("/")
public class HomeController {
 
    @RequestMapping(method = RequestMethod.GET)
    public String home() {
        return "home";
    }
}
```
web.xml片段:
```
<servlet>
  <servlet-name>dispatch</servlet-name>
  <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
  <init-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>classpath:springmvc-servlet.xml</param-value>
  </init-param>
</servlet>
<servlet-mapping>
  <servlet-name>dispatch</servlet-name>
  <url-pattern>/*</url-pattern>
</servlet-mapping>
```
## 二.问题解决
### 1.首先说下解决方式:
将web.xml中的 <url-pattern>/*</url-pattern> 改为 <url-pattern>/</url-pattern>

### 2.原因
url映射规则：
<url-pattern>/</url-pattern> 不会匹配到模式为*.jsp这样的后缀型url 。
<url-pattern>/*</url-pattern> 会匹配所有url,包括*.jsp。

当Controller处理完后，View会调用RequestDispatcher的forward方法，此时url是/WEB-INF/home.jsp，而url-pattern为/*时，由DispatcherServlet处理，但是却找不到与之映射的Controller，
至于为什么会显示源码，是因为在Spring MVC的上下文配置文件springmvc-servlet.xml中，添加了`<mvc:default-servlet-handler />`来处理对静态资源的访问，导致将jsp文件当做普通文本处理。如果将此行代码去除，则找不到资源，导致页面404。

### 3.其他情况：

那是不是任何情况都不能用/*呢？不一定。
如果有应用上下文，将url-pattern改为/context{上下文}/*，这样ViewResolver产生的路径(/WEB-INF/...)不会被DispatcherServlet处理，可以正常渲染。

## 三.感想
对Java Web的原理还不够熟悉，需要再深入看下规范。









