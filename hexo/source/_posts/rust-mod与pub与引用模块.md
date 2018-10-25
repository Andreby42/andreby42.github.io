---
title: rust-mod与pub与引用模块
date: 2018-07-16 20:28:32
tags: [Rust]
categories: [Rust]
---

Rust-mod与pub与引用模块<!--more-->

了解rust项目的工程构建对于以后构建项目或者阅读项目代码有非常重要的意义。

#### 创建crate

我们将通过使用 Cargo 创建一个新项目来开始我们的模块之旅，不过这次不再创建一个二进制 crate，而是创建一个库 crate：**一个其他人可以作为依赖导入的项目**。

 那么如何创建一个crate 呢：

```
cargo new "crate_name" --lib
```

注意这里是--lib,创建好的crate里面是没有main.rs的，只有src/lib.rs

结构目录如下：

```

Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----        2018/7/16     21:32                network
-a----        2018/7/16     21:56             66 commonclient.rs
-a----        2018/7/16     22:04            175 lib.rs
-a----        2018/7/16     22:20            156 main.rs
```

lib.rs为crate communicator的顶层声明里面声明了他的子类mod（commonclient,network)

```
 pub mod network;
 //在顶层父级中进行生命commonclient这个module
 pub mod commonclient;
    fn connect (){
        println!("internet connect u  & I " );
    } 
```

commonclient作为一个独立的mod 没有子module,声明在lib中，就无需在自己的文件中声明了。

```
pub fn connect() {
   println!("common client connect");
}
```

network作为一个拥有自己的子module的mod，声明在lib中，目录与lib同级，自己的子mod声明在network/mod.rs中

```
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        2018/7/16     21:32             62 client.rs
-a----        2018/7/16     21:15            109 mod.rs
-a----        2018/7/16     20:59             49 server.rs
```

```ru&amp;#39;s&amp;#39;t
pub mod client;
pub mod  server;
 fn connection(){
     println!("this is  a cliet-server mod" );
 }
```

#### 模块文件系统的规则:

* **如果一个叫做 `foo` 的模块没有子模块，应该将 `foo` 的声明放入叫做 *foo.rs* 的文件中。 **
* **如果一个叫做 `foo` 的模块有子模块，应该将 `foo` 的声明放入叫做 `foo/mod.rs`文件中 **

#### pub控制可见性

* **如果一个项是公有的，它能被任何父模块访问 **

* **如果一个项是私有的，它能被其直接父模块及其任何子模块访问 **

#### 引用不同模块中的名字

* ib.rs同级的目录下创建一个main.rs,在里面写入

  ```
  extern crate communicator;
  ```

  在每个mod的声明前加入pub,保证用到的mod是可见 包括要引用的函数也是pub的，调用如下：

  ```
  extern crate communicator;
  
  fn main() {
      communicator::network::client::connect();
            communicator::commonclient::connect();
  }
  ```

  这样的话使用全限定名的话会导致非常冗长，那么使用别的办法进行简化

* **使用`use`来导入作用域**

  ```
  // extern crate communicator;
  use communicator::network::client;
  
  fn main() {
     client::connect();
          //   communicator::commonclient::connect();
  }
  
  ```

* **使用glob将所有名称引入作用域**

  其他语言中比如java中也有类似的引用，那么这样就可以一下子引入改crate中的所有，官方的程序设计一书是这么说的``请保守的使用 glob：它们是方便的，但是也可能会引入多于预期的内容从而导致命名冲突。 ``

  如java中的

  ```
  import java.util.*;
  ```

  rust中这么干

  ```
  use communicator::*;
  ```

* **使用super访问父类模块**

  tests 这个mod 在起初创建这个crate的时候就存在在lib.rs中，整个的模块结构为：

  ```
  communicator
   ├── commonclient
   ├── network
   |   └── client
   |   └──server
   └── tests
  ```

  那么 我们在lib.rs中改写这个testsmod 来做test，

  使用` use super::commonclient ` 获得要引用的mod ，**此处的super指的就是当前mod tests的父模块就是communicator**

  ```
  
      #[cfg(test)]
      mod tests {
          use super::commonclient;
          #[test]
          fn it_works() {
              commonclient::connect();
          }
  
      }
  ```

  









