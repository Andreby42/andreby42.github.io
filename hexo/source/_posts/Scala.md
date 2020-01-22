---
title: Scala
date: 2020-01-22 18:09:08
tags: scala
categories: scala

---

以前看scala的一些笔记<!---more--->





* scala命名规范

  * 有意义简明

  * 驼峰式

* 运算符

  * 算数运算符

    * 除以

      ```scala
      var i=10/3
      //此处i的值为3.0 因为默认数值类型为int 所以除法获得的结果只能保留整数
      ```

      

    * 取模=a-a/b*b

      ```scala
       var j=10%3
       
       j=-10%3(-10-(-3)*3=-10+9)
       
      ```

      

      ```scala
         ...
         //还有97天放假 还有多少个星期
          var a = 97 / 7;
          val b = 97 % 7;
          println("当前剩余" + a + "个星期零" + b + "天");
          if (b > 0) {
            a = a + 1
            println(a)
          } else {
            println(a)
          }
          //定义变量保存华氏温度 5/9*（华氏温度-100） 请求华氏温度对应的摄氏温度 测试温度为232.5
          val sheshi= huaShi(232.5);
          println(sheshi)
        }
      
        def huaShi(huashi: Double): String = {
          (5.0 / 9 * (huashi - 100.00)).formatted("%.2f")
        }
      ```

      

  * 赋值运算符

    scala中没有++和-- 用+= -= 代替

  * 比较运算符 （关系运算符）

    同java，><=!=,比较的时候左右类型要保持一致

  * 逻辑运算符

    返回boolean

  * 位运算符

    同java

    scala 不支持三目运算符

    运算符优先级别

    （）[] 级别最高

    * 单目运算

    * 算术运算

    * 移位运算

    * 比较运算

    * 位运算

    * 关系
    * 赋值

* 键盘输入语句

* 循环守卫

  ```scala
  for(i<-1 to 3 if i!=2){
  println(i)
  }
  ```

* for循环 util 左包右不包

  ```scala
  for (i<-1  util 3){
  	println(i)
  }
  ```

  

* 循环引入变量

  ```scala
  for (i<- 1 to 3 ; j=4-i){
  	println(j)
  }
  
  ```

* 多重循环

  scala的多重循环 相当于嵌套for循环

  ```scala
  for(i<-1 to3 ; j<-3 to 10){
         println(i)
   
         println(j)
  }
  ```

  

* 循环返回值 使用yeild

  将 1-3遍历 将循环得到的i放到vector中 作为变量result  yield 表示将哪些逻辑作为返回  yield后可以接代码块

  ```scala
  var result= for(i<-1 to 3) yield i
  println(result)
  
  var res=for (i<-3 to 10) yield  i+1
  println(res)
  ```

  

* for循环 步长 

  以下代码 1到10 每隔步长打印

  同样可以用循环守卫实现

  ```scala
  for (i <- Range(1,10,2)){
  println(i)
  }
  ```

* while循环没有返回值 返回的是（）即Unit

* break continue 不支持 在 `util.control.Breaks._` 使用 break() ,breakable(op:=>Unit)

* 高阶函数传入代码块时候 小括号会换成大括号 

* 使用if-else 或循环守卫实现continue

