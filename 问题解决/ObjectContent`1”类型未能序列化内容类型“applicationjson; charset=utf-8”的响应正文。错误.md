## ObjectContent`1”类型未能序列化内容类型“application/json; charset=utf-8”的响应正文。错误

**JQuery 调用 mvcWebAPi时有时候报 ObjectContent`1”类型未能序列化内容类型“application/json; charset=utf-8”的响应正文。**

![image-20220424111103629](https://zfh-tuchuang.oss-cn-shanghai.aliyuncs.com/img/image-20220424111103629.png)

**原因分析**

1. web api自带的return Json把模型间的导航属性也算进去了。（A是B的导航属性，B也是A的导航属性，所以会无限循环，导致Json会生成无数层）
2. 可能查询的数据有外键

**解决办法**

1. 修改泛型的返回值 改成标准的返回对象类（建议）
2. 删掉外键关联【也就是删除导航属性】（不建议）
3. 全局删除循环引用 webapiconfig中修改如下：（高效）

```c#
 public static void Register(HttpConfiguration config)
        {   
            //全局删除循环引用
            config.Formatters.JsonFormatter.SerializerSettings.ReferenceLoopHandling = Newtonsoft.Json.ReferenceLoopHandling.Ignore;
            // Web API 配置和服务
            GlobalConfiguration.Configuration.Formatters.XmlFormatter.SupportedMediaTypes.Clear();
            // Web API 路由
            config.MapHttpAttributeRoutes();

            config.Routes.MapHttpRoute(
                name: "DefaultApi",
                routeTemplate: "api/{controller}/{id}",
                defaults: new { id = RouteParameter.Optional }
            );
        }
```

![image-20220424111646973](https://zfh-tuchuang.oss-cn-shanghai.aliyuncs.com/img/image-20220424111646973.png)