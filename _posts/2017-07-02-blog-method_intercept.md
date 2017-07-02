---
layout: post
title: "记一次方法拦截解决方案"
description: ""
category: 技术分享
tags: [blog]
---
{% include JB/setup %}
# 记一次方法拦截解决方案
---

　
### 记一次方法拦截解决方案
#### 问题
> 项目依赖quartz定时任务，但是有些时候会出现同一时间一个任务执行多次的情况，具体原因可能是由于quartz任务调度有问题。
但是这种情况只是偶尔出现，问题不容易复现。当我的任务同一时间多个线程一起执行就会出现数据错乱，所以当务之急是防止我的业务代码并发执行。

<!-- break -->

#### 大致解决方案
> 拦截执行的业务方法。比如我要执行的业务代码是 BizService.exec()，使用Spring AOP技术拦截该方法的执行，@Around，在该方法执行时将方法执行记录到数据库，
每次执行前都判断一下该方法时候正在执行，如果正在执行则直接跳过，否则 记录执行记录到数据库->执行->更新执行记录。

#### 遇到的问题
> 我的BizService不是一个Spring管理的实体。所以直接对方法进行Spring AOP拦截不起作用，解决方案： 将该方法调用方式修从new对象改成通过BizServiceManager.getInstance()。
在.getInstance()方法里面首先在SpringContext力注册一个该BizService实例的实例。然后再将注册的这个BizService返回，这样返回的实例就会是Spring容器管理的，方法也会被Spring Aop拦截。
以下是.getInstance()方法的具体代码：

``` java

  SpringBeanUtils.registerBeanDefinition("bizService", BizService.class, new Object[]{}, new HashMap<String,Object>());

  return MyApplicationContextUtil.getContext().getBean(BizService.class);

```

SpringBeanUtils.registerBeanDefinition

```java
// 等同 new 操作，但对象是被spring容器托管的
public static synchronized void registerBeanDefinition(String beanName, Class<?> clazz, Object[] argValues, Map<String, Object> propertyValues) {
  ConfigurableApplicationContext configurableApplicationContext = (ConfigurableApplicationContext) MyApplicationContextUtil.getContext();
  DefaultListableBeanFactory defaultListableBeanFactory = (DefaultListableBeanFactory) configurableApplicationContext.getBeanFactory();
  BeanDefinitionBuilder beanDefinitionBuilder = BeanDefinitionBuilder.genericBeanDefinition(clazz);
  beanDefinitionBuilder.setAutowireMode(AutowireCapableBeanFactory.AUTOWIRE_BY_TYPE);

  for (Map.Entry<String, Object> entry : propertyValues.entrySet()) {
      beanDefinitionBuilder.addPropertyValue(entry.getKey(), entry.getValue());
  }

  for (Object argValue : argValues) {
      beanDefinitionBuilder.addConstructorArgValue(argValue);
  }

  defaultListableBeanFactory.registerBeanDefinition(beanName, beanDefinitionBuilder.getBeanDefinition());

}
```

MyApplicationContextUtil

``` java

// 注意： 需要将该类配置到servlet-context.xml中。
public class MyApplicationContextUtil implements ApplicationContextAware {
        private static ApplicationContext instance;
        public void setApplicationContext(ApplicationContext contex) throws BeansException {
                instance=contex;
        }
        public static ApplicationContext getContext(){
                return instance;
        }
}

```

部分web.xml

``` xml
<servlet>
                <servlet-name>appServlet</servlet-name>
                <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
                <init-param>
                        <param-name>contextConfigLocation</param-name>
                        <param-value>/WEB-INF/spring/appServlet/servlet-context.xml</param-value>
                </init-param>
                <load-on-startup>1</load-on-startup>
</servlet>
```
