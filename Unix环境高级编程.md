# Unix环境高级编程

## 一、Linux基础知识

### 1、Linux系统目录

- bin：存放二进制可执行文件
- boot：存放开机启动程序
- dev：存放设备文件
- home：用户
- etc：用户信息和系统配置文件
- lib：库文件
- usr：用户资源管理目录

### 2、Linux系统文件类型

- 普通文件：-
- 目录文件：d
- 字符设备文件：c
- 块设备文件：b
- 软连接：l
- 管道文件：p
- 套接字：s

### 3、硬链接/软链接

- 软链接：ln -s 源文件 软链接
- 硬链接：ln 源文件 硬链接

### 4、find

- 路径
- -name ''
- -maxdepth n
- -type
- -size +20M -size -50M
- -ctime -atime -mtime
- -exec：将搜索的结果集执行某一指令  
  `find /usr/ -name '*.tmp' -exec ls -ld {} \;`
- -xargs：将搜索的结果集执行某一指令。当结果集数量过大时，可以分片映射  
  `find /usr/ -name '*/tmp' | xargs ls -ld`
- -print0：打印的拆分依据为null，默认拆分依据为空格  
  `find /usr/ -name '*/tmp' -print0 | xargs -print0 ls -ld`

### 5、grep

- -r：递归搜索
- -n：行号
- ps aux | grep ''

### 6、vim

- 命令模式->文本模式：i、a、o、I、A、O、s、S
- 文本模式->命令模式：ESC
- 命令模式->末行模式：:
- 末行模式->命令模式：ESC两次
- 跳转到指定行：
  - 88G（命令模式）
  - :88（末行模式）
- 跳转到文件首：gg（命令模式 ）
- 跳转到文件尾：G（命令模式 ）
- 自动格式化：gg=G（命令模式 ）
- 大括号对应：%（命令模式 ）
- 删除单个字符：x（命令模式 ）执行结束，工作模式不变
- 替换单个字符：r+替换字符（命令模式 ）执行结束，工作模式不变
- 光标移至行首：0（命令模式 ）执行结束，工作模式不变
- 光标移至行尾：$（命令模式 ）执行结束，工作模式不变
- 删除一个单词：==dw==（命令模式）光标置于首字母
- 删除光标至行尾：D或d$（命令模式）
- 删除光标至行首：d0
- 删除指定区域：按V切换为可视模式，然后按hjkl选择区域
- 删除多行：数字dd
- 复制：yy
- 剪切：同删除
- 粘贴：p（往后粘贴）、P（往前粘贴）
- 查找：
  - /+关键字，n检索下一个
  - 光标置于单词上，按*或者#
- 替换：光标置于待替换行，进入末行模式
  - :s /原数据/新数据
  - :%s /原数据/新数据/g，不加g只替换每行的首个
  - :29,35s  /原数据/新数据/g
- 撤销：命令模式下按u
- 反撤销：CTRL + r

### 7、gcc

- `gcc -E`：预处理。展开宏、头文件，替换条件编译，删除注释、空行、空白
- `gcc -S`：预处理、编译。检查语法规范（消耗时间、系统资源最多）
- `gcc -c`：预处理、编译、汇编。将汇编指令翻译成机器指令
- 无参数：链接。数据段合并和地址回填
- `gcc -o`：生成文件命名
- `-I`：指定头文件目录位置
- `-g`：编译时添加调试文件
- `-wall`：显示所有警告信息

### 8、静态库和动态库

- 静态库：对空间要求较低，而时间要求较高的核心程序中
- 动态库：对时间要求较低，对空间要求较高
- 静态库制作及使用：
  - 将`.c`生成`.o`
  - 使用ar工具制作静态库：`ar rcs lib库名.a test1.o test2.o ...`
  - 编译静态库到可执行文件中：`gcc main.c lib库名.a -o a.out`
- 头文件防卫：防止头文件被重复包含

```c
#ifndef _HEAD_H_
#define _HEAD_H_
......
#endif
```