* 函数式编程

  * 函数定义声明

    scala中函数和方法几乎等同 

    函数的形参列表和返回值列表数据类型可以是值类型或引用类型

    返回值加return 函数就不能自动推断返回类型 

    如果没有声明函数返回类型 即使加了return 也没有返回值

    如果不确定返回值 可用用Any类型

    可以在函数中定义函数 方法中定义方法

    如果函数形参有默认值 不传参那么走默认值 否则传参会覆盖默认值 从左到右  也可以使用带名参数进行覆盖指定形参

    `mysql(host="127.0.0.1",pwd="123")`

    形参类型默认为val

    scala支持可变参数,可变参数只能出现在形参列表的最后 使用* 来表示可变参数,可变参数其实是个序列集合

    ```scala
      def sum(n1: Int, args: Int*): Int = {
    
        for ( item<-args  ) {
          println(item)
        }
        args.length
      }
    ```

    

  * 函数运行机制

    栈与堆内存的调用

  * **递归 （推荐使用递归）**

    递归函数一定要指定返回类型

  * 过程

    将函数返回Unit的函数 称为过程函数 如果没有返回值 那么 函数签名上的=可以省略

  * 惰性函数与异常

    尽可能延迟表达式求值， 需要时提供元素 无需预先计算他们 lazyLoad ,使用 lazy 关键字 只能修饰val 

    只有当lazy变量被使用时候 才会加载使用

    ```scala
      def main(args: Array[String]): Unit = {
        
        lazy  val lazyV = sum(1,3);
        println(lazyV)
        
      }
    ```

    scala的异常和java稍微有区别 使用 try catch finally 

    1.运行时异常 2.编译（checked）异常 java中的区分

    scala中只有运行时异常,使用case 匹配 不同异常

    ```scala
      def mockExceptions(): Unit ={
        try{
           var i = 10/0
        }catch {
          case  ex:ArithmeticException=>{
            println("catched a ArithmeticException")
          }
          case  ex:Exception=>{
            println("catched a exception")
          }
        }finally {
          println(" finally printed")
        }
      }
    ```

    throw 表达式 是有类型的 为Nothing

    ```scala
    def throwNothing():Nothing{
     throw new Exception("Nothing")
    }
    ```

    使用@throws 注解 来表示抛出异常

    ```scala
    @throws(classOf[NumberFormatException])
    def fn1()={
    	"abc".toInt
    }
    ```

    

  * 值函数

  * 高阶函数

  * 闭包

  * 应用函数

  * 柯里化函数 抽象控制

    

* 面向对象

  * scala 对象类的属性必须赋予默认值 可用_来表示

  * 属性默认是private的 不是public的

    ```scala
    class cat{
        var name :String =""
        var age:Int =_
             }
    ```

  ​        声明成员属性时候会自动编译生成 set get 方法

     	使用下划线给定默认值 必须指定类型

  ​	    一般使用val 声明对象引用 也可以用类型推导 

  ​		**当类型和后面对象类型有继承关系即多态时就必须声明实例变量的类型** 有点像泛型

  ```scala
   val cat:Cat = new cat
  ```

  

  * scala 中的类不用刻意声明为public 默认是public

  * 成员属性的声明

    * 属性声明定义同变量

    * 定义类型可以为值或者引用

    * 声明一个属性必须显式的初始化

    * 如果赋值为null 一定要声明属性的类型 否则该属性的类型就是Null

    * 属性默认值：

      * Byte Short Long Int  _对应0
      * Float Double           _对应0.0
      * String 和引用类型  _ 对应null
      * Boolean                  _对应false

    * 使用BeanProperty注解生成set get **且 这种方式生成的方法 与底层编译后自动生成的带$的setget方法不冲突 可以并存** 

      ```scala
      import scala.beans.BeanProperty
      
      class A(){
        @BeanProperty
        var name:String=""   
      } 
      ```

      

  * scala 翻转循环

    ```scala
    for(i<-0 to 10 reverse){
    	println(i)
    }
    ```

    

    

* scala的方法

  方法就是函数 方法一般为对象的成员

  ```scala
  class cat{
  	def eat()={
  	println("eat")
  	}
  }
  ```

* 构造器

  java的构造器有固定要求 名字与类名一样 用于初始化对象 有默认无参构造器 super this .etc

  scala的构造器,多个构造器可以重载  有**主构造器和辅构造器**

  使用this 表示辅助构造器 可以根据参数列表来区分不同的辅助构造器

  主构造器直接放在类名上声明

  **主构造器会执行类定义中的所有语句，放在类中要执行的语句 除了声明的成员方法**

  如果想让主构造器变成私有 那么可以在class的()前加private 这样只能通过辅助构造器去构造对象了

  **主构造器的形参如果没有用任何修饰符修饰那么 这个参数是局部变量，如果主构造器的参数使用val进行修饰那么 这个属性就会成为一个私有只读属性** 变成immutable了 只读为只有get方法

  ```scala
  class A(inName:String){
   var name=inName;
  }
  class B(val inName:String){
   var name=inName;
  }
  ```

  

  ```scala
  //私有主构造器
  class Person private(inName:String,inAge:Int){
   var name:String=inName
   var age:Int=inAge
   
     def this(inName:String){
         inName=name
     }
      override def toString:String={
          "name"+name+"age"+age
      }
   
  }
  
  ```

  辅助构造器 使用this声明 和java中的this不一样,多个辅助构造器通过不同的形参列表进行区分 底层为java的构造器重载

  **辅助构造器内第一行代码必须显式的调用主构造器**  这样做是为了显式的调用主构造器的父类构造方法

  不能在辅助构造器调用该类的父类的主构造器  只有主构造器才能调用父类构造器 不能使用super.

  ```scala
  //主构造器
  class Person(inName:String,inAge:Int){
   var name:String=inName
   var age:Int=inAge
   
     def this(inName:String){
         inName=name
     }
      override def toString:String={
          "name"+name+"age"+age
      }
   	//辅助构造器
      def this(name:String){
          //调用主构造器 这里给的默认值 如果主构造器没有参数 那么就写this 就ok
          this(_,_)
          this.name=name
      }
  }
  
  ```

