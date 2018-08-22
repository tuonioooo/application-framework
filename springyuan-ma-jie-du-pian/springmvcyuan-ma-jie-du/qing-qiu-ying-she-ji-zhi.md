# 请求映射机制

## HandlerMapping

将请求映射到处理程序以及用于预处理和后处理的拦截器列表。映射基于某些标准，其细节因HandlerMapping实施而异。

## HandlerMapping依赖结构图

![](file:///C:\Users\tony\AppData\Roaming\Tencent\Users\596807862\QQ\WinTemp\RichOle\~BY%%6TX8AFZ26DU8ANJ4QV.png)

RequestMappingHandlerMapping实现支持带@RequestMapping注释的方法

SimpleUrlHandlerMapping并维护URI路径模式到处理程序的显式注册。  




