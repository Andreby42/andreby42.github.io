---
title: Netty之ChannelOption
date: 2018-04-10 21:34:42
tags: [Netty]
categories: [Netty]
---

ChannelOption 了解下<!--more-->

#### ChannelOption 

ChannelOption 是一个配置类 继承*** AbstractConstant<ChannelOption<T>>***

它的类注释是这样写的：

```
/**
 * A {@link ChannelOption} allows to configure a {@link ChannelConfig} in a type-safe
 * way. Which {@link ChannelOption} is supported depends on the actual implementation
 * of {@link ChannelConfig} and may depend on the nature of the transport it belongs
 * to.
 *
 * @param <T>   the type of the value which is valid for the {@link ChannelOption}
 */
```

可以很明显的看到这是个配置类

```
 //构建常量配置池
 private static final ConstantPool<ChannelOption<Object>> pool = new ConstantPool<ChannelOption<Object>>() {
        @Override
        protected ChannelOption<Object> newConstant(int id, String name) {
            return new ChannelOption<Object>(id, name);
        }
    };
    /**
     * Returns the {@link ChannelOption} of the specified name. 这里是主要的方法
     */
    @SuppressWarnings("unchecked")
    public static <T> ChannelOption<T> valueOf(String name) {
        return (ChannelOption<T>) pool.valueOf(name);
    }
    //具体的调用
    /**
     * Shortcut of {@link #valueOf(String) valueOf(firstNameComponent.getName() + "#" + 			secondNameComponent)}.
     */
    @SuppressWarnings("unchecked")
    public static <T> ChannelOption<T> valueOf(Class<?> firstNameComponent, String secondNameComponent) {
        return (ChannelOption<T>) pool.valueOf(firstNameComponent, secondNameComponent);
    }
    //具体的常量配置
        public static final ChannelOption<Integer> CONNECT_TIMEOUT_MILLIS = valueOf("CONNECT_TIMEOUT_MILLIS");

```

下面解释每项参数：

* ChannelOption.SO_BACKLOG

  ChannelOption.SO_BACKLOG 对应的是 tcp/ip 协议 listen 函数中的 backlog 参数，函数 listen(int socketfd,int backlog) 用来初始化服务端可连接队列，服务端处理客户端连接请求是顺序处理的，所以同一时间只能处理一个客户端连接，多个客户端来的时候，服务端将不能处理的客户端连接请求放在队列中等待处理，***backlog 参数指定了队列的大小***

* ChannelOption.SO_REUSEADDR

  ChanneOption.SO_REUSEADDR 对应于套接字选项中的 SO_REUSEADDR，这个参数***表示允许重复使用本地地址和端口***，比如，某个服务器进程占用了 TCP 的 80 端口进行监听，此时再次监听该端口就会返回错误，使用该参数就可以解决问题，该参数允许共用该端口，这个在服务器程序中比较常使用，比如某个进程非正常退出，该程序占用的端口可能要被占用一段时间才能允许其他进程使用，而且程序死掉以后，内核一需要一定的时间才能够释放此端口，不设置 SO_REUSEADDR

* ChannelOption.SO_KEEPALIVE

  这个就比较常见了，Channeloption.SO_KEEPALIVE 参数对应于套接字选项中的 SO_KEEPALIVE，该参数用于设置 TCP 连接，***当设置该选项以后，连接会测试链接的状态***，这个选项用于可能长时间没有数据交流的连接。当设置该选项以后，如果在两小时内没有数据的通信时，TCP 会自动发送一个活动探测数据报文

* ChannelOption.SO_SNDBUF 和 ChannelOption.SO_RCVBUF

  ChannelOption.SO_SNDBUF 参数对应于套接字选项中的 SO_SNDBUF，ChannelOption.SO_RCVBUF 参数对应于套接字选项中的 SO_RCVBUF 这两个参数***用于操作接收缓冲区和发送缓冲区的大小***，接收缓冲区用于保存网络协议站内收到的数据，直到应用程序读取成功，发送缓冲区用于保存发送数据，直到发送成功。

* ChannelOption.SO_LINGER

  ChannelOption.SO_LINGER 参数对应于套接字选项中的 SO_LINGER,Linux 内核默认的处理方式是当用户调用 close（）方法的时候，函数返回，在可能的情况下，尽量发送数据，不一定保证会发生剩余的数据，造成了数据的不确定性，***使用 SO_LINGER 可以阻塞 close() 的调用时间，直到数据完全发送***

* ChannelOption.TCP_NODELAY

  ChannelOption.TCP_NODELAY 参数对应于套接字选项中的 TCP_NODELAY, 该参数的使用与 Nagle 算法有关

  Nagle 算法是将小的数据包组装为更大的帧然后进行发送，而不是输入一次发送一次, 因此在数据包不足的时候会等待其他数据的到了，组装成大的数据包进行发送，虽然该方式有效提高网络的有效负载，但是却造成了延时，而***该参数的作用就是禁止使用 Nagle 算法，使用于小数据即时传输***，于 TCP_NODELAY 相对应的是 TCP_CORK，该选项是需要等到发送的数据量最大的时候，一次性发送数据，适用于文件传输。

* ChannelOption.ALLOW_HALF_CLOSURE

  可用于防止 Netty 在 `SocketChannel.read(..)` 返回 `-1` 时自动关闭连接

* ChannelOption.AUTO_READ

  `ChannelOption.AUTO_READ` 为 true 时，在 Channel Active 事件或读完成事件之后，Netty 便将 NIO channel 所对应的 SelectionKey 加上 OP_READ `selectionKey.interestOps(interestOps | readInterestOp)`

  ***有时通道建立起来之后并不想读取消息，也许只是发消息，或者手工，或者特定条件下读取消息，这是就不应该设置 ChannelOption.AUTO_READ***。`NioServerSocketChannel` 在接受客户端连接之后，便会在客户端连接对应的 Selector 上添加 `OP_READ`，所以不用设置 `ChannelOption.AUTO_READ`。

* ChannelOption.SO_TIMEOUT

  等待客户端链接超时时间设置

* ChannelOption.WRITE_BUFFER_WATER_MARK 

  替代了原来的WRITE_BUFFER_LOW_WATER_MARK 与 WRITE_BUFFER_HIGH_WATER_MARK

  意为设置应用支持的连接数的buffer水位控制

* ChannelOption.CONNECT_TIMEOUT_MILLIS 

  链接超时毫秒数 注意和bootstrap.connect(address).await(1000, TimeUnit.MILLISECONDS) 这个await 要适应

  ​

  ​