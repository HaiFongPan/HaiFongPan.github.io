

---
author: LeoPan
comments: true
date: 2013-05-10 16:18:10+00:00
layout: post
slug: monodevelop开发asp.net-mvc2-json序列化出错的问题
title: MonoDevelop开发ASP.NET MVC2 JSON序列化出错的问题
wordpress_id: 590
categories:
- ASP.NET
- MONO
- Study Note
tags:
- ASP.NET
- MONO
---

### PS:我知道在Linux上写C# ASP.NET是很反人类的事





Mono 版本 2.10.9-1





简单的ASP.NET MVC2的程序
前台框架:Extjs





关键代码





<!-- more -->
`
//UserController简单代码
public ActionResult getUser(){
    User user = new User (a,b,c);
    JavaScriptSerializer serializer = new JavaScriptSerializer();
    String strJson = serializer.Serialize(user);
    return Json(strJson,JsonRequestBehavior.AllowGet);
}
`
{% highlight javascript %}
//Extjs Model中简单代码
Ext.define('leo.model.User',{
    extend: 'Ext.data.Model',
    fields: ['a','b','c'],




    
    <code>proxy: {
        type: 'ajax',
        api:{
            read:'/User/getUser'
        } , 
        reader: {
            type: 'json'
        },  
    }
    </code>





})
{% endhighlight %}





当请求发送至后台,后台返回json数据的时候出现错误(###在vs2010下正常)
`(Cannot cast from source type to destination type)
at System.Web.Script.Serialization.JavaScriptSerializer..ctor (System.Web.Script.Serialization.JavaScriptTypeResolver resolver, Boolean registerConverters) [0x0000d] in /build/mono/src/mono-2.10.9/mcs/class/System.Web.Extensions/System.Web.Script.Serialization/JavaScriptSerializer.cs:66 
at System.Web.Script.Serialization.JavaScriptSerializer..cctor () [0x00000] in <filename unknown>:0 `







  * 分析:
在程序启动后,MonoDevelop应用程序输出中发现System.Web.Extensions 4.0.0.0已经加载了
[![2013-05-11-000530_1091x668_scrot](http://www.leopan.me/wp-content/uploads/2013/05/2013-05-11-000530_1091x668_scrot.png)](http://www.leopan.me/wp-content/uploads/2013/05/2013-05-11-000530_1091x668_scrot.png)





`Loaded assembly: ..../System.Web.Extensions/4.0.0.0__31bf3856ad364e35/System.Web.Extensions.dll`





可是当请求Controller时,程序又加载了一遍System.Web.Extensions,不过这次是3.5.0.0
`Loaded assembly: ..../System.Web.Extensions/3.5.0.0__31bf3856ad364e35/System.Web.Extensions.dll`





于是就知道基本上是冲突了吧.







  * 解决方法
在web.config中配置下绑定System.Web.Extensions不指向3.5.0.0即可





{% highlight xml %}
 
      
         
            
            
         
      
   
{% endhighlight %}





### PPS: 此处web.config 不是Views视图中的web.config 而是整个解决方案中的Web.config.





最后,发现我根本就可以不需要用JavaScriptSerializer,不过解决问题总是好的.



