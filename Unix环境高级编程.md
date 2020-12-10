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

## 三、进程

### 1、进程控制块PCB

- 每个进程在内核中都有一个进程控制块(PCB)来维护进程相关的信息，Linux内核的进程控制块是`task_struct`结构体
- `/usr/src/linux-headers-4.15.0-112/include/linux/sched.h`可以查看`task_struct`结构
  - 进程ID
  - 进程的状态。有就绪、运行、挂起、停止等
  - 进程切换时需要保存和恢复的一些CPU寄存器
  - 描述虚拟地址空间的信息
  - 描述控制终端的信息
  - 当前工作目录
  - umask掩码
  - 文件描述符表，包含很多指向file结构体的指针
  - 和信号相关的信息
  - 用户id和组id
  - 会话和进程组
  - 进程可以使用的资源上限

### 2、fork

```c
#include <unistd.h>
pid_t fork(void);
```

- 创建一个子进程
- 父进程返回子进程pid、子进程返回0 
- `getpid`、`getppid`

### 3、进程共享

父子进程之间在fork之后的相异之处

- 相同：全局变量、.data、.text、栈、堆、环境变量、用户ID、宿主目录、进程工作目录、信号处理方式
- 不同：进程ID、fork返回值、父进程ID、进程运行时间、闹钟(定时器)、未决信号集
- 父子进程间遵循**读时共享、写时复制**原则
- 父子进程共享：
  - ==文件描述符（打开文件的结构体）==
  - ==mmap建立的映射区==

### 4、exec族

- `execlp`函数：加载一个进程，借助PATH环境变量
- `execl`函数：加载一个进程，通过 路径+程序名 来加载

### 5、wait

- 一个进程在终止时会关闭所有文件描述符，释放在用户空间分配的内存，但它的PCB还留着，内核在其中保存了信息：
  - 如果是正常终止则保存着退出状态
  - 如果是异常终止则保存着导致该进程终止的信号是哪个
- 这个进程的父进程可以调用`wait`或`waitpid`获取这些信息，然后彻底清除掉这个进程

- 父进程调用`wait`函数可以回收子进程终止信息。该函数有三个功能
  - 阻塞等待子进程退出
  - 回收子进程残留资源
  - 获取子进程结束状态

```c
#include <sys/wait.h>

pid_t wait(int *status);//status是传出参数，表示子进程退出状态
```

-  可使用`wait`函数传输参数status来保存进程的退出状态。借助宏函数来进一步判断进程终止的具体原因。宏函数可分为以下三组：
  - `WIFEXITED(status)`为非0 -> 进程正常结束  
    `WEXITSTATUS(status)`如上宏为真，使用此宏获取进程退出状态(exit的参数)
  - `WIFSIGNALED(status)`为非0 -> 进程异常终止  
    `WTERMSIG(status)`如上宏为真，使用此宏获取使进程终止的那个信号的编号
  - `WIFSTOPPED(status)`为非0 -> 进程处于暂停状态   
    `WSTOPSIG(status)`如上宏为真，使用此宏获取使进程暂停的那个信号的编号  
    `WIFCONTINUED(status)`为真 -> 进程暂停后已经继续运行

```c
#include <stdio.h>
#include <unistd.h>
#include <wait.h>
#include <stdlib.h>

int main(int argc, char *argv[]){
    __pid_t pid, wpid;
    int status;

    pid = fork();
    if (pid == 0) {
        printf("---I'm child, my id = %d,going to sleep 10s\n", getpid());
        sleep(10);
        printf("--------child die--------\n");
        return 73;
    } else {
        //如果子进程未终止，父进程阻塞在这个函数上
        wpid = wait(&status);
        if (wpid == -1) {
            perror("wait error");
            exit(1);
        }
        if (WIFEXITED(status)) {
        //true, child process exit normally
            printf("child exit with %d\n", WEXITSTATUS(status));
        
        }
        if (WIFSIGNALED(status)) {
        //true, child process is killed by singal
            printf("child killed with signal %d\n", WTERMSIG(status));
        }
    }
    return 0;
}

```

### 6、waitpid

```c
#include <sys/wait.h>

pid_t waitpid(pid_t pid, int *status, int options);
```

- 参数pid：
  - 正数：回收指定ID的子进程
  - -1：回收任意子进程(相当于wait)
  - 0：回收和当前调用waitpid一个组的所有子进程
  - <-1：回收指定进程组内的任意子进程
- 参数options：
  - `WNOHANG`：非阻塞回收 
  - 0：阻塞回收
- 返回值：
  - 正数：表示成功回收的子进程pid
  - 0：函数调用时参数3指定了`WNOHANG`，并且没有子进程结束
  - -1：失败，设置errno

## 四、进程间通信(IPC)

- InterProcess Communication

- Linux环境下，进程地址空间相互独立，每个进程各自有不同的用户地址空间。
- 进程和进程之间不能相互访问，要交换数据必须通过内核，在==内核中开辟一块缓冲区==，进程1把数据从用户空间拷贝到内核缓冲区，进程2再从内核缓冲区把数据读走，内核提供的这种机制称为进程间通信

