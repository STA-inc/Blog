​
今天将以前的一个项目移植到新机器上开发，在原本的机器上跑得好好的项目移植过来之后报了个莫名其妙的错：  

在xml中给我说xsd的版本不对，于是将原本的如下代码片：  
```xml
xsi:schemaLocation="http://www.springframework.org/schema/beans  
                        http://www.springframework.org/schema/beans/spring-beans-3.1.xsd  
                        http://www.springframework.org/schema/context  
                        http://www.springframework.org/schema/context/spring-context-3.1.xsd  
                        http://www.springframework.org/schema/mvc
                        http://www.springframework.org/schema/mvc/spring-mvc-4.0.xsd">


之中的版本号统统去掉，使用本地jar中自带的xsd版本，遂成功。  
```
更改之后的代码如下：   
```xml
xsi:schemaLocation="http://www.springframework.org/schema/beans  
                        http://www.springframework.org/schema/beans/spring-beans.xsd  
                        http://www.springframework.org/schema/context  
                        http://www.springframework.org/schema/context/spring-context.xsd  
                        http://www.springframework.org/schema/mvc
                        http://www.springframework.org/schema/mvc/spring-mvc.xsd">
```
如此，默认不使用网络上下载的xsd文件，理论上较为优雅，且可以防止断网时应用无法启动、开源软件更换域名时无法启动、项目转移时出现乱七八糟的问题等情况。  
 



​
