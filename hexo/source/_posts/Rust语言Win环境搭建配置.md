---
title: Rust语言Win环境搭建配置
date: 2017-11-19 01:06:28
tags: [Rust]
categories: [Rust]
---

Windows环境下Rust语言的环境搭建与IDE配置<!--more-->

> Rust 语言是Rust 是 Mozilla 开发的注重安全、性能和并发性的编程语言。“Rust”，由 web 语言的领军人物 [Brendan Eich](https://baike.baidu.com/item/Brendan%20Eich)（js 之父），Dave Herman 以及 [Mozilla](https://baike.baidu.com/item/Mozilla) 公司的 Graydon Hoare 合力开发。吸收了其他的语言的一些特性,并有自己独到之处

* 进行下载安装

  首先进入rust 的官网进行下载安装

  []: https://www.rust-lang.org/zh-CN/	"Rust官网"

  现在都用rustup来进行rust语言版本的管理因为rust语言版本更新很快

  双击rustup-init.exe后进入安装界面 选择**2 Customize installation**

  - 是否用**default host tripe** 我选是
  - 使用哪个版本(stable,nightly),我选nightly
  - 是否加入环境变量(这个后期可以修改),我选是

* 配置环境变量及中科大源

  安装完成后在C盘下-->用户-->你的登陆用户名文件夹下会多出.rust文件夹 及.cargo文件夹

  如果你不想将rust的安装目录放置在系统盘 那么可以进行文件夹迁移,同时进行修改环境变量

  - 环境变量配置:

    ```
    CARGO_HOME:D:\Cargo\.cargo
    RUSTUP_HOME:D:\Rustup\.rustup
    ```

    简单介绍下cargo ,cargo是rust语言的包管理器 有点类似java中maven等一系列项目构建工具,使用cargo可以很方便的管理rust工程.

    Path环境变量:

    ``` 
    PATH:%CARGO_HOME%/bin;
    ```

    ​

  - 添加中科大镜像源

    因为GFW的问题 rust的官方源会链接不上 那么我们加入中科大的镜像源

    在.cargo文件夹下创建一个config文件,不带任何后缀

    ```
    [registry]
    index = "https://mirrors.ustc.edu.cn/crates.io-index/"
    [source.crates-io]
    registry = "https://github.com/rust-lang/crates.io-index"
    replace-with = 'ustc'
    [source.ustc]
    registry = "https://mirrors.ustc.edu.cn/crates.io-index/"
    ```

    ​

  - 加入RUSTUP_DIST_SERVER环境变量及RUSTUP_UPDATE_ROOT环境变量

    ```
    RUSTUP_DIST_SERVER:http://mirrors.ustc.edu.cn/rust-static
    RUSTUP_UPDATE_ROOT:http://mirrors.ustc.edu.cn/rust-static/rustup
    ```



**至此rust语言安装完毕**

 *  检查 rustup安装 

    打开cmd输入

    ```
    C:\Users\andy>rustup -V
    rustup 1.7.0 (813f7b7a8 2017-10-30)
    ```

* 检查cargo安装

  ```
  C:\Users\andy>cargo -V
  cargo 0.24.0-nightly (abd137ad1 2017-11-12)
  ```

  ​

如果 提示没有默认的 toolchain 那么 install:

```
rustup install nightly-x86_64-pc-windows-gnu
```

将stable-gnu版本设为默认toolchain

```
rustup default nightly-x86_64-pc-windows-gnu
```

* 安装相关插件

  - 安装rustfmt

    ```
    C:\Users\andy>rustup show
    Default host: x86_64-pc-windows-gnu

    nightly-x86_64-pc-windows-gnu (default)
    rustc 1.23.0-nightly (6160040d8 2017-11-18)
    ```

  - 将gnu版本设为默认

    ```
    C:\Users\andy>rustup default nightly-x86_64-pc-windows-gnu
    info: using existing install for 'nightly-x86_64-pc-windows-gnu'
    info: default toolchain set to 'nightly-x86_64-pc-windows-gnu'

      nightly-x86_64-pc-windows-gnu unchanged - rustc 1.23.0-nightly (6160040d8 2017
    -11-18)
    ```

    ​

  - 安装fmt

    ```
    cargo +nightly-x86_64-pc-windows-gnu install rustfmt
    ```

    ​

  - 安装racer

    ```
    rustup component add rust-src --toolchain nightly-x86_64-pc-windows-gnu
    ```

  - 设置 rust_src_path 环境变量(注意路径)

    ```
    RUST_SRC_PATH:D:\Rustup\.rustup\toolchains\nightly-x86_64-pc-windows-gnu\lib\rustlib\src\rust\src

    ```

    ​

  - 执行 

    ```
    cargo +nightly install racer
    ```

    **在安装插件时候如果之前你安装过该插件 那么就在命令后加入 --forece 这样就会覆盖安装**

  - 安装RLS 如果nightly 已经安装了 那么只执行第二条就行了 

    ```
    rustup component add rls --toolchain nightly-x86_64-pc-windows-gnu
    rustup component add rust-analysis --toolchain nightly-x86_64-pc-windows-gnu
    ```

    ​

**至此 插件安装完毕**

* 配置IDE

  eclipse intellij idea vscode sumblime vs 都支持rust 这里只介绍idea 与vscode

  **Idea配置**

  - 下载IntelliJ 旗舰版 只有 CLion 支持调试

    进入Configureation--->Settings---->Plugins 搜索Rust,会提示无插件可用,那么Search in repositories 可以看到Rust语言插件 进行安装 然后重启 idea

    重启完毕后,可以new rust工程了

  **Vscode配置**

  - 首先安装用于调试的GDB,我这里安装的是gnu版本的rust 

    下载*VisualGDB*

    ​

  - 配置vscode首选项,可以按你自己的来

    ```
    // 将设置放入此文件中以覆盖默认设置
    {
        "terminal.integrated.shell.windows": "C:\\Windows\\Sysnative\\WindowsPowerShell\\v1.0\\powershell.exe",
        "editor.fontSize": 14,
        "editor.fontFamily": "Consolas, 'Courier New', monospace",
        "editor.tabSize": 4,
        "editor.cursorStyle": "line",
        "editor.multiCursorModifier": "alt",
        "editor.insertSpaces": true,
        "editor.wordWrap": "off",
        "rust.racerPath": "D:\\Cargo\\.cargo\\bin\\racer.exe", // Specifies path to Racer binary if it's not in PATH
        "rust.rustLangSrcPath": "D:\\Rustup\\.rustup\\toolchains\\nightly-x86_64-pc-windows-gnu\\lib\\rustlib\\src\\rust\\src", // Specifies path to /src directory of local copy of Rust sources
        "rust.rustfmtPath": null, // Specifies path to Rustfmt binary if it's not in PATH
        "rust.cargoPath": "D:\\Cargo\\.cargo/bin/cargo.exe", // Specifies path to Cargo binary if it's not in PATH
        "rust.formatOnSave": true, // Turn on/off autoformatting file on save (EXPERIMENTAL)
        "rust.checkOnSave": true,    
        "[rust]": {
            
        },
        "workbench.startupEditor": "welcomePage",
        "rust.mode": "rls",
        "rust.rustup": {
            "toolchain": "nightly-x86_64-pc-windows-gnu",
            "nightlyToolchain": "nightly-x86_64-pc-windows-gnu"
        
        },
        "rust.rls": {
            "useRustfmt":true,
            "env": {
            "RUST_LOG": "rls=debug"
            }
        }
    }
    ```

    ​

**这样就结束了,IDE配置这块以后再补充,时间太长了**

参考:

中文社区QQ群:303838735 

[]: https://zhuanlan.zhihu.com/p/27782375	"Rust 环境配置事项一览"

