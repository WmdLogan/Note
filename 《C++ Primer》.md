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
- 右值引用：必须绑定到右值的引用
- 定义删除的函数来阻止拷贝构造函数等：NoCopy(const NoCopy&) = delete

### 移动构造函数：

与拷贝构造函数不同，移动构造函数不分配任何新内存，接管原对象的内存。之后，将给定对象中的指针都置为nullptr。

假如用对象a初始化对象b，之后对象a就不用了，与其将a的内容拷贝一份给b，不如让b直接使用a的空间，然后让a指向nullptr，析构函数释放空间之间判空即可。

最好a初始化后就不需要了，立即析构。但是如果初始化后仍要对a进行操作，这种方法就不合适了，因此C++专门引入了移动构造函数，用a初始化b后，就将a析构。

但是移动构造函数的参数必须是一个右值引用，move语句将一个左值变成一个右值引用

🙃在移动操作之后，移后源对象必须保持有效的、可析构的状态

## 第十四章 重载运算与类型转换

### **重载的运算符**

- 是具有特殊名字的函数：函数名字 = operator+要定义的运算符号。

- 和其他函数一样，重载的运算符也包含返回类型、参数列表、函数体

- 重载运算符函数的数量与该运算符作用的运算对象数量一样多

- 当一个重载的运算符是成员函数时，this绑定到左侧运算对象，因此成员运算符函数的（显式）参数数量比运算对象的数量少一个

- 不能重载内置类型的运算符

  ```c
  int operator+(int, int)//错误
  ```

- 非成员运算符的等价调用

  ```c
  data1 + data2;
  operator+(data1, data2);
  ```

- 成员函数的等价调用

  ```c
  data1 += data2;
  data1.operator+=(data2);
  ```

- 重载运算符的返回类型通常情况下应该与其内置版本的返回类型兼容

### 重载输出运算符<<

- 通常情况下，输出运算符的第一个形参是一个非常量的ostream的引用（ostream &os），非常量是因为向流写入内容会改变其状态；引用是因为无法直接复制一个ostream对象

- 第二个形参一般来说是一个常量的引用，该常量是想要打印的类型

- operator<<一般返回它的ostream形参

  ```c++
  ostream &operator<<(ostream &os, const Sales_data &item)
  {
      os << item.isbn() << " " << item.revenue << " " << item.avg_price();
      return os;
  }
  ```


- 输入输出运算符必须是非成员函数。IO运算符通常需要读写类的非共有数据成员，所以一般被声明为友元

### 重载输入运算符

- 第一个形参是运算符将要读取的流的引用
- 第二个形参是将要读取到的（非常量）对象的引用
- 通常会返回某个给定流的引用

```c++
istream &operator>>(istream &is, Sales_data &item){
    double price;
    is >> item.bookNo >> item.units_sold >> price;
    return is;
}
```

### 递增和递减运算符

- 定义前置递增/递减运算符

  ```c++
  class StrBlobPtr{
  public:
  	StrBlobPtr& operator++();
      StrBlobPtr& operator--();
  }
  //前置运算符应该返回递增或递减后对象的引用
  ```

- 定义后置递增/递减运算符

```c++
class StrBlobPtr{
public:
	StrBlobPtr& operator++(int);
    StrBlobPtr& operator--(int);
}
//后置运算符应该返回对象的原值（递增或递减之前的值），返回的形式是一个值而非引用	
```

后置版本接受一个额外的（不被使用）int类型的形参。这个形参唯一的作用就是区分前置版本和后置版本的函数，而不是真的要在实现后置版本时参与运算

### 标准库定义的函数对象

- 在算法中使用标准库函数对象

  ```c++
  sort(a.begin(), a.end(), greater<string>())
  ```

- 标准库function类型：function<int(int, int)声明了一个function类型，表示可以接收两个int，返回一个int类型的可调用对象

```c++
function<int(int, int) f1 = add;//函数指针
function<int(int, int) f2 = divide();//函数对象类的对象(divide类重载了()运算符)
function<int(int, int) f3 = [](int i, int j)//lambda    
map<string, function<int(int, int)>> binops;//可以把以上三种可调用对象全都添加到map中
```

### 类型转换运算符

- 一个类型转换运算符必须是类的成员函数；
- 不能声明返回类型，形参列表也必须为空；
- 通常应该是const

```c++
class SmallInt {
public:
    explicit SmallInt(int i = 0):val(i){
        if (i < 0 || i > 255) {
            throw std::out_of_range("Bad SmallInt value");
        }
    }
    SmallInt& operator=(const SmallInt& smallInt){
        cout << "this is default operator = !!!" << endl;
        val = smallInt.val;
        return *this;
    }
//类型转换运算符
    explicit operator double() const { return val; }
private:
    double val;
};
int main() {
    SmallInt si(4.1);
//    si = 4.1;
    cout << (double)si + 3.0;
    return 0;
}
```

- 当类同时定义了类型转换运算符及重载运算符时特别容易产生二义性