---
layout: post
title: "Spring Security 3 升级到 4 的过程中遇到的一些问题"
date: 2019-03-25 16:00:00 +0800
categories: SSH
tags: Spring Security
author: Bobby
---

* content
{:toc}

近几日在整理一个以前做过的项目，并顺便升级了一些其中用到的框架。将以前用的 Spring Security 3.1 升级到了 Spring Security 4.2。结果发现以前写的认证逻辑几乎可以重用，但是还是被有些变化给**`坑`**到了，而这些变化不是很明显，有的甚至也不报错。



### XML 文件头改变

因为 Spring Security 3 配置使用的 XML 文件，为了尽量重用以前的代码，升级后也采用 XML 文件配置。首先就是 Spring Security 4 配置文件的 schema （xsd 文件）的变化。其实，整个 Spring 升级到 4 之后统一都做了改变。

Spring Security 3 配置头：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:security="http://www.springframework.org/schema/security"
	xmlns:context="http://www.springframework.org/schema/context"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans 
        http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
        http://www.springframework.org/schema/context 
        http://www.springframework.org/schema/context/spring-context-3.0.xsd  
        http://www.springframework.org/schema/security 
        http://www.springframework.org/schema/security/spring-security-3.1.xsd">

                        ... ...
</beans>
```

Spring Security 4 配置头：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans:beans xmlns="http://www.springframework.org/schema/security"
	xmlns:beans="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:context="http://www.springframework.org/schema/context"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
		http://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/context
		http://www.springframework.org/schema/context/spring-context.xsd
		http://www.springframework.org/schema/security
		http://www.springframework.org/schema/security/spring-security.xsd">

                                ... ...
</beans:bean>
```

其实就是取消了后面具体的版本号后缀。而且 Spring Security 4 也推荐使用 security 作为默认的命名空间（namespace）。

### 默认的登录路径和表单域变化

整理好这些配置后（有些配置的写法发生了变化），启动程序。输入正确的用户名和密码，进行登录，结果始终无法登录。通过 debug 发现在我自己的写的`org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter` 类的继承类中的`attemptAuthentication()` 方法中无法得到前台页面上输入的用户名和密码。通过查看源代码发现，Spring Security 4 中使用的默认的用户名和密码表单域名称变成了`username` 和 `password`，而对应的再 Spring Security 3 中分别默认的是`j_username` 和 `j_password`。另外默认提交的 action 路径也由以前的`/j_spring_security_check`变成了`/login`。这些如果没有显式地配置，确实比较坑。

Spring Security 3 中：

```java
public class UsernamePasswordAuthenticationFilter extends AbstractAuthenticationProcessingFilter {
    //~ Static fields/initializers =====================================================================================

    public static final String SPRING_SECURITY_FORM_USERNAME_KEY = "j_username";
    public static final String SPRING_SECURITY_FORM_PASSWORD_KEY = "j_password";
    /**
     * @deprecated If you want to retain the username, cache it in a customized {@code AuthenticationFailureHandler}
     */
    @Deprecated
    public static final String SPRING_SECURITY_LAST_USERNAME_KEY = "SPRING_SECURITY_LAST_USERNAME";

    private String usernameParameter = SPRING_SECURITY_FORM_USERNAME_KEY;
    private String passwordParameter = SPRING_SECURITY_FORM_PASSWORD_KEY;
    private boolean postOnly = true;

    //~ Constructors ===================================================================================================

    public UsernamePasswordAuthenticationFilter() {
        super("/j_spring_security_check");
    }

... ...
```

Spring Security 4 中：

```java
public class UsernamePasswordAuthenticationFilter extends
		AbstractAuthenticationProcessingFilter {
	// ~ Static fields/initializers
	// =====================================================================================

	public static final String SPRING_SECURITY_FORM_USERNAME_KEY = "username";
	public static final String SPRING_SECURITY_FORM_PASSWORD_KEY = "password";

	private String usernameParameter = SPRING_SECURITY_FORM_USERNAME_KEY;
	private String passwordParameter = SPRING_SECURITY_FORM_PASSWORD_KEY;
	private boolean postOnly = true;

	// ~ Constructors
	// ===================================================================================================

	public UsernamePasswordAuthenticationFilter() {
		super(new AntPathRequestMatcher("/login", "POST"));
	}

... ...
```

### 默认的请求响应安全头变化

Security Header 相关的内容是 Spring Security 4 新增的。其实新增了好几个，而且还可以添加自定义的 header。我这里只是说一下我遇到的问题。

修改了上面的问题，现在可以正常登录，并且获取的了相应权限。但是当我点击有些菜单时（具有权限的菜单），却报出了下面的错误。后台并没有任何错误信息，只在浏览器 Console 控制台发现了：

![spring-security-frame-deny](/assets/images/2019/03/spring-security-frame-deny.jpg)

最终，在网上找了资料，也看了下 Spring Security 的官方文档。原来是我们的程序中用到了 frame/iframe 布局格式，而 Spring Security 对 frame 中的请求默认拒绝。需要将 header `X-Frame-Options` 的默认值`DENY` 改为 `SAMEORIGIN`，即允许同一个域下的请求。

Spring Security 4 中配置：

```xml
<http use-expressions="true" ...  >
        ... ...
    <headers>
        <frame-options policy="SAMEORIGIN"/>
    </headers>
        ... ...
</http>
```

### 遇到的一些其他改变

#### 新增 CSRF （Cross Site Request Forgery）防护

CSRF 防护默认是开启的，在登录的时候需要验证 token。以前用的 Spring Security 3 程序中没有相关功能，所以暂时需要禁用 CSRF：

Spring Security 4 中配置：

```xml
<http use-expressions="true" ... >
        ... ...
    <csrf disabled="true" />
        ... ...
</http>
```

#### LoginUrlAuthenticationEntryPoint 配置变化

在配置 `org.springframework.security.web.authentication.LoginUrlAuthenticationEntryPoint` 时，需要注入 `loginFormUrl` 属性。在 Spring Security 3 中 `LoginUrlAuthenticationEntryPoint` 有无参构造函数，所以我们可以使用 property 注入。但是 Spring Security 4 中，只有一个带 `loginFormUrl` 属性的构造函数，删除了无参构造函数，所以只能用构造函数方式注入。

Spring Security 3 中配置：

```xml
<bean id="authenticationProcessingFilterEntryPoint"
    class="org.springframework.security.web.authentication.LoginUrlAuthenticationEntryPoint">
    <property name="loginFormUrl" value="/login.jsp"></property>
</bean>
```

Spring Security 4 中配置：

```xml
<beans:bean id="authenticationProcessingFilterEntryPoint"
    class="org.springframework.security.web.authentication.LoginUrlAuthenticationEntryPoint">
    <beans:constructor-arg name="loginFormUrl" value="/login.jsp" />
</beans:bean>
```

*注*：一个用的 `<bean/>`，一个用的 `<beans:bean/>`。是因为使用的配置文件的默认命名空间不同，本质上是一样的，上面已经提到过了。

---

### 参考资料：

[通过 Spring Security配置 解决X-Frame-Options deny 造成的页面空白](https://blog.csdn.net/princeluan/article/details/73268637)  
[X-Frame-Options防止网页放在iframe中](https://www.cnblogs.com/5426z/articles/6097324.html)  
