## 编写测试 APP

### 1、C 库文件操作基本函数

编写测试 APP 就是编写 Linux 应用，需要用到 C 库里面和文件操作有关的一些函数，比如open、read、write 和 close 这四个函数。

#### open函数
open 函数原型如下：

```cpp
int open(const char *pathname, int flags)
```

open 函数参数含义如下：

**pathname**：要打开的设备或者文件名。  
**flags**：文件打开模式，以下三种模式必选其一：  

O_RDONLY  只读模式  

O_WRONLY	只写模式  

O_RDWR	读写模式  

因为我们要对 chrdevbase 这个设备进行读写操作，所以选择 O_RDWR。除了上述三种模式以外还有其他的可选模式，通过逻辑或来选择多种模式：  

O_APPEND  每次写操作都写入文件的末尾  

O_CREAT  如果指定文件不存在，则创建这个文件  

O_EXCL  如果要创建的文件已存在，则返回 -1，并且修改 errno 的值  

O_TRUNC  如果文件存在，并且以只写/读写方式打开，则清空文件全部内容  

O_NOCTTY  如果路径名指向终端设备，不要把这个设备用作控制终端。  

O_NONBLOCK  如果路径名指向 FIFO/块文件/字符文件，则把文件的打开和后继I/O 设置为非阻塞  

DSYNC  等待物理 I/O 结束后再 write。在不影响读取新写入的数据的前提下，不等待文件属性更新。  

O_RSYNC  read 等待所有写入同一区域的写操作完成后再进行。  

O_SYNC  等待物理 I/O 结束后再 write，包括更新文件属性的 I/O。  

**返回值**：如果文件打开成功的话返回文件的文件描述符。  


#### read函数
read 函数原型如下：  

```c
ssize_t read(int fd, void *buf, size_t count)
```
read 函数参数含义如下：  

**fd**：要读取的文件描述符，读取文件之前要先用 open 函数打开文件，open 函数打开文件成功以后会得到文件描述符。  

**buf**：数据读取到此 buf 中。  

**count**：要读取的数据长度，也就是字节数。  

**返回值**：读取成功的话返回读取到的字节数；如果返回 0 表示读取到了文件末尾；如果返回负值，表示读取失败。在 Ubuntu 中输入“man 2 read”命令即可查看 read 函数的详细内容。  


#### write函数
write 函数原型如下：
```c
ssize_t write(int fd, const void *buf, size_t count);
```

write 函数参数含义如下：  

**fd**：要进行写操作的文件描述符，写文件之前要先用 open 函数打开文件，open 函数打开文件成功以后会得到文件描述符。  

**buf**：要写入的数据。  

**count**：要写入的数据长度，也就是字节数。 

**返回值**：写入成功的话返回写入的字节数；如果返回 0 表示没有写入任何数据；如果返回负值，表示写入失败。在 Ubuntu 中输入“man 2 write”命令即可查看 write 函数的详细内容。  


#### close函数
close 函数原型如下：  

```c
int close(int fd);
```

close 函数参数含义如下：  

**fd**：要关闭的文件描述符。  

**返回值**：0 表示关闭成功，负值表示关闭失败。在 Ubuntu 中输入“man 2 close”命令即可查看 close 函数的详细内容。


### 2、chrdevbaseAPP.c

