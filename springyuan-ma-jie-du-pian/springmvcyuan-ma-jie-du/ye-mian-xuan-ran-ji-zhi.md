# 页面渲染机制

## 前言

当我们对SpringMVC控制的资源发起请求时，这些请求都会被SpringMVC的DispatcherServlet处理，接着Spring会分析看哪一个HandlerMapping定义的所有请求映射中存在对该请求的最合理的映射。然后通过该HandlerMapping取得其对应的Handler，接着再通过相应的HandlerAdapter处理该Handler。HandlerAdapter在对Handler进行处理之后会返回一个ModelAndView对象。在获得了ModelAndView对象之后，Spring就需要把该View渲染给用户，即返回给浏览器。在这个渲染的过程中，发挥作用的就是ViewResolver和View。当Handler返回的ModelAndView中不包含真正的视图，只返回一个逻辑视图名称的时候，ViewResolver就会把该逻辑视图名称解析为真正的视图View对象。View是真正进行视图渲染，把结果返回给浏览器的。

## ViewResolver和View介绍

**ViewResolver结构图**

![](/assets/view.png)

**主要ViewResolver简介**

| InternalResourceViewResolver | 将逻辑视图名解析为一个路径 |
| :--- | :--- |
| BeanNameViewResolver | 将逻辑视图名解析为bean的name属性，从而根据name属性去找对应的bean |
| ResourceBundleResolver | 和BeanNameViewResolver一样，只不过定义的view-bean都在一个properties文件中，用这个类进行加载这个properties文件 |
| XmlViewResolver | 和ResourceBundleResolver一样，只不过定义的view-bean在一个xml文件中，用这个类来加载xml文件 |
| UrlBasedViewResolver | 它简单实现了ViewResolver接口, 不用任何映射就能通过逻辑视图名称访问资源 |
| VelocityViewResolver | 对Velocity模版引擎的支持 |
| FreeMarkerViewResolver | 对FreeMarker模版引擎的支持 |



SpringMVC用于处理视图最重要的两个接口是ViewResolver和View。ViewResolver的主要作用是把一个逻辑上的视图名称解析为一个真正的视图，SpringMVC中用于把View对象呈现给客户端的是View对象本身，而ViewResolver只是把逻辑视图名称解析为对象的View对象。View接口的主要作用是用于处理视图，然后返回给客户端。

Spring为我们提供了非常多的视图解析器，下面将列举一些视图解析器。

**AbstractCachingViewResolver**：这是一个抽象类，这种视图解析器会把它曾经解析过的视图保存起来，然后每次要解析视图的时候先从缓存里面找，如果找到了对应的视图就直接返回，如果没有就创建一个新的视图对象，然后把它放到一个用于缓存的map中，接着再把新建的视图返回。使用这种视图缓存的方式可以把解析视图的性能问题降到最低。

**UrlBasedViewResolver**：它是对ViewResolver的一种简单实现，而且继承了AbstractCachingViewResolver，主要就是提供的一种拼接URL的方式来解析视图，它可以让我们通过prefix属性指定一个指定的前缀，通过suffix属性指定一个指定的后缀，然后把返回的逻辑视图名称加上指定的前缀和后缀就是指定的视图URL了。如prefix=/WEB-INF/jsps/，suffix=.jsp，返回的视图名称viewName=test/indx，则UrlBasedViewResolver解析出来的视图URL就是/WEB-INF/jsps/test/index.jsp。默认的prefix和suffix都是空串。URLBasedViewResolver支持返回的视图名称中包含redirect:前缀，这样就可以支持URL在客户端的跳转，如当返回的视图名称是”redirect:test.do”的时候，URLBasedViewResolver发现返回的视图名称包含”redirect:”前缀，于是把返回的视图名称前缀”redirect:”去掉，取后面的test.do组成一个RedirectView，RedirectView中将把请求返回的模型属性组合成查询参数的形式组合到redirect的URL后面，然后调用HttpServletResponse对象的sendRedirect方法进行重定向。同样URLBasedViewResolver还支持forword:前缀，对于视图名称中包含forword:前缀的视图名称将会被封装成一个InternalResourceView对象，然后在服务器端利用RequestDispatcher的forword方式跳转到指定的地址。使用UrlBasedViewResolver的时候必须指定属性viewClass，表示解析成哪种视图，一般使用较多的就是InternalResourceView，利用它来展现jsp，但是当我们使用JSTL的时候我们必须使用JstlView。下面是一段UrlBasedViewResolver的定义，根据该定义，当返回的逻辑视图名称是test的时候，UrlBasedViewResolver将把逻辑视图名称加上定义好的前缀和后缀，即“/WEB-INF/test.jsp”，然后新建一个viewClass属性指定的视图类型予以返回，即返回一个url为“/WEB-INF/test.jsp”的InternalResourceView对象。