- 在进程间完成数据传递需要借助操作系统提供特殊的方法。常用的有：
  - 管道（使用最简单）
  - 信号（开销最小）
  - 共享映射区（无血缘关系）
  - 本地套接字（最稳定）

### 1、管道

- 原理：管道实为内核使用环形队列机制，借助内核缓冲区(4k)实现
- 特征：
  - 伪文件
  - 管道中的数组只能一次读取
  - 数据在管道中只能单向流动
  - 只能在有公共祖先的进程中使用

```c
#include <unistd.h>

int pipe(int pipefd[2]);
```

- 参数：
  - `fd[0]`：读端
  - `fd[1]`：写端
- 成功：0，失败-1

- 管道的读写行为：
  - 读管道：
    - 管道有数据：read返回实际读到的字节数
    - 管道无数据：
      - 无写端：read返回0
      - 有写端：read阻塞等待
  - 写管道：
    - 无读端：异常终止(`SIGPIPE`导致的)
    - 有读端：
      - 管道已满：阻塞等待
      - 管道未满：返回写出的字节个数

- 管道的优劣：
  - 优点：简单
  - 缺点：只能单向通信。只能用于父子、兄弟进程(有公共祖先)间通信。FIFO有名管道可以解决此问题

### 2、mmap

- 存储映射I/O(Memory-mapped I/O)使一个磁盘文件与存储空间中的一个缓冲区相映射。于是从缓冲区中取数据，就相当于读文件中的相应字节。与此类似，讲数据存入缓冲区，则相应的字节就自动写入文件。这样，就可以在不使用read和write函数的情况下，使用地址(指针)完成I/O操作
- 使用这种方法，首先应通知内核，将一个指定文件映射到存储区域中。这个映射工作可以通过mmap函数来实现

```c
#include <sys/mman.h>

void *mmap(void *addr, size_t length, int prot, int flags,
                  int fd, off_t offset);
```

- 参数：
  - addr：指定映射区的首地址。通常传NULL，表示让系统自动分配
  - length：指定内存映射区的大小
  - prot：共享内存映射区的读写属性。`PROT_READ、PROT_WRITE、PROT_READ|PROT_WRITE`
  - flags：标注共享内存的共享属性。`MAP_SHARED、MAP_PRIVATE`
  - fd：用于创建共享内存映射区的那个文件的文件描述符
  - offset：偏移位置。必须是4k的整数倍。默认0表示映射文件全部
- 返回值：
  - 成功：映射区的首地址
  - 失败：`MAP_FAILED`，设置error
- mmap函数的保险调用方式
  - `fd = open("文件名",O_RDWR)`
  - `mmap(NULL, 有效文件大小, PROT_READ|PROT_WRITE, MAP_SHARED, fd, 0)`

- 注意事项：
  - 创建映射区的过程中，隐含着一次对映射文件的读操作
  - 映射区的释放与文件关闭无关。只要映射建立成功，文件可以立即关闭
  - 当映射文件大小为0时，不能创建映射区
  - munmap传入的地址一定是mmap的返回地址
  - 文件偏移量必须是4k的整数倍
  - mmap创建映射区出错概率非常高，一定要检查返回值

### 3、信号

- 共性：简单、不能携带大量信息、满足条件才能发送
- 特质：信号是软件层面上的“中断”。一旦信号产生，无论程序执行到什么位置，必须立即停止运行，处理信号，处理结束，再继续执行后续指令
- 所有信号的产生及处理全部都是由==内核==完成的
- 产生信号的方法：
  - 按键产生：如`Ctrl+c`
  - 系统调用产生：如kill
  - 软件条件产生：如定时器
  - 硬件异常产生：如非法访问内存(段错误)
  - 命令产生
-  递达：递送并且到达进程
- 未决：产生和递达之间的状态。主要由于阻塞(屏蔽)导致该状态
- 信号的处理方式
  - 执行默认动作。每个信号都有自己的默认处理动作
  - 忽略(丢弃)
  - 捕捉(调用用户处理函数)
-  Linux内核的进程控制块PCB是一个结构体(task_struct)，除了包含进程id、状态、工作目录、用户id、组id、文件描述符表，还包含了信号相关的信息，主要指阻塞信号集和未决信号集
- 阻塞信号集(信号屏蔽字)：将某些信号加入集合，对他们设置屏蔽，当屏蔽x信号后，再收到该信号，该信号的处理将推后(解除屏蔽后)
- 未决信号集：
  - 信号产生，未决信号集中描述该信号的位立刻翻转为1，表示信号处于未决状态。当信号被处理对应位翻转回0。这一时刻往往非常短暂
  - 信号产生后由于某些原因(主要是阻塞)不能抵达。这类信号的集合称之为未决信号集。在解除屏蔽前，信号一直处于未决状态