* **对象的创建过程**

  * 加载类信息 成员信息（属性及方法）及其他
  * 在内存中开辟一块堆空间
  * 使用父类构造器（主辅）进行初始化
  * 使用主构造器进行属性的初始化
  * 使用辅助构造器对属性进行再次初始化
  * 将对象堆内存的地址赋给变量引用

* scala包

  除了java中的两种方式 scala还可以在同一个文件中创建多个包

  ````scala
  package com{
  	objcet People{
  		main
  	}
  	package scala{
  	
  		class A{
  			def say()={
  				println("A")
  			}
  		}
  		package test{
  			class B{
  			 	def sayHello()={
  			 		println("B")
  			 	}
  			}
  		}
  	}
  }
  ````

  scala的包名和远吗所在的系统文件目录结构可以不一致 但是编译后端字节码文件路径和包名会保持一致 

  scala自动引入 lang包和 scala包和 preDef包

  scala的泛型使用[]中括号

  import可以写在任何位置

  包对象是对静态成员的抽象 包对象的名字要和包的名字一致

  在包对象中定义方法变量 不能直接在包的大括号中定义会报错

  ```
  package object scala{
  
  }
  pacakage scala{
  
  }
  ```

  scala中的可见性  priate表示私有 public一般为默认 

  scala伴生类是伴生对象的静态成员或者属性的抽象封装

  私有属性可以在本类中访问 也可以在伴生类中访问 

  不可以用public显式的修饰属性或者方法

  方法的访问权限默认为public

  protected只能在子类访问 同包下不能访问

  ```
  class A{
  	var name:String="" //可读写
  	private var sla:Int=0 //只读
  		
  }
  ```

  包访问权限示例，由于同包下的私有不能访问 所以使用中括号加包名表示 某个属性可以在某个包下访问

  ```scala
  class Person{
  	private [scala] val name:String=""
  }
  ```

  虽然用的protected 修饰 但是编译器生成的变量类型还是private 默认public 编译后也是private

  只是该属性的访问方法是public或者private的

* 包的引入

  * import语句可以出现在任何地方

  * 如果想要引入某个包中的所有的类 那么使用 `import scala._` 下划线

  * 在需要用的时候才引用的话 那么作用域只在该代码块所在的大括号内

  * 如果 多个包中有相同名字的类 可以重命名 使用{}进行重命名

    ```scala
    import scala.collection.mutable.HashMap 
    import java.util.{HashMap=>JavaHashMap}
    ```