```
<bean  
   class="org.springframework.web.servlet.view.UrlBasedViewResolver">  
   <property name="prefix" value="/WEB-INF/" />  
   <property name="suffix" value=".jsp" />  
   <property name="viewClass" value="org.springframework.web.servlet.view.InternalResourceView"/>  
</bean>
```

**InternalResourceViewResolver**

：它是URLBasedViewResolver的子类，所以URLBasedViewResolver支持的特性它都支持。在实际应用中InternalResourceViewResolver也是使用的最广泛的一个视图解析器。那么InternalResourceViewResolver有什么自己独有的特性呢？单从字面意思来看，我们可以把InternalResourceViewResolver解释为内部资源视图解析器，这就是InternalResourceViewResolver的一个特性。InternalResourceViewResolver会把返回的视图名称都解析为InternalResourceView对象，InternalResourceView会把Controller处理器方法返回的模型属性都存放到对应的request属性中，然后通过RequestDispatcher在服务器端把请求forword重定向到目标URL。比如在InternalResourceViewResolver中定义了prefix=/WEB-INF/，suffix=.jsp，然后请求的Controller处理器方法返回的视图名称为test，那么这个时候InternalResourceViewResolver就会把test解析为一个InternalResourceView对象，先把返回的模型属性都存放到对应的HttpServletRequest属性中，然后利用RequestDispatcher在服务器端把请求forword到/WEB-INF/test.jsp。这就是InternalResourceViewResolver一个非常重要的特性，我们都知道存放在/WEB-INF/下面的内容是不能直接通过request请求的方式请求到的，为了安全性考虑，我们通常会把jsp文件放在WEB-INF目录下，而InternalResourceView在服务器端跳转的方式可以很好的解决这个问题。下面是一个InternalResourceViewResolver的定义，根据该定义当返回的逻辑视图名称是test的时候，InternalResourceViewResolver会给它加上定义好的前缀和后缀，组成“/WEB-INF/test.jsp”的形式，然后把它当做一个InternalResourceView的url新建一个InternalResourceView对象返回。

```
<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">  
   <property name="prefix" value="/WEB-INF/"/>  
   <property name="suffix" value=".jsp"></property>  
</bean>  
```

**XmlViewResolver**：它继承自AbstractCachingViewResolver抽象类，所以它也是支持视图缓存的。XmlViewResolver需要给定一个xml配置文件，该文件将使用和Spring的bean工厂配置文件一样的DTD定义，所以其实该文件就是用来定义视图的bean对象的。在该文件中定义的每一个视图的bean对象都给定一个名字，然后XmlViewResolver将根据Controller处理器方法返回的逻辑视图名称到XmlViewResolver指定的配置文件中寻找对应名称的视图bean用于处理视图。该配置文件默认是/WEB-INF/views.xml文件，如果不使用默认值的时候可以在XmlViewResolver的location属性中指定它的位置。XmlViewResolver还实现了Ordered接口，因此我们可以通过其order属性来指定在ViewResolver链中它所处的位置，order的值越小优先级越高。以下是使用XmlViewResolver的一个示例：

（1）在SpringMVC的配置文件中加入XmlViewResolver的bean定义。使用location属性指定其配置文件所在的位置，order属性指定当有多个ViewResolver的时候其处理视图的优先级。关于ViewResolver链的问题将在后续内容中讲到。

```
<bean class="org.springframework.web.servlet.view.XmlViewResolver">  
   <property name="location" value="/WEB-INF/views.xml"/>  
   <property name="order" value="1"/>  
</bean>  
```

（2）在XmlViewResolver对应的配置文件中配置好所需要的视图定义。在下面的代码中我们就配置了一个名为internalResource的InternalResourceView，其url属性为“/index.jsp”。

```
<?xml version="1.0" encoding="UTF-8"?>  
<beans xmlns="http://www.springframework.org/schema/beans"  
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
    xsi:schemaLocation="http://www.springframework.org/schema/beans  
     http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">  
    <bean id="internalResource" class="org.springframework.web.servlet.view.InternalResourceView">  
       <property name="url" value="/index.jsp"/>  
    </bean>  
</beans> 
```

（3）定义一个返回的逻辑视图名称为在XmlViewResolver配置文件中定义的视图名称——internalResource。

```
@RequestMapping("/xmlViewResolver")  
public String testXmlViewResolver() {  
   return "internalResource";  
}  
```