- 动态库制作及使用：
  - 将`.c`生成`.o`（生成与位置无关的代码 -fPIC）
  - `gcc -shared -o lib库名.so test1.o test2.o test3.o`
  - 编译可执行程序时，指定所使用的动态库
    - `-l`：指定库名
    - `-L`：指定库路径
  - `gcc test.c -o a.out -l 库名 -L ./lib -I ./inc`
  - 运行可执行程序：`./a.out`出错原因：
    - 链接器工作与链接阶段，工作时需要`-l`和`-L`。
    - 动态链接器工作与程序运行阶段，工作时需要提供动态库所在目录位置
    - 通过环境变量：export LD_LIBRARY_PATH=动态库路径
    - `./a.out`成功（临时生效，终端重启环境变量失效）
  - 永久生效
    - `vi ~/.bashrc`
    - `export LD_LIBRARY_PATH=动态库绝对路径`
    - `..bashrc`或者`source .bashrc`或者重启终端，让修改后的.bashrc生效

### 9、gdb

常用指令：

- `-g`：使用该参数编译，得到调试表
- `gdb a.out`
- `list n`：列出源码，根据源码指定行号设置断点
- `b 20`：在20行设置断点
- `run/r`：运行程序
- `next/n`：下一条指令（越过函数）
- `step/s`：下一条指令（进入函数）
- `point/p`：查看变量的值
- `continue`：继续执行断点后续指令
- `quit`：退出

其他指令

- `backtrace/bt`：查看函数的调用的栈帧和层级关系
- `frame/f`：切换函数的栈帧 
- `finish`：结束当前函数调用
- `set args`：设置main函数命令行参数
- `run 字串1 字串2 字串3`：设置main函数命令行参数
- `info b`：查看断点信息表
- `b 20 if i = 5`：设置条件断点
- `ptype`：查看变量类型
- `display`：设置跟踪变量
- `undisplay`：取消设置跟踪变量。使用跟踪变量的编号

### 10、Makefile

- 1个规则

目标：依赖条件

​	（一个tab缩进）命令

```makefile
ALL:a.out

a.out:hello.o add.o sub.o div1.o
	gcc hello.o add.o sub.o div1.o -o a.out
hello:hello.c
	gcc hello.c -o hello
hello:add.c
	gcc add.c -o add
hello:sub.c
	gcc sub.c -o sub
hello:div1.c
	gcc div1.c -o div1
```

`ALL`：最重要生成的文件

- 2个函数

`src = $(wildcard *.c)`：匹配当前工作目录下的所有.c文件。将文件名组成列表，赋值给变量src

即`src = add.c sub.c div1.c`

`obj = $(patsubst %.c, %.o, $(src))`：将参数3中包含参数1的部分，替换为参数2

即`obj = add.o sub.o div1.o`

```makefile
src = $(wildcard *.c) #add.c sub.c div1.c hello.c
obj = $(patsubst %.c, %.o, $(src)) #add.o, sub.o, div1.o hello.o

ALL:a.out

a.out: $(obj)
	gcc $(obj) -o a.out
hello:hello.c
	gcc hello.c -o hello
hello:add.c
	gcc add.c -o add
hello:sub.c
	gcc sub.c -o sub
hello:div1.c
	gcc div1.c -o div1
	
clean:
	-rm -rf $(obj) a.out
```

`clean`：没有依赖。”-“的作用是删除文件不存在时不报错

- 3个自动变量

`$@`：表示规则中的目标

`$<`：表示规则中的第一个依赖条件

`$^`：表示规则中的所有依赖条件，组成一个列表，以空格隔开，如果这个列表中有重复的项则消除重复项

