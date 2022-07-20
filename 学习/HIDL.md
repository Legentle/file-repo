# HIDL知识回顾
[HIDL总教程](https://source.android.google.cn/devices/architecture/hidl?hl=zh-cn "安卓开发者")
## 一、介绍  
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
<font color=#00ffff size=72>需温习</font>