（4）这样当我们访问到上面定义好的testXmlViewResolver处理器方法的时候返回的逻辑视图名称为“internalResource”，这时候Spring就会到定义好的views.xml中寻找id或name为“internalResource”的bean对象予以返回，这里Spring找到的是一个url为“/index.jsp”的InternalResourceView对象。

**BeanNameViewResolver**：这个视图解析器跟XmlViewResolver有点类似，也是通过把返回的逻辑视图名称去匹配定义好的视图bean对象。不同点有二，一是BeanNameViewResolver要求视图bean对象都定义在Spring的application context中，而XmlViewResolver是在指定的配置文件中寻找视图bean对象，二是BeanNameViewResolver不会进行视图缓存。看一个例子，在SpringMVC的配置文件中定义了一个BeanNameViewResolver视图解析器和一个id为test的InternalResourceview bean对象。

```
<bean class="org.springframework.web.servlet.view.BeanNameViewResolver">  
   <property name="order" value="1"/>  
</bean>  
  
<bean id="test" class="org.springframework.web.servlet.view.InternalResourceView">  
   <property name="url" value="/index.jsp"/>  
</bean>
```

这样当返回的逻辑视图名称是 test的时候，就会解析为上面定义好id为test的InternalResourceView。

**ResourceBundleViewResolver**：它和XmlViewResolver一样，也是继承自AbstractCachingViewResolver，但是它缓存的不是视图，这个会在后面有说到。和XmlViewResolver一样它也需要有一个配置文件来定义逻辑视图名称和真正的View对象的对应关系，不同的是ResourceBundleViewResolver的配置文件是一个属性文件，而且必须是放在classpath路径下面的，默认情况下这个配置文件是在classpath根目录下的views.properties文件，如果不使用默认值的话，则可以通过属性baseName或baseNames来指定。baseName只是指定一个基名称，Spring会在指定的classpath根目录下寻找以指定的baseName开始的属性文件进行View解析，如指定的baseName是base，那么base.properties、baseabc.properties等等以base开始的属性文件都会被Spring当做ResourceBundleViewResolver解析视图的资源文件。ResourceBundleViewResolver使用的属性配置文件的内容类似于这样：

```
resourceBundle.(class)=org.springframework.web.servlet.view.InternalResourceView  
resourceBundle.url=/index.jsp  
test.(class)=org.springframework.web.servlet.view.InternalResourceView  
test.url=/test.jsp  
```

在这个配置文件中我们定义了两个InternalResourceView对象，一个的名称是resourceBundle，对应URL是/index.jsp，另一个名称是test，对应的URL是/test.jsp。从这个定义来看我们可以知道resourceBundle是对应的视图名称，使用resourceBundle.\(class\)来指定它对应的视图类型，resourceBundle.url指定这个视图的url属性。会思考的读者看到这里可能会有这样一个问题：为什么resourceBundle的class属性要用小括号包起来，而它的url属性就不需要呢？这就需要从ResourceBundleViewResolver进行视图解析的方法来说了。ResourceBundleViewResolver还是通过bean工厂来获得对应视图名称的视图bean对象来解析视图的。那么这些bean从哪里来呢？就是从我们定义的properties属性文件中来。在ResourceBundleViewResolver第一次进行视图解析的时候会先new一个BeanFactory对象，然后把properties文件中定义好的属性按照它自身的规则生成一个个的bean对象注册到该BeanFactory中，之后会把该BeanFactory对象保存起来，所以ResourceBundleViewResolver缓存的是BeanFactory，而不是直接的缓存从BeanFactory中取出的视图bean。然后会从bean工厂中取出名称为逻辑视图名称的视图bean进行返回。接下来就讲讲Spring通过properties文件生成bean的规则。它会把properties文件中定义的属性名称按最后一个点“.”进行分割，把点前面的内容当做是bean名称，点后面的内容当做是bean的属性。这其中有几个特别的属性，Spring把它们用小括号包起来了，这些特殊的属性一般是对应的attribute，但不是bean对象所有的attribute都可以这样用。其中\(class\)是一个，除了\(class\)之外，还有\(scope\)、\(parent\)、\(abstract\)、\(lazy-init\)。而除了这些特殊的属性之外的其他属性，Spring会把它们当做bean对象的一般属性进行处理，就是bean对象对应的property。所以根据上面的属性配置文件将生成如下两个bean对象：

```
<bean id="resourceBundle" class="org.springframework.web.servlet.view.InternalResourceView">  
   <property name="url" value="/index.jsp"/>  
</bean>  
  
<bean id="test" class="org.springframework.web.servlet.view.InternalResourceView">  
   <property name="url" value="/test.jsp"/>  
</bean>  
```

