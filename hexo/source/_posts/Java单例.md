---
title: Java单例
date: 2018-03-19 21:54:38
tags: [Java]
categories: [Java，设计模式]
---

 

​	单例模式了解下<!--more-->记录下常用设计模式

* 饿汉式

  ```
  public class Singleton {   
      private static Singleton = new Singleton();
      private Singleton() {}
      public static getSignleton(){
          return singleton;
      }
  }
  ```

  **好处**：

   * 编写简单

  **缺点**：

  * 无法懒加载，饿汉嘛 你要我就给

* 懒汉式：

  ```
   public class SingletonDemo {
    private static SingletonDemo instance;
      private SingletonDemo(){

      }
      public static SingletonDemo getInstance(){
          if(instance==null){
              instance=new SingletonDemo();
          }
          return instance;
      }
  ```

  **优点**：

  * 实现懒加载

  **缺点**:

  * 未考虑线程安全，多线程环境下

* 加锁懒汉式

  ```
  public class Singleton {
      private static volatile Singleton singleton = null;
       
      private Singleton(){}
       
      public static Singleton getSingleton(){
          if(singleton == null){
              synchronized (Singleton.class){
                  if(singleton == null){
                      singleton = new Singleton();
                  }
              }
          }
          return singleton;
      }    
  }
  ```

  **优点**：

   * 线程安全，保证效率，避免在锁前排队

  **缺点**：

  * volatile 存在问题

* 枚举式

  ```
  public Class A{
      
  }
  public enum A_Singleton {
     	INSTANCE;
      private A instance;
      A_Singleton() {
          instance = new A();
      }
      public A getInstance() {
          return instance;
      }
  }
  ```

  **优点**：

  * 自由序列化
  * 保证只有一个实例
  * 线程安全 **推荐这种写法**

* 静态内部类

  ```
  public class Singleton {
      private static class Holder {
          private static Singleton singleton = new Singleton();
      }
       
      private Singleton(){}
           
      public static Singleton getSingleton(){
          return Holder.singleton;
      }
  }
  ```

   **优点**：

  * 静态内部类只加载一次 
  * 静态实例在类加载时候就加载了

  **缺点**：

  * 额外的序列化工作如（Serializable、transient、readResolve()）,否则每次反序列化一个已序列化的对象实例都会创建一个新的实例