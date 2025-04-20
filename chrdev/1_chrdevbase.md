## 字符设备驱动简介

字符设备是 Linux 驱动中最基本的一类设备驱动，字符设备就是一个一个字节，按照字节 流进行读写操作的设备，读写数据是分先后顺序的。比如我们最常见的点灯、按键、IIC、SPI，LCD等等都是字符设备，这些设备的驱动就叫做字符设备驱动。

### chrdevbase 字符设备驱动开发实验

新建一个文件夹/home/hya/workspace/Linux_drivers/1_chrdevbase

创建驱动文件

```c
#include <linux/module.h>

static int chrdevbase_init(void)
{
printk("chrdevbase_init\n");
return 0;
}

static void chrdevbase_exit(void)
{
printk("chrdevbase_exit\n");
}

/*
模块入口和出口
*/

module_init(chrdevbase_init);  /* 入口 */
module_exit(chrdevbase_exit);  /* 出口 */

// 必须添加的许可证声明
MODULE_LICENSE("GPL");          // 使用 GPL 许可证
MODULE_AUTHOR("HYA");           // 可选（作者信息）

```

创建Makefile文件

在写Makefile文件时要注意格式的缩进，另外在rv1106平台上，要配置好交叉编译器的路径

```shell
obj-m += chrdevbase.o
 
KERNEL_BUILD_DIR := /home/hya/workspace/luckfox-pico/sysdrv/source/objs_kernel
 
KERNEL_SRC_DIR := /home/hya/workspace/luckfox-pico/sysdrv/source/kernel
PWD?=$(shell pwd)
ARCH := arm
TOOLCHAIN_DIR := /home/hya/workspace/luckfox-pico/tools/linux/toolchain/arm-rockchip830-linux-uclibcgnueabihf/bin
CROSS_COMPILE := $(TOOLCHAIN_DIR)/arm-rockchip830-linux-uclibcgnueabihf-
 
 
all:
	$(MAKE) O=$(KERNEL_BUILD_DIR) -C $(KERNEL_SRC_DIR) M=$(PWD) modules ARCH=arm CROSS_COMPILE=$(CROSS_COMPILE) 
 
clean:
	rm -f *.ko *.o *.mod *.mod.o *.mod.c *.symvers *.order
```

在~/workspace/Linux_drivers/1_chrdevbase下执行make

```shell
hya@hya:~/workspace/Linux_drivers/1_chrdevbase$ ls
chrdevbase.c   chrdevbase.mod    chrdevbase.mod.o  Makefile       Module.symvers
chrdevbase.ko  chrdevbase.mod.c  chrdevbase.o      modules.order
```

执行完成后会生成chrdevbase.ko文件

将ko文件通过adb push到rv1106板端

通过insmod和rmmod注册和卸载模块

```shell
[root@luckfox ]# insmod chrdevbase.ko 
[root@luckfox ]# dmesg 
[ 3220.091382] chrdevbase_init
[root@luckfox ]# 
[root@luckfox ]# rmmod chrdevbase.ko 
[root@luckfox ]# dmesg 
[ 3220.091382] chrdevbase_init
[ 3227.083903] chrdevbase_exit
```

