# HIDL知识回顾
[HIDL总教程](https://source.android.google.cn/devices/architecture/hidl?hl=zh-cn "安卓开发者")
## 介绍  
>HIDL（hal接口语言）适用于指定HAL和其用户之间的接口的一种接口描述语言（IDL)  

HAL一般指硬件抽象层。硬件抽象层是位于操作系统 内核与硬件电路之间的接口层，其目的在于将硬件抽象化。它隐藏了特定平台的硬件接口细节,为操作系统提供虚拟硬件平台,使其具有硬件无关性,可在多种平台上进行移植。

HIDL的出现是为了更好的服务于Treble这个项目,Project Treble 将供应商实现（由芯片制造商编写的设备专属底层软件）与 Android 操作系统框架分离开来。  
HIDL 的目标是，可以在无需重新构建 HAL 的情况下替换框架。HAL 将由供应商或 SOC 制造商构建，并放置在设备的 /vendor 分区中，这样一来，就可以在框架自己的分区中通过 OTA 替换框架，而无需重新编译 HAL。  

### HAL类型
[HIDL源码分析](https://blog.csdn.net/u010164190/article/details/106477273?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522165828306616781667892282%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fspecialcolumn.%2522%257D&request_id=165828306616781667892282&biz_id=&utm_medium=distribute.pc_search_result.none-task-special_column-2~specialcolumn~first_rank_ecpm_v1~column_rank-3-8693744-null-null.pc_column&utm_term=HIDL&spm=1018.2226.3001.4417 "讲解详细")
- binder HAL  
绑定式 HAL ，通过 Binder 机制实现跨进程通信。
- passthrough HAL  
直通式 HAL ，通过 dlopen 方式加载库，也就是在同一进程直接调用。这里需要注意，默认情况下通常会使用 *Binder 化直通式 HAL*，也就是说最后仍然是跨进程 Binder 通信。

### HIDL 语法
根据设计，HIDL 语言与 C 语言类似（但前者不使用 C 预处理器）。  
如需详细了解 HIDL 代码样式，请参阅[代码样式指南](https://source.android.google.cn/devices/architecture/hidl/code-style?hl=zh-cn)。  

### HIDL术语
- Binder化：表示 HIDL 用于进程之间的远程过程调用，并通过类似 Binder 的机制来实现。另请参阅“直通式”。
- 异步回调：由 HAL 用户提供、传递给 HAL（通过 HIDL 方法）并由 HAL 调用以随时返回数据的接口。
- 同步回调：将数据从服务器的 HIDL 方法实现返回客户端。不用于返回无效值或单个原始值的方法。
<font color=#00ffff size=6>需温习</font>


## HIDL接口和软件包
>HIDL 围绕接口构建而成，而接口是在面向对象的语言中用来定义行为的抽象类型。每个接口都是软件包的一部分。
### 软件包
- 已发布的 HIDL 软件包的根目录为 hardware/interfaces 或 vendor/vendorName（例如，对于 Pixel 设备，根目录为 vendor/google）。
- 定义同一个软件包的所有文件都位于同一目录下。例如，可以在 hardware/interfaces/example/extension/light/2.0 下找到 package android.hardware.example.extension.light@2.0。
### 接口定义
除了 types.hal 之外，其他每个 .hal 文件均定义一个接口。接口通常定义如下：
~~~
interface IBar extends IFoo { // IFoo is another interface
    // embedded types
    struct MyStruct {/*...*/};

    // interface methods
    create(int32_t id) generates (MyStruct s);
    close();
};
~~~

### 导入
import 语句本身涉及两个实体：
- 导入实体：可以是软件包或接口；以及
- 被导入实体：也可以是软件包或接口。
```
import android.hardware.nfc@1.0;            //完整软件包导入
import android.hardware.example@1.0::IQuux; // import an interface and types.hal(部分导入)
import android.hardware.example@1.0::types; // import just types.hal(仅类型导入)
```
### 接口继承

### 接口哈希
这是一种旨在防止意外更改接口并确保接口更改经过全面审查的机制。这种机制是必需的，因为 HIDL 接口带有版本编号，也就是说，接口一经发布便不得再更改，但不会影响应用二进制接口 (ABI) 的情况（例如更正备注）除外。

#### 布局
每个软件包根目录（即映射到 ```hardware/interfaces ```的``` android.hardware``` 或映射到 ```vendor/foo/hardware/interfaces``` 的 ```vendor.foo```）都必须包含一个列出所有已发布 HIDL 接口文件的 current.txt 文件。
#### 使用 hidl-gen 添加哈希
可以手动将哈希添加到 current.txt 文件中，也可以使用 hidl-gen 添加。
hidl-gen 生成的每个接口定义库都包含哈希，通过调用 IBase::getHashChain 可检索这些哈希。hidl-gen 编译接口时，会检查 HAL 软件包根目录中的 current.txt 文件，查看 HAL 是否已更改：
- 如果没有找到 HAL 的哈希，则接口会被视为未发布（处于开发阶段），并且编译会继续进行。
- 如果找到了相应哈希，则会对照当前接口对其进行检查：
  - 如果接口与哈希匹配，则编译会继续进行。
  - 如果接口与哈希不匹配，则编译会暂停，因为这意味着之前发布的接口会被更改。
    - 如需在更改的同时不影响 ABI（请参阅 ABI 稳定性），必须先修改 current.txt 文件，然后编译才能继续进行。
    - 其他所有更改都应在接口的次要或主要版本升级中进行。

## 服务和数据转移
介绍如何注册和发现服务。及通过.hal文件内的接口中定义的方法将数据发送到服务。
### 注册服务
HIDL 接口服务器（实现接口的对象）可注册为已命名的服务。注册的名称不需要与接口或软件包名称相关。
### 发现服务
HIDL类调用```getService```
### 服务终止通知
如需接收通知，客户端必须：
1. 创建 HIDL 类/接口 ```hidl_death_recipient``` 的子类（在 C++ 代码中，而不是在 HIDL 中）。
2. 替换其 ```serviceDied()``` 方法。
3. 实例化 ```hidl_death_recipient``` 子类的对象。
4. 在要监控的服务上调用 ```linkToDeath()``` 方法，并传入 ```IDeathRecipient``` 的接口对象。请注意，此方法并不具备在其上调用它的终止接收方或代理的所有权。

### 服务转移
可通过调用 .hal 文件内的接口中定义的方法将数据发送到服务。具体方法有两类：
    1. 阻塞方法
    2. 单方向法

## 快速消息队列
>HIDL 的远程过程调用 (RPC) 基础架构使用 Binder 机制，这意味着调用涉及开销，需要内核操作，并且可能会触发调度程序操作。

调用函数的开销大致可分两个部分：传递参数的开销和保存当前程序上下文信息所花费的开销。

>消息队列
查阅连接:[消息队列](https://www.jianshu.com/p/780bcb80006a)
队列是一个基础的“先进先出”的数据结构，消息队列，就是一个用来存放消息的队列。主要的功能有```应用解藕```，```流量消峰```，```消息分发```，除此之前还有```保持一致性```和```方便动态扩容```等功能。    
    缺点：系统可用性降低；系统复杂性增加。

### MessageQueue类型
Android 支持两种队列类型（称为“风格”）：
 -  未同步队列：可以溢出，并且可以有多个读取器；每个读取器都必须及时读取数据，否则数据将会丢失。
 -  已同步队列：不能溢出，并且只能有一个读取器。

这两种队列都不能下溢（从空队列进行读取将会失败），并且只能有一个写入器。

未同步队列只有一个写入器，但可以有任意多个读取器。此类队列有一个写入位置；不过，每个读取器都会跟踪各自的独立读取位置。
对此类队列执行写入操作一定会成功（不会检查是否出现溢出情况），但前提是写入的内容不超出配置的队列容量（如果写入的内容超出队列容量，则操作会立即失败）。

已同步队列有一个写入器和一个读取器，其中写入器有一个写入位置，读取器有一个读取位置。写入的数据量不可能超出队列可提供的空间

### 设置 FMQ
### 使用 MessageQueue
### 零复制操作
### 通过 HIDL 发送队列

## 使用 Binder IPC
介绍了 Android 8 中对 Binder 驱动程序进行的更改，提供了有关使用 Binder IPC 的详细信息，并列出了必需的 SELinux 政策。
### 对 Binder 驱动程序进行的更改
从 Android 8 开始，Android 框架和 HAL 现在使用 Binder 互相通信。由于这种通信方式极大地增加了 Binder 流量，因此 Android 8 包含了几项改进，旨在确保 Binder IPC 的速度

多个 Binder 域（上下文）
*通用 4.4 及更高版本，包括上游*
Android 8 引入了“Binder 上下文”的概念。每个 Binder 上下文都有自己的设备节点和上下文（服务）管理器。

分散-集中
*通用 4.4 及更高版本，包括上游*
在之前的 Android 版本中，Binder 调用中的每条数据都会被复制 3 次：
- 一次是在调用进程中将数据序列化为 Parcel
- 一次是在内核驱动程序中将 Parcel 复制到目标进程
- 一次是在目标进程中反序列化 Parcel
Android 8 使用分散-集中优化将副本数量从 3 减少到 1。数据保留其原始结构和内存布局，且 Binder 驱动程序会立即将数据复制到目标进程中，而不是先在 Parcel 中对数据进行序列化。在目标进程中，这些数据的结构和内存布局保持不变，并且，在无需再次复制的情况下即可读取这些数据。

精细锁定
*通用 4.4 及更高版本，包括上游*

实时优先级继承
*通用 4.4 和通用 4.9（即将推送到上游）*

用户空间变更

通用内核的 SHA

### 使用 Binder IPC

### SELinux 政策

### HIDL 内存块