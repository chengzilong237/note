# Spring手写经验总

1、IOC容器部分

​	原版Spring中IOC容器和springMvc容器是单独初始化的

​	IOC容器初始化入口为web.xml文件中配置的

```xml
<listener>
<description>spring监听器</description>
<listenerclass>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>
```

对于IOC容器的初始化，spring中的`ContextLoaderListener`实现了servlet的`ServletContextListener`，重写了`ServletContextListener`中的`contextInitialized()`方法，web容器初始化的时会调用`contextInitialized（）`方法进行IOC容器的初始化，涉及到的部分初始化的类关系图如下

![](E:\note\spring\ioc类图.bmp)



对于MVC容器的初始化，spring中的`DispatcherServlet`继承自`HttpServlet`类，在其父类`HttpServletBean`中重写了`HttpServlet`中的`init()`方法，在`init()`方法中调用了`FrameWorkServlet`中的`initServletBean()`方法在此方法中完成`WebApplicationContext`的初始化，WebApplicationContext的初始化过程包含以下内容，

![](E:\note\spring\mvc容器.bmp)