* 面向对象

  子类继承了所有属性 私有属性不能直接访问 需要通过公共方法访问

  父类的protected属性 编译后也是public 

  

  * 方法重写

    使用override 关键字

    ```scala
    override def toString():String{
    	println("toString")
    }
    ```

    调用超类方法使用super关键字

  * 类型检查 多态

    * classOf[String] 获取String的class对象
    * obj.isInstanceOf[String] 判断某个对象是否是String类型
    * obj.asInstanceOf[T] 将obj对象强转为T类型

  * 复写字段

    * 使用override进行复写字段，java中没有属性或者字段的重写 （子类如果有同名的属性 那么父类引用调用的时候还是父类的属性，子类引用调用还是子类属性）scala中无论是子类引用还是父类引用 调用后都是子类属性

    * 复写字段其实是复写方法

    * 动态绑定

      java的动态绑定机制

      如果调用的是方法 那么jvm机会将该方法和对象内存地址绑定

      如果调用的是一个属性 则没有动态绑定机制 在哪里调用就返回对应的值

      scala则全部有动态绑定

      * def 只能重写另个方法 即 方法只能重写另一个方法

      * val只能重写另一个val属性或者 重写不带参数的def （因为 是get set方法） 带了参数就是方法的重载了

      * var 只能重写另一个抽象的var

        **抽象字段就是没有初始化的属性** 抽象字段要求所在类也为抽象类 标记为abstract

        编译后不会生成对应的属性声明 只会生成两个对应的抽象方法 name name_$eq

        ```scala
        abstract class A{
         	var name:String=_
        }
        class B extends A{
            var name:String=""
        }
        ```

  * 抽象类

    抽象类的属性可以没有值 没有值为抽象字段

    可以有抽象字段可以有普通字段

    抽象方法没有代码块

    可以有抽象方法 可以有普通方法

    **抽象的方法或者属性就不能用private修饰了**

    ```scala
    abstract classA{
    	var name:String
    	
    	def cry()
    	def cry(inName:String)
    }
    ```

  * 匿名子类

    ```scala
    abstract class A{
     	var name :String
     	def cry()
    }
    object anno {
    main{
    	var b = new A{
    		override var name :String=""
    		override def cry():Unit{
    			println("cry")
    		}
    	}
        b.cry()
    }
    }
    ```

  * 继承层次

    * Any 

      * AnyVal 
        * 基础类型  
      * AnyRef 
        * collections
        * javaclass 
        * other
        * Null

      * Nothing

    * Any是顶级
    * AnyRef相当于Object
    * Null  shi null 类型
    * Nothing 没有实力  泛型 底层类 

  * 伴生对象

    将静态属性和静态方法放到 该类的伴生对象中声明使用,抽象封装

    这个和包对象有异曲同工之妙

    伴生对象必须与该类同名

    伴生对象依赖`public static final MOUDLE$`

  * apply 方法

    在伴生对象定义apply方法 那么就可以实现类名（参数）方式创建对象实例

    ```scala
    class Pig(inName:String){
    var name :Stirng=inName
    }
    object Pig{
    	def apply(inName:String):Pig=new Pig(inName)
        def apply():Pig= new Pig()
    }
    
    ```

* trait

  trait等价与接口+抽象类

  动态混入 mix in

  trait  继承

  在 scala中 java的接口可以作为trait

  特质可以看作对继承的一种补充

  ```scala
  trait behav {
  	trait codes
  }
  ```

  * 如果没有父类 多个特质继承

    ```scala
    class A extends behav with traitA ...{
    
    }
    ```

    

  * 如果有父类  多个特质

    ```scala
    class B extends Father with   behav with  traitA with...{
    
    }
    ```

    当trait中有抽象和普通,会编译为两个class对象 一个trait.class接口 一个trait$.class抽象类 也是implement trait.class接口 但普通方法依赖MOUDLE来实现抽象类中的普通方法

    当trait中只有抽象方法 生成 trait.class 接口 implements

  

  

  * 使用 type 进行类型别名

  * 动态混入 

    解耦 

    scala特有 补休该类的声明定义下 扩展类的功能 

    不影响原有继承关系的基础上 给指定的类扩展功能

    ```scala
     class A{
     
     }
    abstract class B{
        def say();
    }
     object Mains{
     	main{
            //不修改类定义声明的情况下 进行动态混入
     		val a= new A with traitA
            //含有抽象方法的类动态混入
            var b = new B with traitB{
                def say (){
                    println("hi")
                }
            }
     	}
     }
    ```

  * 叠加特质

    构建对象的时候 混入多个特质  即叠加特质

    特质声明从左到右 方法执行顺序从右到左

    ```scala
    object A {
        main {
            var a = new A with traitA with traitB
            a.say()
        }
    }
    ```

    如果 traitA 和 traitB 有共同的父类继承  那么 父类的构造只会走一次，叠加特质的初始化 会从左到右

    如果traitA 和traitB 都有say 方法 那么 先走traitB 的方法 再执行 traitA的方法  遵循栈 从右向左

    当有super的方法调用的时候 指的是左边的traitA的方法 当左边再没有trait的时候 直接找父类trait的方法

    可以使用[?] 泛型 指定super[T]的调用 T类型必须是当前混入trait的父类

  * 对象构造方式

    * new 对象
    * apply
    * 匿名子类
    * 动态混入

  * 富接口

    既有抽象方法又有非抽象方法的trait

  * 特质中的具体字段

    特质中的初始化了的字段 就是具体化字段 否则是抽象字段

    混入该特质的类就拥有了该字段，该字段不是继承 而是直接加入到该类中

    特质中的抽象字段 在具体的类的继承中必须被重写初始化

  * 特质的构造顺序

    ​	普通特质继承的构造顺序

    * 调用该类超类构造器
    * 调用第一个特质的父类及爷类 构造器  依次往上推 如果执行过不再执行
    * 调用第一个特质的构造器
    * 调用第二个特质的父类及爷类 构造器  依次往上推 如果执行过不再执行
    * 调用第二个特质的构造器
    * ...

    ```scala
    class a extend b with traita with traitb{
    	
    }
    ```

    动态混入对的构造顺序

    * 先创建对象
    * 在执行特质的构造 同上面普通特质继承的实现

    ```scala
    var c1 = new A with traitA with traitB
    ```

    两种方式 的区别是是否创建了该类的对象

  * 扩展类的特质

    特质可以继承类  用来拓展该类的一些功能

    ```scala
     trait log extends Exception{
      def printE(){
    	  println(getMessage())
      }
     }
    ```

    所有混入该特质的类 会自动成为该特质所继承的超类的子类

    若某个特质已经继承了某个类 那么混入该特质的类继承的类是该特质的超类的子类 否则会出现多继承现象

  * 特质自身类型

    解决特质循环依赖问题

    限制混入该特质的类的类型

    ```scala
    class A extends Exception with log
    ```

    ```scala
    trait logger{
     this：Excption=>
     def log():Unit={
     	println(getMessage())
     }
    }
    ```

  * 嵌套类

    类中写类 类似java内部类

    scala的静态内部类 是放在伴生类中的

    ````scala
    var innerClass=new outerClass.InnerClass
    ````

    内部类访问外部类的属性

    * 外部类名.this.属性名

    * 外部类名的别名.属性名

      属性定义要在别名后面

      ```scala
      class Outer{
      	myOuter=>
      	
      	class innter {
      		var name=""
      		def ages():Unit={
      			var  outer = myOuter.age
      		}
      	}
      	var age=1
      }
       
      }
      ```

    

  * 类型投影

    屏蔽外部对象对内部类对象的影响 使用# 链接外部类与内部类

    ```scala
    def say(in:OuterClass#innerClass){
    	print("say")
    }
    ```

  

