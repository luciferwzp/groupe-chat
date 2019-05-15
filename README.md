# MCR

[![Passing Status](https://github.com/Shiny-Man/img.org/blob/master/passing.svg)](https://github.com/Shiny-Man/mcr)
[![Bulid Status](https://github.com/Shiny-Man/img.org/blob/master/build.svg)](https://github.com/Shiny-Man/mcr)

## what is MCR

简单来说 MCR 是一个基于 UDP socket 的多人聊天项目，可能有人会这么问网络通信使用 UDP socket 怎么会达到多人通信且准确无误的发送给所要接收此的目标接收者

如果感兴趣请往下看

- 项目总体结构为 CS 模式
- ps:其中 C 端仅仅是用户收发信息，其中所传输的信息包括：用户在聊天界面输入的内容还有用户 id and name 一些列字段拼接而成，将其序列化后发到网络中
- ps:其中 S 端维护了一张在线列表，这张表中包括 C 端用户的信息，除此之外还维护了一个生产者线程一个消费者线程，分别担任从环形队列中 get or put datas
- 该项目中采用“异步”的操作，异步处理不用阻塞当前进程 or 线程，而是后续进行响应，用户发送的信息并不是直接被处理，而是先发往一个“消息队列”中
- 由于使用的时 UDP socket 通信，因此在发送的消息包中得多添加一些字段，所以 CS 之间通信内容不仅仅是用户所要表达的内容
- ps:用户在聊天窗口输入的内容外加消息包的头部，这样的头部可以是“对端 name or id”，所以会运用到第三方库 jsoncpp 来作序列化和反序列化，以此来达到 UDP socket 多人通信的目的
- 其多人聊天界面借助第三方库 ncurses 实现

## UDP socket 聊天原理

```html
        user                        internet                                server
        ----                           _/\_                         ---------------------
       /    \ ----------------->     _/    \_                       |                   |
       \    / <-----------------   _/        \_     生产者生产数据  |-------------      |
        ----                     _/            \_ ----------------->||生产者线程 |      |
                                |                |                  |-------------      |
        ----    send datas      |                |                  |                   |
       /    \ ----------------->|                |                  | circle     queue  |
       \    / <-----------------|                |                  |                   |
        ----    recv datas      |                |                  |-------------      |
                                |                |                  ||消费者线程 |      |
        ----                     \_            _/ <-----------------|-------------      |
       /    \ ----------------->   \_        _/     消费者消费数据  |                   |
       \    / <-----------------     \_    _/                       |                   |
        ----                           \__/                         ---------------------
```

## 项目功能

- 支持自己与自己尬聊 (PS: 基于 echoserver)
- 支持一对一消息模型
- 支持一对多消息模型
- 支持剔除用户功能 (PS: 当且仅当客户端主动退出，服务端收到 "quit" 才进行剔除)，因此也算不上是“剔除”用户吧

## 项目不足

- UDP 的可靠性问题
- 服务端发送消息时发现客户端已经离线 or 退出，导致信息丢失问题

## 项目优化

- UDP 的可靠性问题
  - 实现 sql/ack 机制来解决 UDP 的可靠性问题
  - 具体实现为：当客户端发送消息给对端，如果对端收到之后，并且显示的返回改客户端一个 ack，发送方接收到 ack 就确认传输消息成功，如果没有收到 ack，sleep(3) 再发送原来的信息给对端

- 服务端发送消息时发现客户端已经离线 or 退出，导致信息丢失问题
  - 当服务端发现客户端离线 or 中断退出时，将发送的信息回滚到数据池当中，或者实现一个发送/接收缓冲池，把即将丢失的信息回滚到改缓冲池当中

## 所得经验

个人感悟以及项目详情请见 CSDN 博客：[MCR]()