```makefile
src = $(wildcard ./src/*.c) #add.c sub.c div1.c hello.c
obj = $(patsubst ./src%.c, ./obj/%.o, $(src)) #add.o, sub.o, div1.o hello.o

myArgs= -Wall -g
inc_path=./inc
ALL:a.out

a.out: $(obj)
	gcc $^ -o $@ $(myArgs)
#模式规则
./obj/%.o:./src/%.c
	gcc $< -o $@ $(myArgs) -I $(inc_path)
#静态模式规则
$(obj)./obj/%.o:./src/%.c
	gcc $< -o $@ $(myArgs) -I $(inc_path)
clean:
	-rm -rf $(obj) a.out
#伪目标
.PHONY: clean ALL
```

`-n`：模拟执行make、make clean

`-f`：指定文件执行make命令

## 二、文件I/O 

### 1、open

```c
#include <fcntl.h>
int open(const char *pathname, int flags);
int open(const char *pathname, int flags, mode_t mode);

#include <unistd.h>
#include <fcntl.h>
#include <stdio.h>
#include <errno.h>
#include <string.h>

int main(){
    int fd;
    fd = open("./dict.txt", O_RDONLY);
    printf("fd = %d ,errno=%d:%s", fd, errno, strerror(errno));
    close(fd);
}
```

- `flags`是文件打开方式：`O_RDONLY | O_WRONLY | O_RDWR...`
- 使用`O_CREAT`选项时，open函数需要同时说明第三个参数mode，即文件权限
- 文件权限=`mode & ~umask`，umask默认为0002（八进制的2）
- 成功返回文件的文件描述符（整数），失败返回-1，设置errno

### 2、read/write

```c
#include <unistd.h>


/*
参数：
fd：文件描述符 buf：存数据的缓冲区 count：缓冲区大小
返回值：
0：读到文件末尾 s：读到的字节数 f：-1，设置error
*/
ssize_t read(int fd, void *buf, size_t count);
ssize_t write(int fd, const void *buf, size_t count);

#include <unistd.h>
#include <fcntl.h>
#include <stdio.h>
#include <errno.h>
#include <string.h>
#include <stdlib.h>
int main(int argc,char *argv[]){
    int fd1,fd2;
    int n;
    char buf[1024];
    fd1 = open(argv[1], O_RDONLY);//read
    if (fd1 == -1) {
        perror("open argv1 error");
        exit(1);
    }
    fd2 = open(argv[2], O_RDWR | O_CREAT | O_TRUNC, 0664);
    if (fd2 == -1) {
        perror("open argv2 error");
        exit(1);
    }
    while((n = read(fd1, buf, 1024)) != 0) {
        if (n < 0) {
            perror("read error");
            break;
        }
        int err = write(fd2, buf, n);
    }
    close(fd1);
    close(fd2);
}

```

- 库函数`fputc/fgetc`预读入缓输出
- 如果read返回-1并且`errno=EGAIN或EWOULDBLOCK`，说明不是read失败，而是read在以非阻塞的方式在读一个设备文件（网络文件），且文件无数据

### 3、文件描述符

- PCB进程控制块：本质 结构体
- 成员：文件描述符表
- 文件描述符：0/1/2/3/4..../1023 表中可用最小的
  - `0 - STDIN_FILENO `、`1 - STDOUT_FILENO`、`2 - STDERR_FILENO`

### 4、阻塞、非阻塞

- 阻塞是设备文件、网络文件的属性
- 读常规文件没有阻塞的概念
- 产生阻塞的场景：读设备文件、读网络文件。`/dev/tty` - 终端文件
- `open("/dev/tty",O_RDWR|O_NONBLOCK)` - 设置`/dev/tty`为非阻塞状态（默认为阻塞状态）

### 5、fcntl

```c
#include <unistd.h>
#include <fcntl.h>
int fcntl(int fd, int cmd, ... /* arg */ );

int flags = fcnl(fd, F_GETFL);
```

- 获取文件状态：`F_GETFL`
- 设置文件状态：`F_SETFL`

### 6、lseek

```c
#include <sys/types.h>
#include <unistd.h>
off_t lseek(int fd, off_t offset, int whence);
```

