用addr2line可以将函数地址解析为函数名，在抓取调堆栈时Java层的堆栈本身就是显示函数名与行数，这个不需要转换，但对于native和kernel层的则是函数地址，需要借助addr2line来进行转换。 接下来分析介绍一下这个地址转换方法

## 一、Native地址转换

首先获取symbols表，要找到对应的版本的symbols，以及对应版本的addr2line，这样才能确保完全匹配。 然后执行如下命令，即可转换函数名：

64位：

```
cd prebuilts/gcc/linux-x86/aarch64/aarch64-linux-android-4.9/bin
./aarch64-linux-android-addr2line -f -C -e libxxx.so  <addr1> <addr2> ...
```

32位：

```
cd /prebuilts/gcc/linux-x86/arm/arm-linux-androideabi-4.9/bin
./arm-linux-androideabi-addr2line -f -C -e libxxx.so  <addr1> <addr2> ...
```

## 二、Kernel地址转换

（1）首先，获取符号地址，比如获取epoll_wait的符号地址：

```
prebuilts/gcc/linux-x86/arm/arm-eabi-4.8/bin/arm-eabi-nm  out/target/product/cancro/obj/KERNEL_OBJ/vmlinux |grep epoll_wait
```

该命令执行后，可获取sys_epoll_wait命令的符号地址，如下`c02b2f28 T sys_epoll_wait`

（2）然后，计算地址

例如 [<0000000000000000>] SyS_epoll_wait+0x2a0/0x324

则计算后的地址c02b2f28 + 2a0 = 目标地址。而对于kernel来说都是通过vmlinux来获取的，这时再执行命令可转换函数名

```
./aarch64-linux-android-addr2line -Cfe  /out/target/product/cancro/obj/KERNEL_OBJ/vmlinux [目标地址]
```

（3）实例

```Java
aarch64-linux-android-nm out/target/product/gemini/obj/KERNEL_OBJ/vmlinux | grep binder_thread_read
aarch64-linux-android-addr2line -f -C -e out/target/product/gemini/obj/KERNEL_OBJ/vmlinux ffffffc000aa8cb4
```

------