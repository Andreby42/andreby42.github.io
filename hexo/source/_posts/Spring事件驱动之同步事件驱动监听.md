---
title: Spring事件驱动之同步事件驱动监听
date: 2017-11-19 01:12:23
tags: [Spring,Java]
categories: [Java]
---



Spring事件驱动之同步事件驱动监听<!--more-->



> 简单记录下Spring的事件驱动

* 首先定义个event事件

  ```
  package com.commons.event;
  import org.springframework.context.ApplicationEvent;
  public class MyEvent extends ApplicationEvent {
      private static final long serialVersionUID = 1L;
      private Object obj;
      
      public Object getObj() {
          return obj;
      }
      public void setObj(Object obj) {
          this.obj = obj;
      }
      public MyEvent(Object source) {
          super(source);
      }
      public MyEvent(Object source, Object obj) {
          super(source);
          this.obj = obj;
      }   
  } 
  ```

  我这里就用Object 来代替你放置在事件中的数据了,注意需要进行**生成构造参数**.


* 然后定义个发布者

  ```java
  package com.commons.event;
  import org.springframework.beans.BeansException;
  import org.springframework.context.ApplicationContext;
  import org.springframework.context.ApplicationContextAware;
  import org.springframework.context.ApplicationEvent;
  import org.springframework.stereotype.Service;
  @Service("myPublisher")
  public class MyPublisher implements ApplicationContextAware {
      private static ApplicationContext applicationContext;
      @Override
      public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
          this.applicationContext = applicationContext;
      }
      public static void publish(ApplicationEvent event) {
          applicationContext.publishEvent(event);
      }
  }     
  ```

  我这里将这个发布者注册到spring,好像因为之前spring上下文拿不到导致发布事件不成功,具体以后再说,建议spring管理

* 定义监听器

  ```java
  package com.commons.event;
  import org.slf4j.Logger;
  import org.slf4j.LoggerFactory;
  import org.springframework.context.ApplicationListener;
  import org.springframework.stereotype.Component;
  @Component("myListener")
  public class MyListener implements ApplicationListener<MyEvent> {
      private static final Logger logger = LoggerFactory.getLogger(MyListener.class);
      /**
       * 在这里可以注入你需要的Bean 然后做逻辑操作
       */
      @Override
      public void onApplicationEvent(MyEvent event) {
          Object obj = event.getObj();
          logger.info("监听器监听到事件{}发布",event.getObj());
      }
  }
  ```

* 调用实例(我这里偷个懒就用postConstruct进行测试了因为在spring项目中):

  ```
  package com.commons.event;
  import javax.annotation.PostConstruct;
  import javax.annotation.Resource;
  import org.springframework.stereotype.Component;
  @Component
  public class TestSpringEvent {
      @Resource
      private MyPublisher publisher;
      @PostConstruct
      private void testPublisher() {
          publisher.publish(new MyEvent(MyListener.class,new String("test")));
      }
  }    
  ```

  ​

下一篇的事件驱动记录关于异步事件驱动以及事务相关