- 文件的“读”、“写"使用同一偏移位置
- 使用lseek获取文件大小
- 使用lseek拓展文件大小；要想真正拓展文件大小，必须有IO操作

###  7、stat

```c
int stat(const char *pathname, struct stat *buf);
struct stat {
    dev_t     st_dev;         /* ID of device containing file */
    ino_t     st_ino;         /* inode number */
    mode_t    st_mode;        /* protection */
    nlink_t   st_nlink;       /* number of hard links */
    uid_t     st_uid;         /* user ID of owner */
    gid_t     st_gid;         /* group ID of owner */
    dev_t     st_rdev;        /* device ID (if special file) */
    off_t     st_size;        /* total size, in bytes */
    blksize_t st_blksize;     /* blocksize for filesystem I/O */
    blkcnt_t  st_blocks;      /* number of 512B blocks allocated */
 }
```

- 参数：
  - pathname：文件路径
  - buf：（传出参数）存放文件属性
- 返回值
  - 成功：0
  - 失败：-1，errno

- 获取文件大小：`buf.st_size`
- 获取文件类型：`buf.st_mode`
- 符号穿透：`stat`会，`lstat`不会

### 8、目录(项)

- 目录也是文件。其文件内容是该目录下的所有子文件的目录项。可以尝试用vi打开一个目录
- 目录项(dentry)本质依然是结构体，重要成员变量有两个：文件名和i结点编号，而文件内容保存在磁盘盘块中

### 9、link、unlink

```c
#include <unistd.h>

int link(const char *oldpath, const char *newpath);
int unlink(const char *pathname);
```

- 根据`link`、`unlink`实现mv操作
- unlink函数的特征：清楚文件时，如果文件的硬链接数到0了，没有dentry对应，但该文件仍不会马上被释放。要等到所有打开该文件的进程关闭该文件，系统才会挑时间将该文件回收
- 隐式回收：当进程结束运行时，所有该进程打开的文件会被关闭，申请的内存空间会被释放。系统的这一特征称之为隐式回收系统资源

### 10、目录操作

- `DIR* opendir(char *dp)`
- `int closedir(DIR *dp)`
- `struct dirent *readdir(DIR *dp)`

### 11、递归遍历目录

ls -R

1. 判断命令行参数，获取用户要查询的目录名
2. 判断用户指定的是否是目录
3. 读目录

```c
#include <stdio.h>
#include <unistd.h>
#include <dirent.h>
#include <stdlib.h>
#include <string.h>
#include <sys/stat.h>

void isFile(char *name);
//处理目录
void read_dir(char *dir)
{
    DIR *dp;
    struct dirent *sdp;
    char path[256];
    dp = opendir(dir);
    if (dp == NULL) {
        perror("opendir error");
        return;
    }
//读取目录项
    while ( (sdp = readdir(dp)) != NULL) {
        if(strcmp(sdp->d_name,".") == 0 || strcmp(sdp->d_name,"..") == 0) {
            continue;
        }
//目录项本身不可访问，需要拼接
        sprintf(path, "%s/%s", dir, sdp->d_name);
//判断文件类型
        isFile(path);
    }
    closedir(dp);
    return;
}

void isFile(char *name)
{
    int ret = 0;
    struct stat sb;
//获取文件属性
    ret = stat(name, &sb);
    if (ret == -1) {
        perror("stat error");
        return;
    }
//是目录
    if (S_ISDIR(sb.st_mode)) {
        read_dir(name);
    }
//普通文件，显示名字和大小
    printf("%10s\t\t%ld\n", name, sb.st_size);
    return;
}

int main(int argc, char *argv[])
{
//判断命令行参数
    if (argc == 1) {
        isFile(".");
    } else {
        isFile(argv[1]);
    }
    return 0;
}

```

### 12、dup和dup2

```c
#include <unistd.h>

int dup(int oldfd);
int dup2(int oldfd, int newfd);
```

- 返回新文件描述符
- `fcnt`函数实现dup：`fcntl(fd, F_DUPFD,...)`