从ResourceBundleViewResolver使用的配置文件我们可以看出，它和XmlViewResolver一样可以解析多种不同类型的View，因为它们的View是通过配置的方式指定的，这也就意味着我们可以指定A视图是InternalResourceView，B视图是JstlView。

来看下面这个一个例子，我在SpringMVC的配置文件中定义了一个ResourceBundleViewResolver对象，指定其baseName为views，然后order为1。

```
<bean class="org.springframework.web.servlet.view.ResourceBundleViewResolver">  
   <property name="basename" value="views"/>  
   <property name="order" value="1"/>  
</bean>  
```

我在classpath的根目录下有两个属性文件，一个是views.properties，一个是views\_abc.properties，它们的内容分别如下：

views.properties：

```
resourceBundle.(class)=org.springframework.web.servlet.view.InternalResourceView  
resourceBundle.url=/index.jsp  
test.(class)=org.springframework.web.servlet.view.InternalResourceView  
test.url=/test.jsp  
```

views\_abc.properties：

```
abc.(class)=org.springframework.web.servlet.view.InternalResourceView  
abc.url=/abc.jsp 
```

定义了如下这样一个Controller，它有三个处理器方法。

```
@Controller  
@RequestMapping("/mytest")  
public class MyController {  
    @RequestMapping("resourceBundle")  
    public String resourceBundle() {  
       return "resourceBundle";  
    }  
  
    @RequestMapping("testResourceBundle")  
    public String testResourceBundle() {  
       return "test";  
    }  
  
    @RequestMapping("abc")  
    public String abc() {  
       return "abc";  
    }  
  
}  
```

那么当我们请求/mytest/resourceBundle.do的时候，ResourceBundleViewResolver会首先尝试着来解析该视图，这里Controller处理器方法返回的逻辑视图名称是resourceBundle，ResourceBundleViewResolver按照上面提到的解析方法进行解析，这个时候它发现它是可以解析的，然后就返回了一个url为/index.jsp的InternalResourceView对象。同样，请求/mytest/testResourceBundle.do返回的逻辑视图test和/mytest/abc.do返回的逻辑视图abc它都可以解析。当我们把basename指定为包的形式，如“com.tiantian.views”，的时候Spring会按照点“.”划分为目录的形式，到classpath相应目录下去寻找basename开始的配置文件，如上面我们指定basename为“com.tiantian.views”，那么spring就会到classpath下的com/tiantian目录下寻找文件名以views开始的properties文件作为解析视图的配置文件。

**FreeMarkerViewResolver、VolocityViewResolver**：这两个视图解析器都是UrlBasedViewResolver的子类。FreeMarkerViewResolver会把Controller处理方法返回的逻辑视图解析为FreeMarkerView，而VolocityViewResolver会把返回的逻辑视图解析为VolocityView。因为这两个视图解析器类似，所以这里我就只挑FreeMarkerViewResolver来做一个简单的讲解。FreeMarkerViewResolver和VilocityViewResolver都继承了UrlBasedViewResolver。

对于FreeMarkerViewResolver而言，它会按照UrlBasedViewResolver拼接URL的方式进行视图路径的解析。但是使用FreeMarkerViewResolver的时候不需要我们指定其viewClass，因为FreeMarkerViewResolver中已经把viewClass定死为FreeMarkerView了。

我们先在SpringMVC的配置文件里面定义一个FreeMarkerViewResolver视图解析器，并定义其解析视图的order顺序为1。

```
<bean class="org.springframework.web.servlet.view.freemarker.FreeMarkerViewResolver">  
   <property name="prefix" value="fm_"/>  
   <property name="suffix" value=".ftl"/>  
   <property name="order" value="1"/>  
</bean>  
```

那么当我们请求的处理器方法返回一个逻辑视图名称viewName的时候，就会被该视图处理器加上前后缀解析为一个url为“fm\_viewName.ftl”的FreeMarkerView对象。对于FreeMarkerView我们需要给定一个FreeMarkerConfig的bean对象来定义FreeMarker的配置信息。FreeMarkerConfig是一个接口，Spring已经为我们提供了一个实现，它就是FreeMarkerConfigurer。我们可以通过在SpringMVC的配置文件里面定义该bean对象来定义FreeMarker的配置信息，该配置信息将会在FreeMarkerView进行渲染的时候使用到。对于FreeMarkerConfigurer而言，我们最简单的配置就是配置一个templateLoaderPath，告诉Spring应该到哪里寻找FreeMarker的模板文件。这个templateLoaderPath也支持使用“classpath:”和“file:”前缀。当FreeMarker的模板文件放在多个不同的路径下面的时候，我们可以使用templateLoaderPaths属性来指定多个路径。在这里我们指定模板文件是放在“/WEB-INF/freemarker/template”下面的。

