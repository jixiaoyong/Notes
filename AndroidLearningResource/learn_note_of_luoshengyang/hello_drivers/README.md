# 从底层设备到APP调用整个过程简要分析
> 包括在Android的Linux内核空间添加硬件驱动程序（1）、在Android的硬件抽象层添加硬件接口（2）、在Android的Application Frameworks层提供硬件服务（3，4，5）以及在Android的应用层调用硬件服务的整个过程（6）
> 参考资料：[Android硬件抽象层（HAL）概要介绍和学习计划](https://blog.csdn.net/Luoshengyang/article/details/6567257)

----

**物理设备/内核驱动**

----

# 1.虚拟设备、驱动
> 采用的是系统提供的方法和驱动程序进行交互:通过proc文件系统和devfs文件系统的方法
> 完成这个内核驱动程序后，便可以在Android系统中得到三个文件，分别是/dev/hello、/sys/class/hello/hello/val和/proc/hello

`kernel/derivers/hello`

* hello.c 
* hello.h

定义设备，并提供读写方法

# 2.在android中用可执行程序 打开/读写设备
> 通过自己编译的C语言程序来访问/dev/hello文件来和hello驱动程序交互

`AOSP/external/hello`

* hello.c

编译后在机器的system/bin，可以用来访问硬件设备内核驱动程序

----

**提供java接口**

----

> 在Android的Application Frameworks中，增加Java接口来访问Linux内核驱动程序

# 3.添加硬件抽象层模块
> 在Android系统为我们自己的硬件增加了一个硬件抽象层模块hello.default

`AOSP/hardware/libhardware/include/hardware/`

* hello.h

* hello.c

编译后在AOSP system/lib/hw/hello.default.so

# 4.定义jni方法
> 编写JNI方法和在Android的Application Frameworks层增加API接口，让上层Application访问我们的硬件
> 可以通过Android系统的Application Frameworks层提供的硬件服务HelloService来调用这些JNI方法，进而调用低层的硬件抽象层接口去访问硬件

`AOSP/frameworks/base/services/jni`

* com_android_server_HelloService.cpp

重新打包的system.img镜像文件包含刚才编写的JNI方法

# 5.定义AIDL方法，将其调用APP用来于硬件服务通信的服务联系在一起
> 重新打包后的system.img系统镜像文件就在Application Frameworks层中包含了我们自定义的硬件服务HelloService，
> 并且会在系统启动的时候，自动加载HelloService。这时，应用程序就可以通过Java接口来访问Hello硬件服务

* AOSP/frameworks/base/core/java/android/os/IHelloService.aidl
* AOSP/frameworks/base/service/java/com/android/server/HelloService.java

----

**调用java接口**

----

# 6.在APP中调用该服务
通过`IHelloService hello = (IHelloService)getSystem("hello")`即可获取该服务实例，并调用其方法。
