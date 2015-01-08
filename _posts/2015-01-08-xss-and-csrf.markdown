---
author: LeoPan
comments: true
date: 2015-01-08 19:54:10+00:00
layout: post
slug: '简述SQL注入、XSS、CSRF与防范'
title: 简述SQL注入、XSS、CSRF与防范
wordpress_id: 766
categories:
- xss
- sql注入
- csrf
---

## SQL 注入


> sql 注入攻击，是发生于应用程序数据库层的安全漏洞，攻击者在输入的字符串中添加 SQL 指令，程序没有对输入进行检查而导致数据库认为是误认为是正常的 SQL 指令，从而被攻击者非法增删改查我们的数据

<!-- more -->

### 避免方式

1. 在设计应用程序时，完全使用参数化查询（Parameterized Query）来设计数据访问功能。
2. 在组合SQL字符串时，先针对所传入的参数作字符取代（将单引号字符取代为连续2个单引号字符）。
3. 其他，使用其他更安全的方式连接SQL数据库。
4. 使用SQL防注入系统。
5. PreparedStatement

### 针对 MyBatis 的避免方式

考虑到项目中使用的是 Mybatis，可以这样考虑，SQL 注入只对语句的编译起破坏性作用，而 MyBatis 底层实现中采用了 PreparedStatement（预编译）起作用，如下所示 xml 中配置的语句

{% highlight sql%}
    select * from user where name = #{name} // $则不同
{% endhighlight %}

在参数输入的之后，数据库会将上面语句编译成

{% highlight sql%}
    select * from user where name = ?
{% endhighlight %}

也就是说无论输入参数是什么，在 sql 真正执行前，数据库会先对语句进行编译，执行时直接将参数替换掉占位符，由于不会再对语句进行编译，所以sql注入无法造成破坏，这样能避免掉大部分的攻击。

如果用`${name}`替换了`#{name}`结果就不一样了，采用 $ 符号会将参数一起编译，这样则无法防止 sql 注入，涉及到动态的表名，列名是必须用 $ 符号，所以此时可能需要手动干预。如果该参数是由接口传入或者用户输入，那么需要对输入进行检查（可以通过正则表达式匹配掉参数中的and or delete insert 等等）

\#{...} 比 ${...} 更加安全，但是有时你更想直接在 sql 中插入一个不变的值，如 order by 的时候，那么可以用 $，但是如上所述，${...} 是不安全的用法，能用 #{...} 的地方尽量不要 ${...}，否则要么这部分参数不提供给用户输入的机会，要么就是手动过滤参数。

***Tips***:

    * 确定 MySQL 驱动是否支持默认开启预编译
    * 检查是否有 sql 语句采用了字符串拼接

## XSS 攻击

> `xss` 攻击定义，跨站脚本攻击，是一种网站应用程序的安全漏洞攻击，是代码注入的一种，它允许恶意用户将代码注入到网页上，其他用户在浏览网页时将执行恶意代码，导致隐私泄露等。这类攻击通常包含了`HTML`以及用户端的脚本语言

攻击手段：攻击者使被注入恶意代码的被攻击者在浏览器中执行脚本。
攻击目的：

* 窃取用户 `Cookie`
* 植入 `Java`、`Flash` 等代码获取更高的管理权限
* 利用可被攻击的域收到其他域信任的特点，以受信任来源的身份请求一些平时不允许的操作
* 访问量大的一些页面上 `XSS` 可以产生 `DDoS` 攻击效果
* 利用 `iframe`、`frame`、`XMLHttpRequest` 或者 `Flash` 等方式，可以以用户的身份执行一些管理操作
* 让程序的 CSRF 保护失效

##### Stored XSS

存储型 XSS 攻击是攻击者将攻击脚本存入在服务端（如：blog、论坛帖子等内容），当浏览者在浏览内容的时候会自动执行脚本，如弹出一个伪造的登录，窃取用户信息。如下面代码：

{% highlight js linenos%}
<script>
    document.body.innerHTML="
    <h1>用户登录</h1>
    <form action=http://xxxx.xxx method=post>
        用户名:<input type=text name=user>
        密码:<input type=text name=password>
        <input type=submit name=login>
    </form>"
</script>
{% endhighlight %}
用户在浏览的时候会执行这段注入的JS，攻击者可以窃取用户在站点的登录信息

##### Reflected XSS

反射或者非持久型 XSS 是当不可信的用户输入被服务器在没有任何验证下处理并没有编码或者转移的情况下反射回响应文中，导致代码在浏览器执行的一种 XSS 漏洞

##### DOM BASED XSS

DOM Based XSS 是一种基于网页 DOM 结构的攻击，是客户端 XSS 的一种形式，数据来源于 DOM 中，接收器也在 DOM 中。它发生在一个不可信的数据在源中被给予并被执行，结果导致修改了 DOM 在浏览器中的「环境」。DOM XSS 发生在不可信数据相对于上下文没有被编码或者转义的情况下。

##### 突变 XSS

突变 XSS 是当不可信数据在 DOM 的 InnerHTML 属性的上下文被处理并通过浏览器发生突变，导致变成一种有效的 XSS 向量的一种 XSS 漏洞。

##### 针对策略

