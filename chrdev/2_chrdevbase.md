## 字符设备驱动注册与注销

对于字符设备驱动而言，当驱动模块加载成功以后需要注册字符设备，同样，卸载驱动模块的时候也需要注销掉字符设备。字符设备的注册和注销函数原型如下所示 :

```c
static inline int register_chrdev(unsigned int major, const char *name, 
                    const struct file_operations *fops)
 
static inline void unregister_chrdev(unsigned int major, const char *name)
```
### register_chrdev 与 unregister_chrdev函数
**register_chrdev 函数**用于注册字符设备，此函数一共有三个参数，这三个参数的含义如下：
major：主设备号， Linux 下每个设备都有一个设备号，设备号分为主设备号和次设备号两部分
name：设备名字，指向一串字符串。
fops：结构体 file_operations 类型指针，指向设备的操作函数集合变量。

**unregister_chrdev 函数**用户注销字符设备，此函数有两个参数，这两个参数含义如下：
major：要注销的设备对应的主设备号。
name：要注销的设备对应的设备名。
一般字符设备的注册在驱动模块的入口函数 xxx_init 中进行，字符设备的注销在驱动模块的出口函数 xxx_exit 中进行。 需要注意的是 register_chrdev 与 unregister_chrdev函数在注册字符设备驱动时，会将主设备号下的所有设备都注册掉，即你无法再使用次设备号了，所以在后面的Linux内核中，这种方法已经被逐渐弃用了。 

关于主设备号和次设备号： 
Linux 中每个设备都有一个设备号，设备号由主设备号和次设备号两部分 组成，主设备号表示某一个具体的驱动，次设备号表示使用这个驱动的各个设备。 
Linux 提供了一个名为 dev_t 的数据类型表示设备号， dev_t 定义在文件 include/linux/types.h 里面。 dev_t 其实就是 unsigned int 类型，是一个 32 位的数据类型。
这32位的数据构成了主设备号和次设备号两部分，其中高 12 位为主设备号，低 20 位为次设备号。因此 Linux 系统中主设备号范围为 0~4095，所以在选择主设备号的时候不要超过这个范围。

在选择设备号时，我们需要先查看当前系统中已经有哪些设备号被占用
```shell
[root@luckfox ]# cat /proc/devices 
Character devices:
  1 mem
  4 /dev/vc/0
  4 tty
  4 ttyS
  5 /dev/tty
  5 /dev/console
  5 /dev/ptmx
  7 vcs
 10 misc
 13 input
 29 fb
 81 video4linux
 89 i2c
 90 mtd
116 alsa
128 ptm
136 pts
153 spi
180 usb
189 usb_device
226 drm
243 hidg
244 mpi
245 vcodec
246 mpp_service
247 rpmb
248 watchdog
249 iio
250 media
251 rtc
252 rk_dma_heap
253 ttyFIQ
254 gpiochip
......
```

我们需要在Character devices选择一个没有被占用的设备号，例如200

在设备注册时，还需要有字符设备操作集，我们可以先按照最简的方法实现

```c
#include <linux/module.h>
#include <linux/kernel.h>
#include <linux/init.h>
#include <linux/fs.h>    //增加fs.h头文件
 
 
#define CHRDEVBASE_MAJOR   200 //主设备号
#define CHRDEVBASE_NAME   "chrdevbase" //主设备号
 
/* 打开设备 */
static int chrdevbase_open(struct inode *inode, struct file *filp)
{
    printk("chrdevbase_open\n");
    return 0;
}
 
static int chrdevbase_release(struct inode *inode, struct file *filp)
{
    printk("chrdevbase_release\n");
    return 0;
}
 
/* 从设备读取 */
static ssize_t chrdevbase_read(struct file *filp, char __user *buf, size_t cnt, loff_t *offt)
{
    printk("chrdevbase_read\n");
    return 0;
}
 
static ssize_t chrdevbase_write(struct file *filp, const char __user *buf, size_t cnt, loff_t *offt)
{
    printk("chrdevbase_write\n");
    return 0;
}
 
/*  字符设备操作集  */
struct file_operations chrdevbase_fops =
{
    .owner = THIS_MODULE,
    .open = chrdevbase_open,
    .release = chrdevbase_release,
    .read = chrdevbase_read,
    .write = chrdevbase_write,
 
};
 
 
static int chrdevbase_init(void)
{
    int ret = 0;
    printk("chrdevbase_init\n");
 
    /*注册字符设备*/
    ret = register_chrdev(CHRDEVBASE_MAJOR, CHRDEVBASE_NAME, &chrdevbase_fops);
    if(ret < 0){
        printk("chrdevbase init failed \r\n");
    }
 
    printk("register_chrdev\n");
 
    return 0;
}
 
static void chrdevbase_exit(void)
{
    /*注销字符设备*/
    unregister_chrdev(CHRDEVBASE_MAJOR, CHRDEVBASE_NAME);
    printk("unregister_chrdev\n");
 
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

编译源码，将ko文件通过adb push到板端，加载模块
经过下面的log可以看到，在设备注册后，字符设备里就有了设备号为200的chrdevbase
同样的，卸载模块后，字符设备也被注销了

```shell
[root@luckfox ]# insmod chrdevbase.ko 
[root@luckfox ]# 
[root@luckfox ]# 
[root@luckfox ]# dmesg 
[ 2255.299762] chrdevbase_init
[ 2255.299798] register_chrdev
[root@luckfox ]# 
 
[root@luckfox ]# cat /proc/devices 
Character devices:
  1 mem
  4 /dev/vc/0
  4 tty
  4 ttyS
  5 /dev/tty
  5 /dev/console
  5 /dev/ptmx
  7 vcs
 10 misc
 13 input
 29 fb
 81 video4linux
 89 i2c
 90 mtd
116 alsa
128 ptm
136 pts
153 spi
180 usb
189 usb_device
200 chrdevbase
226 drm
......
 
 
[root@luckfox ]# rmmod chrdevbase.ko 
[root@luckfox ]# 
[root@luckfox ]# 
[root@luckfox ]# dmesg 
[ 2255.299762] chrdevbase_init
[ 2255.299798] register_chrdev
[ 2352.848928] unregister_chrdev
[ 2352.848966] chrdevbase_exit
 
 
[root@luckfox ]# cat /proc/devices 
Character devices:
  1 mem
  4 /dev/vc/0
  4 tty
  4 ttyS
  5 /dev/tty
  5 /dev/console
  5 /dev/ptmx
  7 vcs
 10 misc
 13 input
 29 fb
 81 video4linux
 89 i2c
 90 mtd
116 alsa
128 ptm
136 pts
153 spi
180 usb
189 usb_device
226 drm
243 hidg
244 mpi
245 vcodec
246 mpp_service
247 rpmb
248 watchdog
249 iio
250 media
251 rtc
252 rk_dma_heap
253 ttyFIQ
254 gpiochip
......
 
 
 
```

