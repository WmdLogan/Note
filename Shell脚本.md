# Shell脚本

vi hello.sh

```
#!/bin/bash
cd /root/
touch helloworld.txt
echo "hello world!" >> helloworld.txt
```

chmod 777 hello.sh

./hello.sh

## 一、变量

1. 系统变量:$HOME, $PWD, $SHELL, $USER

2. 自定义变量

   定义：变量=值（A=2，注意不能有空格）

   撤销：unset A

   声明静态变量：readonly B=3

   把变量提升为全局环境变量，可供其他shell程序使用：export 变量名

3. #### 特殊变量

   1. $n

      n为数字，$0代表该脚本名称，$1-$9代表第一个到第九个参数，十以上的参数需要用大括号${10}

   2. $#

      获取输入参数个数，常用于循环

   3. $*

      代表命令行中所有的参数，$*把所有的参数看成一个整体

   4. $@

      代表命令行中所有的参数，$@把每个参数区分对待

   5. $?

      最后一次执行的命令的返回值，0代表成功，非0失败

## 二、运算符

1. $((运算式))或$[运算式]

   s=$[(2+3)*4] 

2. expr 2 + 3（注意expr运算符间必须有空格，否则按字符串处理）

   expr `expr 2 + 3` \\* 4

## 三、条件判断

[ condition +]

## 四、流程控制

1. if

   - [ 条件判断式 ]，中括号和条件判断之间必须有空格
   - if后要有空格

   if [ 条件判断式 ];then

   ​	程序

   fi
   或者

   if [ 条件判断式 ]

   ​	then

   ​		程序

   elif

   ​	then

   ​		程序

   fi

   

2. for

   ```shell
   #!/bin/bash
   #语法1
   s=0
   for((i=1;i<=100;i++))
   do
   	s=$[$s+$i]
   done
   echo $s
   
   #语法2
   for 变量 in 值1 值2 值3
   do
   	程序
   done
   #只循环一次
   for i in "$*"
   do
   	echo "$i"
   done
   
   for i in $@
   do
   	echo "$i"
   done
   ```

   

3. while

```shell
#!/bin/bash
s=0
i=1
while [ $i -le 100 ]
do
	s=$[$s + $i]
	i=$[$i + 1]
done
```

## 五、read

```shell
#!/bin/bash

read -t 7 -p "Enter your name in 7 seconds" NAME
echo $NAME
```

## 六、函数

### 6.1、系统函数

1. basename [string/pathname] [suffix]：

   basename命令会删掉string/pathname的所有前缀包括最后一个/字符，然后将剩余的字符串显示出来。

   suffix为可选项，如果指定了，会将pathname或string中的suffix去掉

2. dirname [文件绝对路径]

   从给定的包含绝对路径的文件名中去除文件名（非目录的部分），然后返回剩下的路径（目录的部分）

### 6.2、自定义函数

[ function ] funname[()]

{

​	Action;

​	[return int;]

}

funname

## 七、shell工具

### 7.1、cut 

cut -f 列号 -d 分隔符 filename

### 7.2、sed

一种流编辑器，它一次处理一行内容。处理时，把当前处理的行存储在临时缓冲区中，称为“模式空间”，接着用sed命令处理缓冲区中的内容，处理完成后，把缓冲区的内容送往屏幕。接着处理下一行，这样不断重复，直到文件末尾。文件内容并没有改变，除非你使用重定向存储输出。

sed [选项参数] 'command' filename

### 7.3、awk

一个强大的文本分析工具，把文件逐行读入，以空格为默认分隔符将每行切片，切开的部分再进行分析处理

awk [选项参数] 'pattern {action1} 	pattern {action2}...' filename

pattern：匹配模式

action：匹配成功时所执行的一系列命令

### 7.4、sort

将文件进行排序，并将排序结果标准输出