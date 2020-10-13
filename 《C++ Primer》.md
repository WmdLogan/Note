# 《C++ Primer》

## 第2章 变量和基本类型

### 1、const限定符

- 顶层const：**指针本身**是个常量（int * const size）
- 底层const：**指针所指的对象**是个常量（int const * size）
- 常量表达式：值不会改变并且在**编译过程**就能得到计算结果的表达式
  - const int sz = get_size()不是常量表达式

### 2、C++11标准：constexpr类型

- 由编译器来验证变量的值是否是一个常量表达式
- constexpr int sz = size()
  - 当size()是一个constexpr函数时才是正确的声明语句
  - constexpr函数：这种函数足够简单，在编译时就可以计算其结果·
- constexpr  int *q = nullptr; q是一个**常量指针**（顶层const）

### 3、类型别名

- 使用关键字typedef
- 使用**别名声明**，用关键字using

### 4、decltype

- 它的作用是选择并返回操作数的数据类型。编译器分析表达式得到它的类型，不计算表达式的值

## 第7章 类

- 使用class和struct定义类的唯一区别就是默认的访问权限，class默认访问权限是private

## 第9章 容器适配器

- 顺序容器：vector、deque、list、forward_list、array、string
- 顺序容器适配器：queue、stack、priority_queue

## 第10章 泛型算法

- 泛型算法运行于迭代器之上而不会执行容器操作
- 只读算法：accumulate、equal（用于确定两个序列是否保存相同的值）
- 写容器算法：fill、fill_n
- 插入迭代器：back_inserter（常常用来创建一个迭代器，作为算法的目的位置来使用）
- 拷贝算法：copy（传递给copy的目的序列至少要包含与输入序列一样多的元素）
- 重排容器元素的算法：sort、unique（不删除元素，只是返回一个指向不重复值范围末尾的迭代器）
- lambda表达式：只在一两个地方使用的简单操作
  - **[capture list]** (paramieter list) -> return type **{function body}** 
  - 可以忽略参数列表和返回类型，必须永远包含捕获列表和函数体
  - auto f = [] {return 42;} 等价于指定一个空参数列表
  - 如果忽略返回类型，lambda表达式根据代码推断出返回类型，如果函数体只是一个return，则从返回类型推断而来，否则返回类型为void
  - 值捕获：被捕获的值在lambda创建时拷贝，因此随后修改不会影响到lambda内对应的值
  - 引用捕获：影响
  - 隐式捕获：&告诉编译器采用引用捕获、=告诉编译器采用值引用
- bind函数：auto g = bind(f, a, b, _2, c, _1)
  - 传递给g的参数按位置绑定到占位符，第一个参数绑定到 _1，第二个参数绑定到 _2
  - 当调用g时 ，bind将g(_1, _2) 映射为 f(a, b, _2, c, _1)
  - 新标准使用bind，弃用bind1st和bind2nd

## 第12章 动态数组

- 动态内存可以用new和delete，新标准采用智能指针shared_ptr和unique_ptr
- 最安全的方法是用make_shared标准库函数在动态内存中分配一个对象并初始化它
- allocator类定义在头文件memory中。它可以将内存分配和对象构造分离，提供一种类型感知的内存分配方法,分配的内存是原始的，未构造的。

```
allocator<T> alloc
```

## 第13章 拷贝控制

- 拷贝构造函数：一个构造函数的第一个参数是自身类型的引用，且任何额外参数都有默认值（Foo(const Foo&)）
- 拷贝赋值运算符：Foo &operator=(const Foo &);
- 如果一个类未定义自己的拷贝赋值运算符，编译器会为他生成一个**合成拷贝赋值运算符**
- 新标准：一个类可以定义一个移动构造函数和一个移动赋值运算符
- 如果一个类需要析构函数，几乎可以肯定他也需要一个拷贝构造函数和一个拷贝赋值运算符