* 隐式转换和隐式值

  * 隐式转换函数 用**implicit**关键字声明**带有单个参数的函数**  可以自动将值从一种类型转换为另一种类型

  ```scala
  implicit def fnc(d:Double):Int={
  		d.toInt
  }
  ```

  ​	可以使用隐式转换给类动态添加功能

  ​    隐式转换函数跟函数名没关系 跟函数签名（入参和返回值有关系）单个类型的隐式转换函数必须是唯一的   	要不然编译器不知道使用哪个隐式转换函数

  * 隐式值 就是隐式变量 将某个形参变量标志为隐式 implicit  编译器会在方法省略隐式参数的情况下去搜索作用于内的隐式值作为默认参数

    隐式值使用 的方法调用的时候 不用带参数列表 直接方法名调用了

    隐式值和函数默认值同时存在 隐式值级别高 没有隐式值 走默认值

  ```scala
    object Mains{
        
        main{
        	implicit val str:String="A"
            def hello(implicit name:Stirng):Unit={
            	print(name)
            }
            //直接调用 不带括号 
            hello 
        }
    }
  ```

  * 隐式类

    可以用隐式类来封装隐式值和隐式方法

    * 构造器的构造参数有且只能有一个

    * 隐式类必须被定义在类或者伴生对象或者包对象中，即隐式类不能是顶级的类

    * 隐式类不能是case class即样式类

    * 作用域内不能有与之同名称的标识符

      ```scala
      implicit class DB(val n:Mysql){
      	def say():String={
      		n+"scala"
      	}
      }
      main{
          var mysql=new Mysql
          //当在DB隐式类的作用域 创建mysql对象 该隐式类就会生效
      }
      ```

  * 隐式转换时机

    **即什么时候使用隐式转换**

    * 当方法中的参数的类型与目标类型不一致
    * 当对象调用所在类中不存在的方法或成员属性时候，编译器会自动将对象进行隐式转换（根据类型）

  * 编译器如何查找隐式转换

    * 在当前代码作用域下查找隐式实体
    * 如果上述查找失败 那么继续在隐式参数的类型的作用域里查找
      * T with A with B with C  在ABC中都搜索
      * 如果T 是参数化类型  比如List[String]  会在List和String的伴生对象找
      * 如果T是个单利对象 p.T  那么 p 对象也会被搜索
      * 如果T是个类型注入 即内部类  即S#T 那么S和T 都会被搜索

  

  * 隐式转换前提

    不能存在二义性

    隐式操作不能嵌套使用 循环调用

  