```c
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>
#include <string.h>


/*
 *argc:应用程序参数个数
 *argv[]:具体的参数内容，字符串形式 
 *./chrdevbaseAPP  <filename>  <1:2> 1表示读，2表示写
 * ./chrdevbaseAPP /dev/chrdevbase 1    表示从驱动里面读数据
 * ./chrdevbaseAPP /dev/chrdevbase 2    表示向驱动里面写数据
 * */
int main(int argc, char *argv[])
{
    int ret = 0;
    int fd = 0;
    char *filename;
    char readbuf[100], writebuf[100];
    static char usrdata[] = {"usr data!"};

    if(argc != 3) {
        printf("Error usage!\r\n");
        return -1;
    }

    filename = argv[1];

    fd = open(filename, O_RDWR);
    if(fd < 0 ) {
        printf("Can't open file %s\r\n", filename);
        return -1;
    }

    if(atoi(argv[2]) == 1){ /*  读 */
        /* read */
        ret = read(fd, readbuf, 50);
        if (ret < 0) {
            printf("read file %s failed!\r\n", filename);
        }
        else {
            printf("APP read data:%s\r\n", readbuf);
        }
    }

    /* write */
    if(atoi(argv[2]) == 2) { /* 写 */ 
        memcpy(writebuf, usrdata, sizeof(usrdata));
        ret = write(fd, writebuf, 50);
        if (ret < 0) {
            printf("write file %s failed!\r\n", filename);
        }
        else {

        }
    }

    /* close */
    ret = close(fd);
    if(ret < 0) {
        printf("close file %s falied!\r\n", filename);
    }

    return 0 ;

}

```

### 3、创建设备节点进行测试

驱动加载成功需要在/dev 目录下创建一个与之对应的设备节点文件，应用程序就是通过操作这个设备节点文件来完成对具体设备的操作。输入如下命令创建/dev/chrdevbase 这个设备节点文件：  

```shell
mknod /dev/chrdevbase c 200 0
```

其中“mknod”是创建节点命令，“/dev/chrdevbase”是要创建的节点文件，“c”表示这是个字符设备，“200”是设备的主设备号，“0”是设备的次设备号。创建完成以后就会存在/dev/chrdevbase 这个文件，可以使用“ls -l /dev/chrdevbase ”命令查看  

```shell
[root@luckfox ]# ls -l dev/chrdevbase 
crw-r--r--    1 root     root      200,   0 Jan  1 12:22 dev/chrdevbase
```
如果 chrdevbaseAPP 想要读写 chrdevbase 设备，直接对/dev/chrdevbase 进行读写操作即可。相当于/dev/chrdevbase 这个文件是 chrdevbase 设备在用户空间中的实现。

```shell
[root@luckfox ]# insmod chrdevbase.ko 
[root@luckfox ]# 
[root@luckfox ]# dmesg 
[ 2473.903237] chrdevbase_init
[ 2473.903274] register_chrdev
[root@luckfox ]# 
[root@luckfox ]# mknod /dev/chrdevbase c 200 0
[root@luckfox ]# 
[root@luckfox ]# ./chrdevbaseAPP /dev/chrdevbase 1
APP read data:�}��
[root@luckfox ]# 
[root@luckfox ]# dmesg 
[ 2473.903237] chrdevbase_init
[ 2473.903274] register_chrdev
[ 2518.565880] chrdevbase_open
[ 2518.565959] chrdevbase_read
[ 2518.566297] chrdevbase_release
[root@luckfox ]# 
[root@luckfox ]# ./chrdevbaseAPP /dev/chrdevbase 2
[root@luckfox ]# 
[root@luckfox ]# dmesg 
[ 2473.903237] chrdevbase_init
[ 2473.903274] register_chrdev
[ 2518.565880] chrdevbase_open
[ 2518.565959] chrdevbase_read
[ 2518.566297] chrdevbase_release
[ 2523.051944] chrdevbase_open
[ 2523.052053] chrdevbase_write
[ 2523.052078] chrdevbase_release
[root@luckfox ]# 
[root@luckfox ]# 
[root@luckfox ]# rmmod chrdevbase
[root@luckfox ]# 
[root@luckfox ]# dmesg 
[ 2473.903237] chrdevbase_init
[ 2473.903274] register_chrdev
[ 2518.565880] chrdevbase_open
[ 2518.565959] chrdevbase_read
[ 2518.566297] chrdevbase_release
[ 2523.051944] chrdevbase_open
[ 2523.052053] chrdevbase_write
[ 2523.052078] chrdevbase_release
[ 2530.509526] unregister_chrdev
[ 2530.509558] chrdevbase_exit

```

可以看到在测试时./chrdevbaseAPP /dev/chrdevbase 1对应读操作，内核则打印了read函数的内容  
./chrdevbaseAPP /dev/chrdevbase 2对应写操作，内核打印了写操作的内容



