### 字符设备驱动简介&#xA;
字符设备是 Linux 驱动中最基本的一类设备驱动，字符设备就是一个一个字节，按照字节 流进行读写操作的设备，读写数据是分先后顺序的。比如我们最常见的点灯、按键、IIC、SPI，LCD等等都是字符设备，这些设备的驱动就叫做字符设备驱动。&#xA;
chrdevbase 字符设备驱动开发实验&#xA;
新建一个文件夹/home/hya/workspace/Linux\_drivers/1\_chrdevbase&#xA;
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

