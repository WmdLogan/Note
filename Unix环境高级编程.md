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
- 硬链接：ln 源文件 软链接

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

 