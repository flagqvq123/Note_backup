# Module
- 可以随时加载/卸载模块，减少内存开销。
- 缺陷：兼容性；加载卸载导致内存会不断分发和回收，产生碎片。长时间的使用会导致碎片越来越多
- 显示存在的所有模块
```
# cd /lib/modules/2.6.32-22-generic/
# find . -name "*.ko"
```
- 显示已加载的模块
```
# lsmod
或者
# cat /proc/modules
```

##### Commands For Modules
```
# insmod ***.ko   /*插入一个模块*/
# rmmod ***.ko    /*卸载一个模块*/
# lsmod           /*列出模块*/
# modinfo ***.ko  /*获得模块信息*/
# modprobe
# depmod
```

##### Printk 函数:
- 在c语言中，我们使用`printf`打印信息。但是这是一个用户态的函数在内核中已经发生了改变。
- 使用函数`printk`
```
printk(KERN_INFO"Message:%s\n", arg)
```
- KERN_INFO：系统打印log具有一定优先级，具体可见[[lec3-new.pdf#page=13]]，这里使用INFO级别防止过度占用操作系统资源。

##### dmesg command:
- `dmesg`指令用于查看内核中打印的所有消息。
- `dmesg [options]`
- 展示头部和尾部的命令。

### ==A Simple Module==
- 头文件：使用内核头文件
- 入口函数与出口函数，无main函数。出入口函数名必须相同。
- 用`module_init()`和`module_exit()`指定出入口函数。 
>[!tip]+ 简单的实例
>![[lec3-new.pdf#page=21]]

##### Compiling Modules
- 写一个脚本makefile文件
	```
	obj-m := hello.o
	all:
		make -C /lib/modules/$(shell uname -r)/build M=$(shell pwd) modules
	clean:
		make -C /lib/modules/$(shell uname -r)/build M=$(shell pwd) clean
	```
	- 进入该目录，输入make命令，再输入insmod hello来加载程序。
	- insmod后，内核模块以`hello.ko`的后缀形式出现。

### 虚拟文件系统与 /proc
- `ls /proc/`是管理进程的虚拟文件系统，用来记录进程相关的信息。
	- 左边蓝色字体：代替当前正在执行的进程，数字代表pid
	- 右边白色字体：代表与系统相关的进程的信息。

- `cat /proc/version` 获得操作系统版本号，等同`uname -a`
- `cat /proc/1800/status` 获得1800进程状态
- `cat /proc/cpuinfo` 获得cpu信息
- `cat /proc/meminfo | head -10` 获得前十行内存信息
- 特别强调：
	- `/proc`是一个虚拟文件系统
	- 它是实时、动态的文件系统。
	- 可以跟踪当前的所有进程运行状态
	- 关机的时候`/proc`都被清除，再开机的时候`/proc`又被建立
- 其他特质：
	- `/proc`中所有文件都是只读状态。
	- 依旧有部分参数可以修改，但是都是一些参数类型的数字。
- `/proc/kcore` 物理内存信息
-  使用proc文件系统的优劣
	- 优点
		- 可以给用户和程序员展现出一个动态的、一致的信息。
		- 非常容易获得与进程相关的信息，易与使用而且直观。
	- 缺点
		- 构建文件系统需要开销
		- 文件的修改需要谨慎，它的存在与系统稳定性密切相关

### 在 /proc 文件系统下的模块编程
- 模块化编程只能使用系统内部的API
- 模块编译之后，它就是内核的一部分，所以尽量不要出bug，否则会影响系统运行。
- 一定要使用C语言写代码。

- 创建一个补丁(patch)
	- 面向系统安全、漏洞来开发的小代码，是为了替换现有的有问题的部分代码。