```
<bean class="org.springframework.web.servlet.view.freemarker.FreeMarkerConfigurer">  
   <property name="templateLoaderPath" value="/WEB-INF/freemarker/template"/>  
</bean>  
```

接下来我们定义如下一个Controller：

```
@Controller  
@RequestMapping("/mytest")  
public class MyController {  
  
    @RequestMapping("freemarker")  
    public ModelAndView freemarker() {  
       ModelAndView mav = new ModelAndView();  
       mav.addObject("hello", "andy");  
       mav.setViewName("freemarker");  
       return mav;  
    }  
  
}  
```

由上面的定义我们可以看到这个Controller的处理器方法freemarker返回的逻辑视图名称是“freemarker”。那么如果我们需要把该freemarker视图交给FreeMarkerViewResolver来解析的话，我们就需要根据上面的定义，在模板路径下定义视图对应的模板，即在“/WEB-INF/freemarker/template”目录下建立fm\_freemarker.ftl模板文件。这里我们定义其内容如下：

```
<html>  
    <head>  
       <title>FreeMarker</title>  
    </head>  
    <body>  
       <b>Hello World</b>  
       <font color="red">Hello World!</font>  
       ${hello}  
    </body>  
</html>  
```

经过上面的定义当我们访问/mytest/freemarker.do的时候就会返回一个逻辑视图名称为“freemarker”的ModelAndView对象，根据定义好的视图解析的顺序，首先进行视图解析的是FreeMarkerViewResolver，这个时候FreeMarkerViewResolver会试着解析该视图，根据它自身的定义，它会先解析到该视图的URL为fm\_freemarker.ftl，然后它会看是否能够实例化该视图对象，即在定义好的模板路径下是否有该模板存在，如果有则返回该模板对应的FreeMarkerView。在这里的话/WEB-INF/freemarker/template目录下是存在模板文件fm\_freemarker.ftl的，所以会返回一个url为fm\_freemarker.ftl的FreeMarkerView对象。接着FreeMarkerView就可以利用该模板文件进行视图的渲染了。所以访问结果应该如下所示：  

## 视图解析器链

       在SpringMVC中可以同时定义多个ViewResolver视图解析器，然后它们会组成一个ViewResolver链。当Controller处理器方法返回一个逻辑视图名称后，ViewResolver链将根据其中ViewResolver的优先级来进行处理。所有的ViewResolver都实现了Ordered接口，在Spring中实现了这个接口的类都是可以排序的。在ViewResolver中是通过order属性来指定顺序的，默认都是最大值。所以我们可以通过指定ViewResolver的order属性来实现ViewResolver的优先级，order属性是Integer类型，order越小，对应的ViewResolver将有越高的解析视图的权利，所以第一个进行解析的将是ViewResolver链中order值最小的那个。当一个ViewResolver在进行视图解析后返回的View对象是null的话就表示该ViewResolver不能解析该视图，这个时候如果还存在其他order值比它大的ViewResolver就会调用剩余的ViewResolver中的order值最小的那个来解析该视图，依此类推。当ViewResolver在进行视图解析后返回的是一个非空的View对象的时候，就表示该ViewResolver能够解析该视图，那么视图解析这一步就完成了，后续的ViewResolver将不会再用来解析该视图。当定义的所有ViewResolver都不能解析该视图的时候，Spring就会抛出一个异常。

       基于Spring支持的这种ViewResolver链模式，我们就可以在SpringMVC应用中同时定义多个ViewResolver，给定不同的order值，这样我们就可以对特定的视图特定处理，以此来支持同一应用中有多种视图类型。注意：像InternalResourceViewResolver这种能解析所有的视图，即永远能返回一个非空View对象的ViewResolver一定要把它放在ViewResolver链的最后面。

```
<bean class="org.springframework.web.servlet.view.XmlViewResolver">  
   <property name="location" value="/WEB-INF/views.xml"/>  
   <property name="order" value="1"/>  
</bean>  
  
<bean  
   class="org.springframework.web.servlet.view.UrlBasedViewResolver">  
   <property name="prefix" value="/WEB-INF/" />  
   <property name="suffix" value=".jsp" />  
   <property name="viewClass" value="org.springframework.web.servlet.view.InternalResourceView"/>  
</bean>  
```