* 重要的 Cookie 将标记为 `http-only`，这样用户无法通过 JS 获取 Cookie
* 页面上的一些输入只允许指定的类型，如电话的输入框只允许数字等
* 对数据进行 HTML Encode（前台后台都需要encode）[owasp Java Encoder Project](https://www.owasp.org/index.php/OWASP_Java_Encoder_Project)
* html 标签的一些过滤，禁止用户输入一些不安全的 html 标签
* 如果标签属性是用户输入值，该值必须进行 encode
* 实施「内容安全策略」（CSP）
* 如果 http 头值是用户提供，则禁止用户输入 `\r` 以及 `\n`(CRLF)，否则容易绕过 CSP
* X-XSS-Protection: 1; mode=block（Spring Security 默认开启）

不安全 HTML 标签

    <applet><body><button><embed>
    <form><frame><frameset><html>
    <iframe><image><ilayer><input>
    <layer><link><math><meta>
    <object><script><style><video>

注：

1. 针对 html 所做的 escape 只能保证注入的 html 可以安全，如果输出是 js 则无效了，如果输出的是文本我则不需要做任何的 escape，针对不明确的输出可以进行多重 escape，缺点是 escape 的顺序需要注意，参考知乎问题：[DOM-based XSS 与 存储性XSS、反射型XSS有什么区别？](http://www.zhihu.com/question/26628342/answer/33572866)
2. ***原则***：不信任任何输入数据
3. ***XSS 规避表*** [XSS Filter Evasion Cheat Sheet](https://www.owasp.org/index.php/XSS_Filter_Evasion_Cheat_Sheet)
4. [给开发者的终极XSS防护备忘录 V1.0](http://blog.knownsec.com/2014/07/%E7%BB%99%E5%BC%80%E5%8F%91%E8%80%85%E7%9A%84%E7%BB%88%E6%9E%81xss%E9%98%B2%E6%8A%A4%E5%A4%87%E5%BF%98%E5%BD%95-v1-0/)
5. BurpSuite 等攻击扫描漏洞


## CSRF 攻击

> 跨站请求伪造(Cross-site request forgery)，也被称为 on-click attck 或者 session riding。

攻击者通过一些技术手段欺骗用户的浏览器去访问一个自己曾经认证过的网站并执行一些操作。因为此时用户的浏览器记录了目标站点的 cookie，假如攻击者在了解到目标网站的一些涉及敏感的接口时，利用用户浏览器有 cookie 这个特性可以触发一些敏感的操作

##### 针对策略

1. 认准 Referer 字段，Referer 规则使用正则匹配
2. 添加 token 【session级别的token会在请求被攻击者拦截后攻击，此时可以考虑请求级别的token】
3. 正确使用 GET / POST 和 Cookie
4. HTTP 头中自定义属性并检验
6. 规避 CSRF 的前提一定要先规避 XSS
7. 重要操作需要重新输入密码或是验证码（可能会影响用户体验，但是涉及的金融的项目可以理解）



### Spring Security(3.2.5 RELEASE) 规避 CSRF

1. 使用合适的 HTTP 方法
2. 配置开启 Spring CSRF
3. 使用 Token
    * form 表单提交，hidden 字段加入 _cxrf 内容
    * Ajax / JSON 提交，可放在 meta 头中。JQuery or [rest.js](https://github.com/cujojs/rest) 都可以发送请求
    * Timeout 方案，自定义 AccessorDeniedHander 处理 InvalidCsrfTokenException
    * 登录/登出处理
    * Multipart (file/upload) 处理
    * 可重写默认的 Token 形式

以上参考：[Spring Doc](http://docs.spring.io/spring-security/site/docs/3.2.5.RELEASE/reference/htmlsingle/#csrf)

也可以不采用 Spring Security 这套默认方案，根据 Eyal Lupu [方案](http://blog.eyallupu.com/2012/04/csrf-defense-in-spring-mvc-31.html) 造一个自己的轮子


使用 Spring Security 配置：

方法一

1. 注册过滤器

{% highlight xml linenos%}
<!-- web.xml -->
<filter>
    <filter-name>springSecurityFilterChain</filter-name>
    <filter-class>org.springframework.web
            .filter.DelegatingFilterProxy</filter-class>
</filter>
<filter-mapping>
    <filter-name>springSecurityFilterChain</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
{% endhighlight %}

2. Spring 配置文件中增加过滤器

{% highlight xml linenos%}
<bean id="csrfFilter" class="org.springframework
                            .security.web.csrf.CsrfFilter">
    <constructor-arg>
        <bean class="org.springframework.security.web
                        .csrf.HttpSessionCsrfTokenRepository"/>
    </constructor-arg>
</bean>

<!--
    如果用的是spring mvc 的form标签，则配置此项时自动将crsf的token
    放入到一个hidden的input中，而不需要开发人员显式的写入form
-->
<bean id="crdvp" class="org.springframework.security
        .web.servlet.support.csrf.CsrfRequestDataValueProcessor"/>
{% endhighlight %}


方法二

1. 配置 http 标签

{% highlight xml linenos%}
<!-- *-security.xml -->
<http>
    <!-- ..Spring Security 4.x 的 xml namespace 中默认开启
            使用 JavaConfig 也是默认开启 -->
    <csrf />
</http>
{% endhighlight %}


Ajax 请求加入 _csrf 的 token

{% highlight js linenos%}
// html 可以将 <sec:csrfMetaTags> 置于 header 中
// JQuery 获取代码
var token = $("meta[name='_csrf']").attr("content");
var header = $("meta[name='_csrf_header']").attr("content");
{% endhighlight %}


form 提交加入 token

{% highlight html%}
<input type="hidden" name="${_csrf.parameterName}" value="${_csrf.token}"/>
{% endhighlight %}
Spring 的 <form:form> 配合 CsrfRequestDataValueProcessor 可以自动添加这个 hidden 属性

自己实现 RequestDataValueProcessor 的 getExtraHiddenFields 也能实现自动在 form 中添加

所有内容均来自网络
<完>

