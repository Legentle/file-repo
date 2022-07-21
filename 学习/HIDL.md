# HIDL知识回顾
[HIDL总教程](https://source.android.google.cn/devices/architecture/hidl?hl=zh-cn "安卓开发者")
## 介绍  
>HIDL（hal接口语言）适用于指定HAL和其用户之间的接口的一种接口描述语言（IDL)  

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