<h1>C++ Primer 笔记</h1>

## chapter_02 变量和基本类型

### 基本内置类型

基本内置类型分为 **算术类型** 和 **空类型**

算数类型包括：

- 整形：`bool`  `char`  `short`  `int`  `long`  `long long`
- 浮点型：`float`    `double`    `long double`



```c
unsigned u = 10;
int i = -42;
std::cout << u + i << std::endl; //输出 4294967264，10 + (2^32 - 42)


unsigned u1 = 42, u2 = 10;
cout << int(u2 - u1) << endl; //输出 -32，
cout << u2 - u1 << endl; //输出 4294967264，-32 + 2^32
```



<p style="border-bottom: 1px solid black;">初始化</p>

初始化不是赋值：

- 初始化的含义是创建变量时赋予其一个初始值，
- 而赋值的含义是把对象的当前值擦除，而以一个新值来替代

当创建变量未显式初始化时，程序一般会给一个默认初始值。



<p style="border-bottom: 1px solid black;">声明和定义</p>

变量声明规定了变量的类型和名字。定义除此之外，还申请了存储空间，可能会为变量赋一个初始值。

```c
extern int i; //声明而非定义 i
int j; //声明并定义 j
extern double pi = 3.14159; //定义，初始化抵消了 extern 的作用
```

- 在函数体内部，如果试图初始化一个由 extern 关键字标记的变量，将引发错误

    ```c
    void func()
    {
        extern int i = 3; //错误
        extern int j;
        j = 3; //错误
    }
    ```

- 变量能且只能被定义一次，但是可以被多次声明



<p style="border-bottom: 1px solid black;">c++ 关键字</p>

- `constexpr` ：用于声明一个编译时常量表达式，它可以在编译阶段求值，从而在运行时之前就确定其值
- `const_cast` ：从一个指向常量的指针或引用中去除 `const` 修饰符，或者添加 `const` 修饰符到一个非常量指针或引用上。
- `decltype` ：用于获取表达式的类型。`decltype` 不会执行表达式，它只是在编译阶段进行类型推断
- `dynamic_cast` ：用于执行动态类型转换的运算符。通常用于在继承体系中处理多态性（polymorphism）时，允许你在运行时确定对象的实际类型，并进行安全的类型转换。
- `explict` ：用于修饰单参数的构造函数，以防止隐式（implicit）的类型转换。通过在单参数构造函数前使用 `explicit` 关键字，你可以禁止编译器在某些情况下执行隐式类型转换
- `export` ：声明模板的定义以及如何实例化模板，以便编译器可以在不同的编译单元中实例化模板
- `mutable` ：用于修改类中被声明为 `const` 的成员变量的可变性。通过在成员变量前添加 `mutable` 关键字，你 ，而不违反 `const` 限制
- `regster` ：提示编译器将变量存储在寄存器中，以提高访问速度
- `typeid` ：用于获取表达式的运行时类型信息。通过使用 `typeid`，你可以在运行时获得一个 `std::type_info` 对象，该对象包含有关表达式类型的信息



### 复合类型

**复合类型（compound type）**是指基于其他类型定义的类型。

<p style="border-bottom: 1px solid black;">引用</p>

- 引用并非对象，只是给已经存在的对象起别名，**不占用内存**（没有自己的地址）。因此，指针无法指向引用，也无法定义引用的引用。

- 一般初始化变量时，初始值会被拷贝到新建的对象中（拷贝构造函数）。然而定义引用时，程序把引用和它的初始值**绑定（bind）**在一起。

    

<p style="border-bottom: 1px solid black;"> 指针 </p>

**指针（pointer）**是“指向（point to）”另外一种类型的复合类型。与引用类似，指针也实现了对其他对象的间接访问。

然而指针与引用相比有很多不同点：

- 指针本身就是一个对象，允许对指针赋值和拷贝，而且在指针的生命周期内它可以先后指向几个不同的对象。

- 指针无须在定义时赋初值。

    

指针存放某个对象的地址，换句话说，指针的值就是地址。
指针值应属下列 4 种状态之一：

- 指向一个对象
- 指向紧邻对象所占空间的下一个位置。（左闭右开区间，如 `it.end()`)
- 空指针，意味着指针没有指向任何对象。
- 无效指针，也就是上述情况之外的其他值。



**在大多数编译器环境下，如果使用了未经初始化的指针，则该指针所占内存空间的当前内容将被看作一个地址值。**

```c
#include <iostream>

using namespace std;

int main()
{
	int a = 3;
	int *p1 = &a;
	int *p2;
	int *p3 = p2, *p4 = p1;
	
	printf("a addr = %#x\n", &a); // a addr = 0x70fe1c
	printf("%#d  %#x   %#x\n", *p1, p1, &p1); // 3 0x70fe1c 0x70fe10
	printf("%#x  %#x\n", p2, &p2); // 0x19 0x70fe08
	printf("%#x  %#x\n", p3, &p3); // 0x19 0x70fe00
	printf("%#x  %#x\n", p4, &p4); //0x70fe1c 0x70fdf8
	
	return 0;
}
```

 

<p style="border-bottom: 1px solid black;"> 指向指针的引用 </p>

指针是对象，所以存在对指针的引用。不过，**引用本身不是一个对象，因此不能定义指向引用的指针**

```c
int i = 42;
int *p; // p 是一个 int 型指针
int *&r = p; // r 是一个对指针 p 的引用

r = &i; // r 解引用了一个指针，因此给 r 赋值 &i 就是令 p 指向 i
*r = 0; // 解引用 r 得到 i ，也就是 p 指向的对象，将 i 的值改为 0

//等价语句
p = &i;
*p = 0;
```

理解`int *&r = p;` ：

- 从右到左读，`&r` 表明 `r` 是一个引用
- 声明符的其余部分用以确定 `r` 引用的类型是什么
- `*` 说明 `r` 引用的是一个指针
- 最后，声明的基本数据类型部分指出 `r` 引用的是一个 `int` 指针



### `const` 限定符

因为 const 对象一旦创建后其值就不能再改变，所以 const 对象必须初始化。

```c
const int i = get_size(); //正确：运行时初始化
const int j = 42; //正确：编译时初始化
const int k; //错误：k 是一个未经初始化的常量
```



<p style="border-bottom: 1px solid black;"> 指针和 const </p>

- 指向常量的指针

    ```c
    int i = 42, j = 43;
    const int *p = &i;
    *p = 3; //错误，不能给 *p 赋值
    p = &j; //正确，可以给 p 赋值
    ```

- const 指针

    ```c
    int i = 3, j = 4;
    int *const p = &i;
    *p = 4; //正确，可以给 *p 赋值
    p = &j; //错误，不能给 p 赋值
    ```

    

<p style="border-bottom: 1px solid black;"> constexpr 和常量表达式</p>

**常量表达式（const expression）**是指值不会改变并且在编译过程就能得到计算结果的表达式。

字面值属于常量表达式，用常量表达式初始化的 const 对象也是常量表达式。



**字面值类型（Literal Types）**是指那些可以用于定义字面值的数据类型。

算数类型、引用和指针都属于字面值类型。可以被定义成 constexpr 。



**必须明确一点，在 constexpr 声明中如果定义了一个指针，限定符 constexpr 仅对指针有效，与指针所指对象无关**

```c++
const int *p = nullptr;		// p 是一个指向 整型常量 的 指针
constexpr int *q = nullptr; // q 是一个指向 整数 的 常量指针
```

constexpr 指针既可以指向常量，也可以指向一个非常量

```c++
constexpr int *np = nullptr;
// i j 都必须定义在函数体之外
int j = 0;
constexpr int i = 42;

constexpr const int *p = &i; // p 是常量指针，指向整型常量 i 
constexpr int *p1 = &j; // p1 是常量指针，指向整数 j
```



### 处理类型

<p style="border-bottom: 1px solid black;"> 类型别名</p>

类型别名（type alias）是一个名字，它是某种类型的同义词。

定义类型别名的两种方法：`typedef` 和 `using`



<p style="border-bottom: 1px solid black;"> auto 类型说明符</p>

auto 让编译器通过初始值来推算变量的类型。显然，auto 定义的变量必须有初始值。

auto 一般会忽略顶层 const ，同时底层 const 则会保留下来。



<p style="border-bottom: 1px solid black;"> decltype 类型指示符</p>

decltype 的作用是选择并返回操作数的数据类型。

在此过程中，编译器不会执行表达式，它只是在编译阶段进行类型推断。

```c++
const int ci = 0, &cj = ci;
decltype(ci) x = 0; // x 的类型是 const int
decltype(cj) y = x; // y 的类型是 const int& ，y 绑定到变量 x
decltype(cj) z; //错误：z 是一个引用，必须初始化
```
**值得注意的是**，引用从来都作为其所指对象的同义词出现，只有用在 decltype 处是一个例外。



- 有些**表达式**将向 `decltype` 返回一个引用类型，一般来说，这意味着该表达式的结果对象能作为一条赋值语句的左值（反过来也成立，如果表达式的求职结果是左值，则返回引用类型）。
    - 如果表达式的内容是解引用操作，则 `decltype` 将得到引用类型。
    - 如果给变量加上括号，则编译器会把它当成一个表达式，则得到引用类型。





## chapter_03 字符串、向量和数组

嘻嘻，随便看看，不做笔记









## chapter_04 表达式

### 左值和右值

（啧，只能说坑多）

一个简单的归纳：

- 当一个对象被用作右值的时候，用的是对象的值（内容）
- 当对象被用作左值的时候，用的是对象的身份（在内存中的位置）



一些运算符使用的值和返回的结果：

- 赋值运算符（=）需要一个（非常量）**左值**作为其左侧运算对象，得到的结果也仍然是一个**左值**
- 取地址运算符（&）作用于一个**左值**运算对象，返回一个指向该运算对象的指针，这个指针是一个**右值**。
- 内置解引用运算符（*）、下标运算符（[]）、迭代器解引用运算符、string 和 vector 的下标运算符的求值结果都是**左值**。
- 内置类型和迭代器的递增递减运算符（`++`、`--`）作用于**左值**运算对象，其前置版本所得的结果也是**左值**。



再看`decltype` ：

```c++
int *p;

decltype(p); 	// p 是指针，因此返回 int*
decltype(*p); 	// *p 是左值，因此返回 int&
decltype(&p);	// &p 是右值，因此返回 int**
```





### 运算符

<p style="border-bottom: 1px solid black;"> 算术运算符</p>

c++11 新标准规定商一律向 0 取整

在除法运算中，如果两个运算对象的符号相同则商为正，否则商为负

- `(-m) / n` 和 `m / (-n)` 都等于 `-(m / n)`

在取余运算中，结果的符号与 m 相同

- `m % (-n)` 等于 `m % n`
- `(-m) % n` 等于 `-(m % n)`





<p style="border-bottom: 1px solid black;"> 递增和递减运算符</p>

优先级规定了运算对象的组合方式，但是没有说明运算对象按照什么顺序求值

```c++
int i = 0;
cout << i << " " << ++i << endl; //未定义，我们不知道 ++i 什么时候求值
```



求值顺序、优先级、结合律

```c++
f() + g() * h() + j()
```

- 优先级规定，`g()` 的返回值和 `h()` 的返回值相乘
- 结合律规定，`f()` 的返回值先与 `g()` 和 `h()` 的乘积相加，所得结果再与 `j()` 的返回值相加
- 但是，**对于这些函数的调用顺序没有明确规定**



- 举个例子：

    ```c++
    auto pbeg = v.begin();
    /*
    后置递增运算符的优先级高于解引用运算符，因此
    *pbeg++ 等价于 *(pbeg++)
    
    1. pbeg++ 把 pbeg 的值加一
    2. 返回 pbeg 的初始值的副本作为其求值结果
    
    此时解引用的运算对象是 pbeg 未增加之前的值，
    最终，输出 pbeg 开始时指向的那个元素，并将指针移动一个位置
    
    */
    while(pbeg != v.end())
        cout << *pbeg++ << endl;
    ```

- 再来个例子：

    ```c++
    while(beg != s.end() && !isspace(*beg))
        *beg = toupper(*beg++); //错误：未定义行为
    
    //因为求值顺序不一定，所以求值结果取决于编译器如何解释
    ```

    



<p style="border-bottom: 1px solid black;"> 逗号运算符（comma operator）</p>

- 先对左侧的表达式求值，然后将求值结果丢弃掉
- 返回右侧表达式的值（如果右侧运算对象是左值，那么最终的求值结果也是左值）





### 类型转换

<p style="border-bottom: 1px solid black;"> 隐式转换</p>

何时发生隐式转换

- 在大多数表达式中，比 int 类型小的整形值首先提升为较大的整数类型
- 在条件中，非布尔值转换成布尔类型
- 初始化过程中，初始值转换成变量的类型；在赋值语句中，右侧运算对象转换成左侧运算对象的类型
- 如果算术运算或关系运算的运算对象有多种类型，需要转换成同一种类型。
- 函数调用时会发生类型转换





<p style="border-bottom: 1px solid black;"> 算术转换</p>

先进行整型提升，再balabala

**要理解算数转换，方法之一就是研究大量的例子**

**然而很多结果都取决于机器**





<p style="border-bottom: 1px solid black;"> 显式转换</p>

一个命名的强制类型转换具有如下形式：

```c++
/*
type		要转换的目标类型
expression	要转换的值
如果 type 是引用类型，则结果是左值
*/
cast-name<type>(experssion);
```

`cast-name`: 

- **static_cast** ：任何具有明确定义的类型转换，只要不包含底层 const ，都可以使用 static_cast
-  **dynamic_cast** ：支持运行时类型识别
- **const_cast** ：只能改变运算对象的底层 const
- **reinterpret_cast** ：通常为运算对象的位模式提供较低层次上的重新解释（不用就完事了）





## chapter_05 语句

### switch 语句

c++语言规定，不允许跨过变量的**初始化语句**直接跳转到该变量作用域内的另一个位置。

```c++
switch(flag)
{
    case true:
        string file_name; 	//错误：控制流绕过一个隐式初始化的变量
        int jval = 0;		//错误：控制流绕过一个显式初始化的变量
        int jval;			//正确：因为 jval 没有初始化
        break;
    case false:
        jval = func();		//正确：给 jval 赋一个值
        
}

switch(flag)
{
    case true:
        {
            //正确，声明语句位于语句块内部（这不是定义吗）
            string file_name = func();
        }
        break;
    case false:
        if(file_name.empty())	//错误：file_name 不在作用域之内
        
}
```

 



### try 语句块和异常处理

额，依然不会处理异常







## chapter_06 函数

函数是一个命名了的代码块。

一个典型的函数定义包括四个部分：返回类型（return type）、函数名字、形参列表和函数体。

函数三要素：返回类型，函数名，形参类型 描述了函数的接口，说明了调用该函数所需的全部信息。



### 局部对象

在 c++ 语言中，名字有作用域，对象有生命周期（lifetime）

- 名字的作用域是程序文本的一部分，名字在其中可见
- 对象的生命周期是程序执行过程中该对象存在的一段时间



<p style="border-bottom: 1px solid black;"> 自动对象</p>

我们把只存在于块执行期间的对象称为**自动对象（automatic object）**。

形参是一种自动对象。函数开始时为形参申请存储空间，一旦函数终止，形参也就被销毁。



<p style="border-bottom: 1px solid black;"> 局部静态对象</p>

**局部静态对象（local static object）**在程序的执行路径第一次经过对象定义语句时初始化，并且直到**程序**终止时才被销毁。

如果局部静态变量没有显式的初始值，它将被执行 值初始化。（初始化，一生之敌）

内置类型的局部静态变量初始化为 0 。





### 含有可变形参的函数

为了编写能处理不同数量实参的函数，c++11 新标准提供了两种主要的方法：

- 如果所有的实参类型相同，可以传递一个名为 `initializer_list` 的标准库类型
- 如果实参的类型不同，我们可以编写一种特殊的函数，即可变参数模板

c++ 还有一种特殊的形参类型（省略符），可以用它传递可变数量的实参。



<p style="border-bottom: 1px solid black;"> initializer_list 形参</p>

略略略



<p style="border-bottom: 1px solid black;"> 省略符形参</p>

省略符形参是为了便于 C++ 程序访问某些特殊的 C 代码而设置的，这些代码使用了名为 varargs 的 C 标准库功能。





### 返回类型和 return 语句

<p style="border-bottom: 1px solid black;"> 值是如何被返回的</p>

返回一个值的方式和初始化一个变量或形参的方式完全相同：**返回的值用于初始化调用点的一个临时量，该临时量就是函数调用的结果**。

返回值将被**拷贝到调用点**，如果返回类型是引用，则不会拷贝对象。



<p style="border-bottom: 1px solid black;">不要返回局部对象的引用或指针</p>

函数完成后，它所占用的存储空间也随之被释放掉。因此，函数终止意味着局部变量的引用将指向不再有效的内存区域。



<p style="border-bottom: 1px solid black;">引用返回左值</p>

函数的返回类型决定函数调用是否是左值。

- 调用一个返回引用的函数得到左值
- 其他返回类型得到右值



<p style="border-bottom: 1px solid black;"> 列表初始化返回值</p>

C++11 新标准规定，函数可以返回花括号包围的值的列表。

此处的列表可以用来表示函数返回的临时量进行初始化。

如果列表为空，临时量执行值初始化；否则，返回的值由函数的返回类型决定

```c++
vector<string> process()
{
    if(expected.empty()) return {}; //返回一个空 vector 对象
    else if(expected == actual)
        return {"functionX", "okay"}; //返回列表初始化的 vector 对象
}
```





### 返回数组指针

<p style="border-bottom: 1px solid black;">声明一个返回数组指针的函数</p>

```c++
int (*func(int i))[10];
```

- `func(int i)` 表示调用 `func` 函数时需要一个 int 类型的实参
- `(*func(int i))` 意味着我们可以对函数调用的结果执行解引用操作
- `(*func(int i))[10]` 表示解引用 `func` 的调用将得到一个大小是 10 的数组
-  `int (*func(int i))[10]` 表示数组中的元素是 int 类型



- 使用类型别名

    ```c++
    typedef int arrT[10]; // arrT 是一个类型别名，它表示的类型是含有 10 个整数的数组
    
    using arrT = int[10]; //等价声明
    
    arrT* func(int i); // func 返回一个指向含有 10 个整数的数组 的指针
    ```

- 使用尾置返回类型

    ```c++
    // func 接受一个 int 类型的实参，返回一个指针，该指针指向含有 10 个整数的数组
    auto func(int i) -> int(*)[10];
    ```

- 使用 `decltype`

    ```c++
    int odd[] = {1, 3, 5, 7, 9};
    int even[] = {0, 2, 4, 6, 8};
    
    decltype(odd) *arrPtr(int i)
    {
        return (i % 2) ? &odd : &even; //返回一个指向数组的指针
    }
    ```

    

### 函数重载

<p style="border-bottom: 1px solid black;">重载和 const 形参</p>

顶层 const 不影响传入的函数的对象。一个拥有顶层 const 的形参无法和另一个没有顶层 const 的形参区分开来

```c++
Record lookup(Phone);
Record lookup(const Phone); //错误：重复声明

Record lookup(Phone *);
Record lookup(Phone *const); //错误：重复声明
```

底层 const 则会被区分

```c++
Record lookup(Account&);
Record lookup(const Account&); //不同的声明

Record lookup(Account *);
Record lookup(const Account *); //不同的声明
```



值得注意的是，因为非常量可以转换成 const ，所以非常量对象的调用也可以适用常量对象。不过编译器会优先选择非常量版本的函数。





<p style="border-bottom: 1px solid black;"> const_cast 和重载</p>

```c++
const string &shorterString(const string &s1, const string &s2)
{
    return s1.size() <= s2.size() ? s1 : s2;
}
```

由于形参是 `const string` 的引用，我们可以对两个非常量的 `string` 实参调用这个函数，但返回的结果必须是 `const string` 的引用。为了改变这一局面，我们可以使用 `const_cast`

```c++
string &shorterString(string &s1, string &s2)
{
    auto &r = shorterString(const_cast<const string&>(s1),
                            const_cast<const string&>(s2));
    return const_cast<string &>(r);
}
```







<p style="border-bottom: 1px solid black;"> 重载与作用域</p>

```c++
string read();
void print(const string &);
void print(double); //重载 print 函数

void foorBar(int ival)
{
    bool read = false; //新作用域：隐藏了外层的 read
    string s = read(); //错误：read 是一个布尔值，并非函数
    
    void print(int); //新作用域：隐藏了之前的 print
    print("Value: "); //错误：print(const string &) 被隐藏掉了
    print(ival); //正确：当前 print(int) 可见
    print(3.14); //正确：但是调用 print(int) ，print(double) 被隐藏
}
```





### 特殊用途语言特性

<p style="border-bottom: 1px solid black;"> 默认实参</p>

- 一旦某个形参被赋予了默认值，它后面的所有形参都必须有默认值

- 局部变量不能作为默认实参

    ```c++
    int height = 30; //全局变量
    int screen(int ht = height, int wd = 80)
    {
        return ht * wd;
    }
    
    int main()
    {
        int height = 40;
        cout << screen() << endl; //输出 2400 ，默认值不受影响
        return 0;
    }
    ```

    

<p style="border-bottom: 1px solid black;">内联函数和 constexpr 函数</p>

一次函数调用包含着一系列工作：

- 调用前要先保存寄存器，并在返回时恢复
- 可能需要拷贝实参
- 程序转向一个新的位置继续执行

将函数指定为**内联函数**，通常就是将它在每个调用点上“内联地”展开，从而避免函数调用时的开销。



**constexpr 函数（constexpr function）**是指能用于常量表达式的函数。

定义 constexpr 函数需要遵守以下约定：

- 函数的返回类型及所有形参的类型都得是字面值类型
- 函数体中必须有且只有一条 `return` 语句

不遵守约定并不代表错误，只不过，函数就不一定返回常量表达式了。（这不就和定义冲突了）

（不过实际操作后，这里好像有大坑）





<p style="border-bottom: 1px solid black;">调试帮助</p>

**assert** 是一种**预处理宏**（preprocessor marco）。

所谓预处理宏其实是一个预处理变量。



`assert` 的行为依赖于一个名为 **NDEBUG** 的预处理变量的状态。

如果定义了 `NDEBUG` ，则 `assert` 什么也不做。默认状态下没有定义 `NDEBUG` ，此时 `assert` 将执行运行时检查。

```c++
//等价于在 main.c 文件的一开始写 #define NDEBUG
gcc -D NDEBUG main.c
```





### 函数匹配

**函数匹配（function matching）**是指一个过程，在这个过程中，我们把函数调用与一组重载函数中的某一个关联起来，函数匹配也叫做**重载确定（overload resolution）**。

- 第一步，选定本次调用对应的重载函数集。集合中的函数称为**候选函数（condidate function）**。候选函数具备两个特征：

    - 与被调用的函数同名
    - 声明在调用点可见

- 第二步，考察本次调用提供的实参，然后从候选函数中选出能被这组实参调用的函数。这些新选出的函数称为**可行函数（viable function）**。可行函数也有两个特征：

    - 形参数量与本次调用提供的实参数量相等

    - 每个实参的类型与对应的形参类型相同，或者能转换成形参的类型


- 第三步，**寻找最佳匹配**，即从可行函数中选择与本次调用最匹配的函数。如果有且只有一个函数满足下列条件，则匹配成功：

    - 该函数每个实参的匹配都不劣于其他可行函数需要的匹配
    - 至少有一个实参的匹配优于其他可行函数提供的匹配

    如果在检查了所有实参之后，没有一个函数脱颖而出，则编译器将报告二义性调用的信息。



举个例子：

```c++
//四个重载函数声明
void f();
void f(int);
void f(int, int);
void f(double, double = 3.14);

f(42, 2.56);
```

函数匹配过程：

- 可行函数是 `f(int, int)` 和 `f(double, double)`
- 首先匹配 `f(int, int)` ，第一个实参能精确匹配，优于 `f(double, double)` ，满足条件 2；第二个实参需要隐式转换，劣于 `f(double, double)` ，不满足条件 1 。因此，不满足最佳匹配。
- 再匹配 `f(double, double)` ，显然，不满足条件 1 。因此，不满足最佳匹配。
- 编译器最终因为这个调用具有二义性而拒绝其请求。



在调用重载函数时有三种可能的结果：

- 编译器找到一个与实参最佳匹配的函数，并生成调用该函数的代码
- 找不到任何一个函数与调用的实参匹配，此时编译器发出**无匹配（no match）**的错误信息
- 有多于一个函数可以匹配，但是每一个都不是明显的最佳选择。此时也将发生错误，称为二义性调用。









### 函数指针

函数指针指向的是函数而非对象。和其他指针一样，函数指针指向某种特定类型。函数的类型由它的返回类型和形参类型共同决定，**与函数名无关**。

```c++
// pf 指向一个函数，该函数的参数是两个 const string & ，返回值是 bool
bool (*pf)(const string&, const string&);
bool (*func(int, int))[10]; //返回 一个指向 包含十个 bool 的数组 的指针
bool *func(int, int)[10]; //不合法声明
```



使用函数指针：

```c++
pf = lengthCompare; // pf 指向名为 lengthCompare 的函数
pf = &lengthCompare; // 等价声明，取地址符是可选的

bool b1 = pf("hello", "goodbye"); //调用 lengthCompare 函数
bool b2 = (*pf)("hello", "goodbye"); //一个等价的调用
bool b3 = lengthCompare("hello", "goodbye"); //另一个等价的调用
```





<p style="border-bottom: 1px solid black;">函数指针形参</p>

数组不能作为函数的形参，但指针可以。同样的，虽然不能定义函数类型的形参，但是形参可以是指向函数的指针。此时，形参看起来是函数类型，实际上却是当成指针使用。

```c++
//第三个形参是函数类型，它会自动地转换成指向函数的指针
void useBigger(const string &s1, const string &s2,
              bool pf(const string &, const string &));
//等价的声明：显式地将形参定义成指向函数的指针
void useBigger(const string &s1, const string &s2,
              bool (*pf)(const string &, const string &));

useBigger(s1, s2, lengthCompare); //自动将函数转换成指向该函数的指针
```





<p style="border-bottom: 1px solid black;">返回指向函数的指针</p>

和数组类似，虽然不能返回一个函数，但是能返回指向函数类型的指针。

```c++
int (*f1(int)) (int *, int);
```

- `f1` 有形参列表，所以 `f1` 是个函数
- `f1` 前有 `*` ，所以 `f1` 返回一个指针
- 指针的指向的类型为 `int (int *, int);` ，因此指针指向函数，该函数的返回类型是 `int`



使用类型别名或尾置返回类型

```c++
using F = int(int *, int); // F 是函数类型，不是指针
using PF = int(*)(int *, int); // PF 是指针类型

PF f1(int); //正确，PF 是指向函数的指针，f1 返回指向函数的指针
F f1(int);  //错误：F 是函数类型，f1 不能返回一个函数
F *f1(int); //正确：显式地指定返回类型是指向函数的指针

auto f1(int) -> int (*)(int *, int);
```





是不是头晕了，来演化一下：

```c++
char f(int *, int); // f 是函数，返回 char
char *f(int *, int); // f 是函数，返回 指针，指针指向 char
char (*f(int *, int))[10]; // f 是函数，返回 指针，指向 char [10]
char (*f)(int *, int); // f 是指针，指向 char (int *, int)
char (*f(int))(int *, int); // f 是函数，返回 指针，指向 char (int *, int)

// signal 是函数，返回 指针，指向 void (int)
//func 是指针，指向 void (int)
void (*signal(int signo, void (*func)(int)))(int);
```

所以好像就是先区分是函数还是指针，

- 是函数，则再分析返回类型
- 是指针，则分析指针指向什么类型









## chapter_07 类

### 构造函数

每个类都分别定义了它的对象被初始化的方式，类通过一个或几个特殊的成员函数来控制其对象的初始化过程，这些函数叫做**构造函数（constructor）**。

- 构造函数不能被声明成 const 的。当我们创建类的一个 const 对象时，直到构造函数完成初始化过程，对象才能真正取得其“常量”属性。



<p style="border-bottom: 1px solid black;"><b>合成的默认构造函数</b></p>

编译器创建的构造函数又被称为**合成的默认构造函数（synthesized default constructor）**。

对大多数类来说，这个合成的默认构造函数将按照如下规则初始化类的数据成员：

- 如果存在类内的初始值，用它来初始化成员
- 否则，默认初始化该成员



某些类不能依赖于合成的默认构造函数，原因有三：

- 当类声明了构造函数时，编译器不会生成合成的默认构造函数
- 对于某些类来说，合成的默认构造函数可能执行错误的操作。比如，复合类型或定义在块中的内置类型的对象被默认初始化，则它们的值是未定义的
- 有时编译器不能为某些类合成默认的构造函数。比如类中包含一个其他类类型的成员且这个成员没有默认构造函数。 





<p style="border-bottom: 1px solid black;">构造函数初始值列表</p>

**随着构造函数体一开始执行，初始化就完成了。**这揭示了两方面：

- 构造函数初始化成员的工作在**初始值列表**中完成
- 如果构造函数没有初始值列表，则成员将在构造函数体之前执行默认初始化。函数体中进行的是赋值操作。

这也很好地解释了，为啥**引用和常量类型必须用初始值列表初始化**。





<p style="border-bottom: 1px solid black;">默认构造函数的作用</p>

当对象被默认初始化或值初始化时自动执行默认构造函数。

- 默认初始化在以下情况下发生：
    - 在块作用域内不使用任何初始值定义一个非静态变量或数组
    - 当一个类本身含有类类型的成员且使用合成的默认构造函数时
    - 当类类型的成员没有在构造函数初始值列表中显式地初始化时
- 值初始化在以下情况下发生：
    - 在数组初始化的过程中，提供的初始值数量少于数组的大小
    - 当我们不使用初始值定义一个局部静态变量时
    - 当我们通过书写形如 `T()` 的表达式显示地请求值初始化时，其中 T 是类型名

chatgpt 如此说道：

```text
默认初始化和值初始化之间的主要区别在于值初始化明确指定了要将变量初始化为某个特定值（0或默认构造函数的结果），而默认初始化没有明确指定初始值，因此留下未定义的值。
```





<p style="border-bottom: 1px solid black;"><b>委托构造函数</b></p>

一个**委托构造函数（delegating constructor）**使用它所属的其他构造函数执行它自己的初始化过程，或者说它把它自己的一些（或全部）职责委托给了其他构造函数。

```c++
class Sales_data
{
    public:
    	Sales_data(std::string s, unsigned cnt, double price):
    		bookNo(s), units_sold(cnt), revenue(cnt * price) { }
    
    	Sales_data() : Sales_data("", 0, 0) { } //默认构造函数
    	Sales_data(std::string s): Sales_data(s, 0, 0) { }
    	Sales_data(std::istream &is): Sales_data() { read(is, *this); }
};
```

执行顺序：

1. 先执行被委托的构造函数的初始值列表和函数体
2. 随后将控制权交还给委托者的函数体





<p style="border-bottom: 1px solid black;"><b>转换构造函数</b></p>

如果构造函数只接受一个实参，则它实际上定义了转换为此类类型的**隐式**转换机制，有时我们把这种构造函数称作**转换构造函数（converting constructor）**。

注意事项：

- 转换构造函数通常只有一个参数，且参数类型是源类型的对象。
- 只允许一步类类型转换

```c++
string null_book = "9-999-99999-9";
//构造了一个临时的 Sales_data 对象
//该对象的 units_sold 和 revenue = 0 ，bookNo = null_book
item.combine(null_book);

//错误，只允许一步类类型转换
// "9-999-99999-9" 会先转换为 string
item.combine("9-999-99999-9");

//正确：显式地转换成 string ，隐式地转换成 Sales_data
item.combine(string("9-999-99999-9"));
//正确：隐式地转换成 string ，显示地转换成 Sales_data
item.combine(Sales_data("9-999-99999-9"));

//会调用拷贝构造函数，然后可能在实参与形参匹配时
//实参 隐式转换成了 类类型
Sales_data item = null_book;
```



<p style="border-bottom: 1px solid black;">explicit</p>

通常用于修饰单参数构造函数（包括转换构造函数），以防止隐式类型转换发生。当你在一个构造函数前面加上 `explicit` 关键字时，它告诉编译器不要自动执行隐式类型转换，只有显式的构造函数调用才会被允许。





### 字面值常量类

累了，毁灭吧





### 类的静态成员

略略略







## chapter_08 IO 库

溜溜溜







## chapter_09 顺序容器

<table>
    <caption>顺序容器类型</caption>
    <tr>
        <td>vector</td>
        <td>可变大小数组。支持快速随机访问。在尾部之外的位置插入或删除元素可能很慢</td>
    </tr>
    <tr>
        <td>deque</td>
        <td>双端队列。支持快速随机访问。在头尾位置插入/删除速度很快</td>
    </tr>
    <tr>
        <td>list</td>
        <td>双向链表。支持双向顺序访问。在 list 中任何位置进行插入/删除操作速度都很快</td>
    </tr>
    <tr>
        <td>forward_list</td>
        <td>单向链表。只支持单向顺序访问。在链表任何位置进行插入/删除操作速度都很快</td>
    </tr>
    <tr>
        <td>array</td>
        <td>固定大小数组。支持快速随机访问。不能添加或删除元素</td>
    </tr>
    <tr>
        <td>string</td>
        <td>与 vector 相似的容器，但专门用于保存字符。随机访问快。在尾部插入/删除速度快</td>
    </tr>
</table>





<table>
    <caption>顺序容器操作</caption>
    <tr>
        <td>c.insert(p, t)</td>
        <td>在迭代器 p 指向的元素之前创建一个值为 t 的元素。返回指向新添加的元素的迭代器</td>
    </tr>
    <tr>
        <td>c.erase(p)</td>
        <td>删除迭代器 p 所指定的元素，返回一个指向被删除元素之后元素的迭代器。若 p 是尾后迭代器，则函数行为为定义</td>
    </tr>
    <tr>
        <td>c.clear()</td>
        <td>删除 c 中所有元素。返回 void </td>
    </tr>
</table>





<p style="border-bottom: 1px solid black;">容器操作可能使迭代器失效</p>

向容器中添加元素和从容器中删除元素的操作可能会使指向容器元素的指针、引用或迭代器失效。

- 在向容器添加元素后：
    - 如果容器 vector 或 string ，且存储空间被重新分配，则指向容器的迭代器、指针和引用都会失效。如果存储空间未重新分配，指向插入位置之前的元素的迭代器、指针和引用仍有效，但指向插入位置之后元素的迭代器、指针和引用将会失效。
    - 对于 deque ，插入到除收尾位置之外的任何位置都会导致迭代器、指针和引用失效。如果在首尾位置添加元素，迭代器会失效，但指向存在的元素的引用和指针不会失效。
    - 对于 list 和 forward_list ，指向容器的迭代器（包括尾后迭代器和首前迭代器）、指针和引用仍有效。
- 当我们删除一个元素后：
    - 对于 list 和 forward_list ，指向容器其他位置的迭代器（包括尾后迭代器和首前迭代器）、引用和指针仍有效
    - 对于 deque ，如果在首尾之外的任何位置删除元素，那么指向被删除元素外其他元素的迭代器、引用和指针也会失效。如果是删除 deque 的尾元素，则尾后迭代器也会失效，但其他迭代器、引用和指针不受影响；如果是删除首元素，这些也不会受影响。
    - 对于 vector 和 string ，指向被删元素之前元素的迭代器、引用和指针仍有效。



```c++
//删除偶数元素，复制每个奇数元素
vector<int> vi = {0, 1, 2, 3, 4, 5, 6, 7};
auto iter = vi.begin();
while(iter != vi.end()) //不要保存 vi.end() 哦
{
    if(*iter % 2) //奇数
    {
        iter = vi.insert(iter, *iter);
        iter += 2;
    }
    else iter = vi.erase(iter);
}
```







<p style="border-bottom: 1px solid black;">vector 对象是如何增长的</p>

倍增的思想。





<p style="border-bottom: 1px solid black;">容器适配器</p>







## chapter_10 泛型算法

### 概述

<p style="border-bottom: 1px solid black;">关键概念：算法永远不会执行容器的操作</p>

泛型算法本身不会执行容器的操作，它们只会运行于迭代器之上，执行迭代器的操作。

这带来的结果：

- 算法永远不会改变底层容器的大小
- 算法可能改变容器中保存的元素的值，也可能在容器内移动元素，但永远不会直接添加或删除元素



### 初识泛型算法

<p style="border-bottom: 1px solid black;">重排容器元素的算法</p>

目标：假设已有一个 vector ，保存了多个故事的文本。现需要简化这个 vector ，使得每个单词只出现一次。

```c++
void elimDups(vector<string> &words)
{
    sort(words.begin(), words.end());
    auto end_unique = unique(words.begin(), words.end());
    words.erase(end_unique, words.end());
}
```





### 定制操作

<p style="border-bottom: 1px solid black;">向算法传递函数</p>

目标：在调用 `elimDups` 之后，将单词按其长度升序排序，大小相同的再按字典序排序。

```c++
bool isShorter(const string &s1, const string &s2)
{
    return s1.size() < s2.size();
}

elimDups(words);
//std::stable_sort 保留了相等元素的相对顺序，即它是稳定的排序算法
//稳定排序的意思是，如果两个元素在排序前相等，那么它们在排序后的相对顺序将保持不变
stable_sort(words.begin(), words.end(), isShorter);

for(const auto &s : words)
    cout << s << " ";
cout << endl;
```



**谓词**是一个可调用的表达式，其返回结果是一个能用做条件的值。

标准库算法所使用的谓词分为两类：

- **一元谓词（unary predicate）**：只接受单一参数
- **二元谓词（binary predicate）**：意味着它们有两个参数

接受谓词参数的算法对输入序列中的元素调用谓词。因此，元素必须能转换成谓词的参数类型。





### lambda 表达式

根据算法接受一元谓词还是二元谓词，我们传递给算法的谓词必须严格接受一个或两个参数。

我们向一个算法传递任何类别的**可调用对象（callable object）**。对于一个对象或一个表达式，如果可以对其使用调用运算符，则称它为可调用的。可调用对象包括：

- 函数
- 函数指针
- 重载了函数调用运算符的类
- lambda 表达式



目标：在调用 `elimDups` 和 `stable_sort` 后，求大于等于一个给定长度的单词有多少。

```c++
string make_plural(size_t ctr, const string &word, const string &ending)
{
    return (ctr > 1) ? word + ending : word;
}

void biggies(vector<string> &words, vector<string>::size_type sz)
{
    //按字典序排序后消除重复出现单词
    elimDups(words);
    //按长度排序，长度相同按字典序排序
    stable_sort(words.begin(), words.end(), isShorter);
    //获取一个迭代器，指向第一个满足 size() >= sz 的元素
    auto wc = find_if(words.begin(), words.end(),
           [sz](const string &a)
            {return a.size() >= sz; });
    
    auto count = words.end() - wc;
    cout << count << " " << make_plural(count, "word", "s")
        << " of length " << sz << " or longer " << endl;
    //打印长度大于等于给定值的单词，每个单词后面接一个空格
    for_each(wc, words.end(),
             [](const string &s)
             {cout << s << " "; });
    
    cout << endl;
}
```





<p style="border-bottom: 1px solid black;">介绍 lambda</p>

一个 lambda 表达式具有如下形式

```c++
[capture list] (parameter list) -> return type { function body }
```

我们可以忽略参数列表和返回类型，但必须永远包含捕获列表和函数体

```c++
auto f = [] { return 42; };
cout << f() << endl; //使用调用运算符 调用 lambda
```



- 捕获列表只用于局部非 static 变量， lambda 可以直接使用局部 static 变量和在它所在函数之外声明的名字
- 当向一个函数传递一个 lambda 时，同时定义了一个新类型和该类型的一个对象：传递的参数就是此编译器生成的类类型的未命名对象。
- 默认情况下，从 lambda 生成的类都包含一个对应该 lambda 所捕获的变量的数据成员。类似任何普通类的数据成员，lambda 的数据成员也在 lambda 对象创建时被初始化。



<p style="border-bottom: 1px solid black;">值捕获</p>

- 与传值参数类似，采用值捕获的前提是变量可以拷贝。

- 与参数不同，被捕获的变量的值是在 lambda **创建时拷贝**，而不是调用时拷贝。



<p style="border-bottom: 1px solid black;">引用捕获</p>

- 当采用引用方式捕获一个变量时，必须确保被引用的对象在 lambda 执行的时候是存在的
- lambda 捕获的都是局部变量，这些变量在函数结束之后就不复存在了



<p style="border-bottom: 1px solid black;">隐式捕获</p>

在捕获列表中写一个 `&` 或  `=` ，可以告诉编译器采用捕获引用的方式还是值捕获的方式。编译器根据 lambda 体中的代码来推断我们要使用哪些变量。

混合使用隐式捕获和显式捕获时：

- 捕获列表中的第一个元素必须是一个 `&` 或  `=` 
- 显式捕获的变量必须使用与隐式捕获不同的方式，即
    - 如果隐式捕获是引用方式，则显式捕获必须使用值方式
    - 如果隐式捕获是值方式，则显式捕获必须使用引用方式





<p style="border-bottom: 1px solid black;">可变 lambda</p>

默认情况下，对于一个值被拷贝，lambda 不会改变其值。如果希望能改变一个被捕获的变量的值，就必须在参数列表首加上关键字 mutable

```c++
void func1()
{
    size_t v1 = 42;
    auto f = [v1] () mutable { return ++v1; };
    cout << f() << " " << v1 << endl; // 43 42
    v1 = 0;
    auto j = f();
    cout << j << " " << v1 << endl; // j = 43  v1 = 0
}

void func2()
{
    size_t v1 = 42;
    auto f = [&v1] { return ++v1; };
    cout << f() << " " << v1 << endl; // 43 42 （咋回事儿啊
    v1 = 0;
    auto j = f();
    cout << j << " " << v1 << endl; // j = 1  v1 = 1
}
```







### 参数绑定

举个例子，标准库算法 `find_if` 只接受一个一元谓词，因此只能用 lambda 而不能使用函数。但通过标准库 `bind` 函数，我们可以解决传递参数问题。

```c++
bool check_size(const string &s, string::size_type sz)
{
    return s.size() >= sz;
}

using std::placeholders::_1;
using namespace std::placeholder; // for _1 _2 _3...

auto wc = find_if(words.begin(), words.end(),
                 bind(check_size, _1, sz));
```





### 再探迭代器

介绍下几种迭代器：

- 插入迭代器（insert iterator）：这些迭代器被绑定到一个容器上，可用来向容器插入元素
- 流迭代器（stream iterator）：被绑定到输入或输出流上，可用来遍历所关联的流
- 反向迭代器（reverse iterator）：除了 `forawrd_list` 都有反向迭代器
- 移动迭代器（move iterator）：不是拷贝其中的元素，而是移动它们





<p style="border-bottom: 1px solid black;">插入迭代器</p>

插入器有三种类型，差异在于元素插入的位置：

- **back_inserter** 创建一个使用 push_back 的迭代器
- **front_inserter** 创建一个使用 push_front 的迭代器
- **inserter** 创建一个使用 insert 的迭代器。此函数接受第二个参数，元素将被插入到给定迭代器所表示的元素之前。



插入迭代器操作

```text
//根据插入插入迭代器的不同种类，可能会调用
// c.push_back(t)	c.push_front(t) 	c.insert(t, p)
it = t;

*it, ++it, it++ //这些操作虽然存在，但不会对 it 做任何事。每个操作都返回 it
    
// 加入 it 是 inserter 生成的迭代器
*it = val;
等价于
it = c.insert(it, val);
++it; //递增 it 使它指向原来的元素
```

```c++
vector<int> lst = {1, 2, 3, 4};
vector<int> lst2, lst3;

copy(lst.cbegin(), lst.cend(), inserter(lst2, lst2.begin()));

//lst2 竟然是 {1, 2, 3, 4} 无法理解
for_each(lst2.begin(), lst2.end(),
         [] (const int &x) { cout << x << " "; });
cout << endl;


copy(lst.cbegin(), lst.cend(), front_insert(lst3)); // 4 3 2 1
```







<p style="border-bottom: 1px solid black;">流迭代器</p>

- **istream_iterator**

    ```c++
    istream_iterator<int> in_iter(cin); //从 cin 读取 int
    istream_iterator<int> eof; //人为定义尾后迭代器
    while(in_iter != eof) vec.push_back(*in_iter++);
    
    //更优雅的等价写法
    istream_iterator<int> in_iter(cin), eof;
    vector<int> vec(in_iter, eof); //从迭代器范围构造 vec
    ```

- **ostream_iterator**

    ```c++
    ostream_iterator<int> out_iter(cout, " ");
    for(auto e : vec) *out_iter++ = e;
    cout << endl;
    
    //或者这样
    for(auto e : vec) out_iter = e;
    
    //又或者
    copy(vec.begin(), vec.end(), out_iter);
    ```

    



<p style="border-bottom: 1px solid black;">反向迭代器</p>

除了 `forawrd_list` 外，其他容器都支持反向迭代器。

```c++
vector<int> vec = {0, 1, 2, 3, 4};
for(auto r_iter = vec.crbegin(); r_iter != vec.crend(); ++r_iter)
    cout << *r_iter << endl;
```

```c++
string line("abc,def,xyz");
auto comma = find(line.cbegin(), line.cend(), ','); //comma = line + 3
cout << string(line.cbegin(), comma) << endl; //打印第一个单词

//打印最后一个单词
auto rcomma = find(line.crbegin(), line.crend(), ','); // rcomma = line + 7
cout << string(line.crbegin(), rcomma) << endl; //错啦，会打印 zyx [line + 10, line + 7)
cout << string(rcomma.base(), line.cend()) << endl; //正确写法 [line + 8, line + 11)
```





<p style="border-bottom: 1px solid black;">移动迭代器</p>

待补充







### 泛型算法结构

任何算法的最基本特性是它要求其迭代器提供哪些操作。某些算法，如 find ，只要求：

- 通过迭代器访问元素
- 递增迭代器
- 比较两个迭代器是否相等

这些能力。其他一些算法，如 sort ，还要求读、写和随机访问元素的能力。

算法所要求的迭代器操作可以分为 5 个迭代器类别：

- 输入迭代器：只读，不写；单独扫描，只能递增
- 输出迭代器：只写，不读；单独扫描，只能递增
- 前向迭代器：可读写，多遍扫描，只能递增
- 双向迭代器：可读写；多遍扫描，可递增递减
- 随机访问迭代器：可读写，多遍扫描，支持全部迭代器运算









## chapter_11 关联容器

标准库提供 8 个关联容器。这 8 个容器间的不同体现在三个维度上：每个容器

- 或者是一个 set ，或者是一个 map；
- 或者要求不重复的关键字，或者允许重复关键字（名字包含 multi）；
- 按顺序保存元素，或无序保存（以 unordered 开头）。



<table>
	<caption>关联容器类型</caption>
    <tr><th colspan=2>按关键字有序保存元素</th></tr>
    <tr><td>map</td> <td>关联数组：保存关键字-值对</td></tr>
    <tr><td>set</td> <td>关键字即值，即只保存关键字的容器</td></tr>
    <tr><td>multimap</td> <td>关键字可重复出现的 map</td></tr>
    <tr><td>multiset</td> <td>关键字可重复出现的 set</td></tr>
    <tr><th colspan=2>无序集合</th></tr>
    <tr><td>unordered_map</td> <td>哈希组织的 map</td></tr>
    <tr><td>unordered_set</td> <td>哈希组织的 set</td></tr>
    <tr><td>unordered_multimap</td> <td>哈希组织的 map ；关键字可重复出现</td></tr>
    <tr><td>unordered_multiset</td> <td>哈希组织的 set ；关键字可重复出现</td></tr>
</table>





<p style="border-bottom: 1px solid black;">set</p>

一直以来，我好像对 set 有误解，以为 set 也有值，可以用下标索引。

但实际上，set 只是保存关键字而已。set 是非常特别的，我们拿 vector 和 map 做对比，可以简单地得出一个结论，vector 的下标只能是数字，而 map 则多种多样。**但 set 没有下标一说**。

```c++
set<string> sset = {"abc", "def"};
for(auto it = sset.begin(); it != sset.end(); ++it)
{
    cout << *it << " ";
    cout << sset[*it] << " "; //错误：set 没有下标索引
}
```





<p style="border-bottom: 1px solid black;">map</p>

没啥好写的，代码镇楼

```c++
#include <iostream>
#include <string.h>
#include <unistd.h>
#include <algorithm>
#include <unordered_map>
#include <fstream>
#include <assert.h>
#include <sstream>


using namespace std;

int main(int argc, char *argv[])
{
    if(argc != 3)
    {
        printf("Usage: %s <file_name1> <flie_name2>\n", argv[0]);
        exit(1);
    }

    ifstream file1(argv[1]);
    ifstream file2(argv[2]);
    assert(file1.is_open() && file2.is_open());

    unordered_map<string, string> map1;
    string line;
    while(getline(file1, line))
    {
        istringstream ss(line);
        string key, val;
        if(!(ss >> key >> val))
        {
            cerr << "Failed to read two strings from line" << line << endl;
            continue;
        }
        map1.insert({key, val});
    }

    FILE *file3 = fopen("txt/data.txt", "w");
    if(file3 == NULL)
    {
        perror("fopen");
        exit(1);
    }
    while(getline(file2, line))
    {
        istringstream iss(line);
        ostringstream oss;
        string word;
        while(iss >> word)
        {
            //cout << (map1.count(word) ? map1[word] : word) << " ";
            word = map1.count(word) ? map1[word] : word;
            cout << word << " ";
            oss << word << " ";
        }
        fputs((oss.str() + "\n").c_str(), file3);
        cout << endl;
    }

    file1.close();
    file2.close();
    fclose(file3);

    return 0;
}
```





<p style="border-bottom: 1px solid black;">无序容器</p>

我们不能直接定义关键字类型为自定义类类型的无需容器。

```c++
size_t hasher(const Sales_data &sd)
{
    return hash<string>() (sd.isbn());
}

bool eqOp(const Sales_data &lhs, const Sales_data &rhs)
{
    return lhs.isbn() == rhs.isbn();
}

using SD_multiset = unordered_multiset<Sales_data, decltype(hasher *),
										decltype<eqOp> *>;
//参数：桶大小 哈希函数指针 相等性判断运算符指针
SD_multiset bookstore(42, hasher, eqOp);
```











## chapter_12动态内存

为了更好地管理动态内存，新的标准库提供了两种**智能指针（smart pointer）**，负责自动释放所指向的对象。

- **shared_ptr** ：允许多个指针指向同一个对象
- **unique_ptr** ：”独占“所指向的对象
- **weak_ptr** ：弱引用，指向 shared_ptr 所管理的对象



### shared_ptr 类

类似 vector ，智能指针也是模板。

```c++
shared_ptr<string> p1;
shared_ptr<list<int>> p2;
```

**默认初始化的智能指针保存着一个空指针。**



shared_ptr 和 unique_ptr 都支持的操作

- shared_ptr<T> sp   unique_ptr<T> up ：空智能指针，可以指向类型为 T 的对象
- p ：将 p 用作一个条件判断，若 p 指向一个对象，则为 true
- *p ：解引用 p ，获得它所指向的对象
- p->mem ：等价于 `(*p).mem`
- p.get() ：返回 p 中保存的指针
- swap(p, q)   p.swap(q) ：交换 p 和 q 的指针



shared_ptr 独有的操作：

- make_shared<T>(args) ：返回一个 shared_ptr ，指向一个动态分配的类型为 T 的对象。
- shared_ptr<T>p(q) ：p 是 shared_ptr 的拷贝；此操作会递增 q 中 的计数器。
- p = q ： p 和 q 都是 shared_ptr ，所保存的指针必须能互相转换。此操作会递减 p 的引用计数，递增 q 的引用计数
- p.unique() ：若 `p.use_count()` 为 1 ，返回 `true` ；否则返回 `false`
- p.ues_count() ：返回与 p 共享对象的智能指针数量；可能很慢，主要用于调试







<p style="border-bottom: 1px solid black;">make_shared 函数</p>

最安全的分配和使用动态内存的方法是调用一个名为 make_shared 的标准库函数。

**此函数在动态内存中分配一个对象并初始化它，返回指向此对象的 share_ptr**。

```c++
//指向一个值为 42 的 int 的 shared_ptr （？啥玩意）
shared_ptr<int> p3 = make_shared<int>(42);
// 指向一个值为 "999999999" 的 string
shared_ptr<string> p4 = make_shared<string>(10, '9');
auto p5 = make_shared<int>(); // p5 指向一个值初始化的 int ，即，值为 0
```





程序使用动态内存处于以下三种原因之一：

- 程序不知道自己需要使用多少对象（容器类）
- 程序不知道所需对象的准确类型（模板类）
- 程序需要在多个对象间共享数据





<p style="border-bottom: 1px solid black;">直接管理内存</p>

```c++
string *ps1 = new string; //默认初始化为空 string
string *ps = new string(); //值初始化为空 string

int *pi1 = new int; //默认初始化，*pi1 的值未定义
int *pi2 = new int(); //值初始化为 0 , *pi2 为 0
```

对于定义了自己的构造函数的类类型来说，要求值初始化是没有意义的。不管采用什么形式，对象都会通过默认构造函数来初始化。

但对于内置类型，两种形式的差别就大了。值初始化的内置类型对象有着良好定义的值，而默认初始化的对象的值则是未定义的。

（懂了吗？懂个屁。初始化的坑能直接把我埋了）



<p style="border-bottom: 1px solid black;">shared_ptr 和 new 结合使用</p>

一句话，不建议混合使用







智能指针陷阱

为了正确使用智能指针，我们必须坚持一些基本规范：

- 不使用相同的内置指针值初始化（或 reset ）多个智能指针
- 不 delete get() 返回的指针
- 不使用 get() 初始化或 reset 另一个智能指针
- 如果你使用 get() 返回的指针，记住当最后一个对应的智能指针销毁后，你的指针就变为无效了
- 如果你使用智能指针管理的资源不是 new 分配的内存，记住传递给它一个删除器





### unique_ptr

一个 unique_ptr ”拥有“ 它所指的对象。

- 与 shared_ptr 不同，某个时刻只能有一个 unique_ptr 指向一个给定对象。
- 与 shared_ptr 不同，没有类似 make_shared 的标准库函数，必须将 shared_ptr 绑定到一个 new 返回的指针上。类似 shared_ptr ，初始化 unique_ptr 必须采用直接初始化形式
- 由于一个 unique_ptr 拥有它所指向的对象，因此 unique_ptr 不支持普通的拷贝或赋值操作
- 但是可以拷贝或赋值一个将要被销毁的 unique_ptr

```c++
uniqueprt<string> p1(new string("hello"));
unique_ptr<string> p2(p1); //错误：unique_ptr 不支持拷贝
unique_ptr<string> p3;
p3 = p2; //错误：unique_ptr 不支持赋值
```



unique_ptr 操作

- unique_ptr<T> u1     unique_ptr<T, D> u2：空 unique_ptr ，可以指向类型为 T 的对象。u1 会使用 delete 来释放它的指针；u2 会使用一个类型为 D 的可调用对象来释放它的指针
- unique_ptr<T, D> u(d) ：空 unique_ptr ，指向类型为 T 的对象，用类型为 D 的对象 d 代替 delete
- u = nullptr ：释放 u 指向的对象，将 u 置为空
- u.release() ：u 放弃对指针的控制权，**返回指针**，并将 u 置为空
- u.reset() ：释放 u 指向的对象（这才是释放内存操作）
- u.reset(q) ：如果提供了内置指针 q ，令 u 指向这个对象；否则置为空
- u.reset(nullptr)

```c++
unique_ptr<string> p1(new string("hello"));
unique_ptr<string> p2(p1.release()); //将所有权从 p1 转移给 p2

unique_ptr<string> p3(new string("world"));
p3.reset(p2.release()); //释放 p3 所指对象，将 p2 所指对象转移给 p3
```





### weak_ptr

weak_ptr 是一种不控制所指向对象生存期的智能指针，它指向由一个 shared_ptr 管理的对象。

weak_ptr 的操作

- weak_ptr<T> w ：
- weak_ptr<T> w(sp) ：T 必须能转换为 sp 指向的类型
- w = p ：赋值后 w 与 p 共享对象
- w.reset() ：将 w 置为空
- w.use_count() ：与 w 共享对象的 shared_ptr 的数量
- w.expired() ：若 w.use_count() 为 0 ，返回 true ，否则返回 false
- w.lock() ：如果 expired 为 true ，返回一个空 shared_ptr ；否则返回 一个指向 w 的对象的 shared_ptr

```c++
auto sp = make_shared<int>(42);
weak_ptr<int> wp(sp);

if(shared_ptr<int> np = wp.lock()) // np 不为空则条件成立
{
}
```









## chapter_13 拷贝控制

当定义一个类时，我嫩显式地或隐式地指定在此类型地对象拷贝、移动、赋值和销毁时做什么。

一个类通过定义五种特殊地成员函数来控制这些操作，我们称这些操作为**拷贝控制操作（copy control）**。

- 拷贝构造函数（copy constructor）
- 拷贝赋值运算符（copy-assignment operator）
- 移动构造函数（move constructor）
- 移动赋值运算符（move-assignment operator）
- 析构函数（destructor）





- 拷贝构造函数地第一个参数必须是一个引用类型（否则实参匹配时会递归调用自己）

- 拷贝构造函数在几种情况下都会被隐式地使用，因此不应该是 `explicit` 的

- 编译器总是会生成 合成拷贝构造函数

- 编译器可能有时将拷贝初始化解释成直接初始化（无所谓了）

- 构造函数中，成员的初始化在函数体执行之前完成，然后执行函数体

- 析构函数中，首先执行函数体，然后按初始化顺序的逆序销毁

    - 说明函数体并不执行销毁操作，那么可以认为 析构函数是作为成员销毁步骤之外的另一部分而进行的

    



**什么时候会调用析构函数**

无论何时，一个对象被销毁，就会自动调用其析构函数：

- 变量在离开其作用域时被销毁
- 当一个对象被销毁时，其成员被销毁
- 容器被销毁时，其元素被销毁
- 对于动态分配的对象，当对指向它的指针应用 delete 运算符时被销毁
- 对于临时对象，当创建它的完整表达式结束时被销毁





### 三/五法则

- 需要析构函数的类也需要拷贝和赋值操作

    > 当我们决定一个类是否要定义它自己版本的拷贝控制成员时，一个基本原则是首先确定这个类是否需要一个析构函数。如果一个类需要一个析构函数，我们几乎可以肯定它也需要一个拷贝构造函数和一个拷贝赋值运算符。

      ```c++
      class HasPtr {
          public:
          	HasPtr(const string &s = string()): ps(new string(s)), i(0)
              { }
          	~HasPtr() { delete ps; }
          private:
      		string *ps;
          	int i;
      };
      
      HasPtr f(HasPtr hp) //调用合成拷贝构造函数，此时有两个指针指向同一内存
      {
          HasPtr ret = hp; //此时有三个指针指向同一内存了
          return ret;
      } //函数返回后，ret 和 hp 被销毁，其中的指针成员被 delete 两次，操作未定义
      
      //所以要咋写呢，自己写个拷贝构造函数和拷贝赋值运算符来避免浅拷贝
      ```

- 需要拷贝操作的类也需要赋值操作，反之亦然



### 对象移动

<p style="border-bottom: 1px solid black;"> 右值引用 </p>

所谓的右值引用就是必须绑定到右值的引用。我们通过 `&&` 来获得右值引用，右值引用有一个重要的性质——**只能绑定到一个将要销毁的对象**。

```c++
int i = 42;

int &r = i; //正确
int &r2 = 42; //错误：不能将一个左值引用绑定到右值上
const int &r3 = 42; //正确：我们可以将一个 const 的引用绑定到右值上

int &&rr = i; //错误：不能将一个右值引用绑定到一个左值上
int &&rr2 = 42; //正确：将 rr2 绑定到惩罚结果上
```

返回左值表达式的例子：

- 返回左值引用的函数
- 赋值、下标、解引用和前置递增/递减运算符

返回右值的例子：

- 返回非引用类型的函数
- 算数、关系、位、后置递增/递减运算符



变量表达式都是左值（变量表达式就是指变量），这带来一个结果：

```c++
int &&rr1 = 42; //正确
int &&rr2 = rr1; //错误：表达式 rr1 是左值
```



虽然不能将一个右值引用直接绑定到一个左值上，但我们可以显式地将一个左值转换为对应的右值引用类型。

**move** 函数接受一个左值并返回一个对应的右值引用

```c++
int &&rr3 = std::move(rr1); //正确
```

- 与大多数标准库名字不同，我们直接调用 `std::move` 而不是 `move`
- 调用 `move` 意味着承若：除了对 `rr1` 赋值或销毁它外，我们将不再使用它





### 移动构造函数和移动赋值运算符

<p style="border-bottom: 1px solid black;"> 移动构造函数 </p>

- 类似拷贝构造函数，移动构造函数的第一个参数是该类类型的一个引用
- 不同于拷贝构造函数，这个引用参数是右值引用
- 与拷贝构造函数一样，任何额外的参数都必须有默认实参

```c++
StrVec::StrVec(StrVec &&s) noexcept : elements(s.elements),
	first_free(s.first_free), cap(s.cap)
{
    //令 s 进入这样的状态，对其运行析构函数是安全的
	s.elements = s.first_free = s.cap = nullptr;
}
```

- 移动构造函数不分配任何新内存，只接管给定的 `StrVec` 中的内存
- 在接管内存后，将给定对象中的指针都置为 `nullptr` 





<p style="border-bottom: 1px solid black;"> 移动赋值运算符 </p>

```c++
StrVec& StrVec::operator=(StrVec &&rhs) noexcept
{
    if(this == &rhs) return *this;

    free();
    elements = rhs.elements;
    first_free = rhs.first_free;
    cap = rhs.cap;

    rhs.elements = rhs.first_free = rhs.cap = nullptr;

    return *this;
}
```







**后面的就看不太懂了，嘻嘻**







## chapter_14 重载运算与类型转换

### 基本概念

- 不能被重载的四个运算符：`::`	` .* `	`.` 	`? :`

- 不建议被重载的运算符：
    - 有三个运算符指定了运算对象的求值顺序：
        - 逗号运算符（`,`）：依次从左到右求值，并返回最右边表达式的结果
        - 逻辑与和（`&&`）逻辑或（`||`）运算符：具有短路求值的特性
        - 三元条件运算符（`? :`）：根据表达式结果，只有一个分支会被求值
    - 使用重载的运算符本质上是一次函数调用，所以求值顺序的规则会失效。所以不建议重载以上三个运算符（虽然其中 条件运算符本来就不能被重载）
    - 另外取地址运算符（`&`）也不建议被重载
- 重载运算符作为普通函数还是成员函数：
    - 赋值、下标、调用和成员访问箭头运算符必须是成员
    - 复合赋值运算符一般来说应该是成员
    - 改变对象状态的运算符或者与给定类型密切相关的运算符，如递增、递减和解引用，应该是成员
    - 具有兑成性的运算符可能转换任意一端的运算对象，例如算数、相等性、关系和位运算符等，因此它们应该是普通的非成员函数
    - （自己补充一点），输入输出运算符必须是非成员函数



### 输入输出运算符

- 因为 IO 运算符通常需要读写类的非公有数据成员，所以 IO 运算符一般被声明为友元
- 通常情况下，第一个形参是流的非常量引用（因为流的状态会被改变并且流不允许被拷贝）
- 输入运算符必须处理输入可能失败的情况，而输出运算符不需要
    - 当流含有错误类型的数据时读取操作可能失败（格式错误）
    - 当读取操作到达文件末尾
    - 用户终止输入

```c++
ostream &operator<<(ostream &os, const Sales_data &item)
{
    os << item.isbn() << " " << item.units_sold << " " <<
        item.revenue << " " << item.agv_price();
    return os;
}

istrema &operator>>(istream &is, Sales_data &item)
{
    double price;
    is >> item.bookNo >> item.units_sold >> price;
    
    if(is) //检查输入是否成功
        item.revenue = item.units_sold * price;
    else item = Sales_data(); //输入失败，对象被赋予默认的状态
    
    return is;
}
```















## chapter_15 面向对象程序设计

### 动态绑定

当我们使用基类的引用或指针调用一个**虚函数**时将发生**动态绑定（dynamic binding）**。

多态性分为两种：

- 编译时多态性（Compile-time Polymorphism）也称为静态多态性：
    - 一个常见的例子是函数重载
- 运行时多态性（Runtime Polymorphism）也称为动态多态性：
    - 主要通过虚函数和继承来实现。当基类的指针或引用指向派生类的对象时，可以通过基类的指针或引用来调用虚函数，实际上会根据对象的实际类型来决定调用哪个函数

给运行时多态性举个例子

```c++
class Base
{
    public:
    	virtual void f1() { cout << "Base" << endl; }
    	virtual void f2() { cout << "Base" << endl; }
};

class Drived : public Base
{
    public:
    	virtual void f1() { cout << "Drived" << endl; }
    	virtual void ff2() { cout << "Drived" << endl; }
};

int main()
{
    Base b, *pb1, *pb2;
    Drived d;
    
    pb1 = &b; //基类指针指向基类
    pb2 = &d; //基类指针指向派生类
    
    pb1->f1(); //Base
    //pb1->Drived::f1(); //编译失败
    pb2->f1(); //Drived
    pb2->Base::f1(); //Base
    
    cout << "********" << endl;
    
    pb1->f2(); //Base
    pb2->f2(); //Base
    cout << "********" << endl;

    //以下 打咩
    // pb1->ff2();
    // pb2->ff2();
    // pb1->Drived::ff2();
    // pb2->Drived::ff2();
    
    return 0;
}
```

所以简单总结一下：

- 基类指针指向基类：没得说，调用基类函数
- 基类指针指向派生类：
    - 由于是基类指针，所以只能调用基类有的虚函数
    - 根据虚函数是否被派生类覆盖来决定调用基类还是派生类。（这不就体现运行时多态了吗）

回顾 虚函数的初衷，就是为了在 基类指针指向派生类时，能够调用派生类覆盖的函数。



<p style="border-bottom: 1px solid black;"> 虚函数 </p>

- 虚函数必须在其内部对所有**重新定义**的虚函数**进行声明**。
- 如果派生类要重新定义虚函数，最好加上关键字 `override` （因为如果参数不一样就会失去 `virtual` 属性）
- 虚函数与非虚函数不太一样，如果要重新定义虚函数，必须保证 返回类型 和 形参列表 都一致
- 当然，有一个例外，当返回类型是类本身的指针或引用时，返回类型可以不一样。





<p style="border-bottom: 1px solid black;"> 类型转换 </p>

- 通常情况下，如果我们想把引用或指针绑定到一个对象上，则引用或指针的类型应与对象的类型一致，或者对象的类型含有一个可接受的 `const` 类型转换规则。不过之前指出有一个例外，现在它来了：**可以将基类的指针或引用绑定到派生类对象上**。

- 对象之间不存在类型转换，但是可以用派生类对象初始化或赋值基类对象。这是因为拷贝构造函数和拷贝赋值运算符的参数都是引用。

    ```c++
    Drived d;
    Base b(d);
    b = d;
    ```

    





<p style="border-bottom: 1px solid black;"> 覆盖与隐藏 </p>

- 覆盖：
    - 覆盖是指在派生类中重新定义基类中的虚函数
    - 覆盖要求派生类的函数与基类中的虚函数在函数签名（包括函数名、参数列表和返回类型）上完全匹配
    - 如果返回类型不同，形参相同，那么编译错误
    - 如果形参不同，那么失去虚函数特性，并且隐藏了基类的虚函数
- 隐藏：
    - 隐藏是指在派生类中定义了与基类中的非虚函数具有相同名称的函数
    - 这指出，无论返回类型和形参列表，只要名称相同，都算是隐藏

所以，至此，我想应该算是弄懂了覆盖与隐藏的区别：

- 覆盖针对虚函数而言，隐藏针对非虚函数而言
- 覆盖如果形参不同，则会失去虚函数特性，如果形参相同，返回类型不同，则会编译错误
- 隐藏无视返回类型和形参，只要名称相同，都算隐藏



另外，覆盖与隐藏都相对于不同作用域（比如基类与派生类），而重载是相对于同一作用域。



<p style="border-bottom: 1px solid black;"> 基类与派生类的作用域 </p>

派生类的作用域嵌套在其基类的作用域。

如果一个名字在派生类的作用域内无法正确解析，则编译器将继续在外层的基类作用域中寻找该名字的定义。

```c++
Bulk_quote bulk;
cout << bulk.isbn();
```

名字 isbn 的解析将按照下述过程所示：

- 先在 bulk_quote 中查找，这一步没有找到名字
- bulk_quote 继承自 Disc_quote ，所以接下来在 Disc_quote 中查找，没找到
- Disc_quote 继承自 Quote ，所以接着查找 Quote ；此时找到了名字 isbn 

所以我们使用的 isbn 最终被解析为 Quote 中的 isbn 。



我想这揭示了两个方面：

- 派生类虽然会继承基类的成员，但终究不是自己的，与自己的还是有许多差别
- 如果派生类重新定义了基类中的成员，那么基类成员会被隐藏。也就是说，当你定义一个派生类对象，会分别为派生类和基类的成员变量分配空间。





### 拷贝控制与继承

```c++
#include <iostream>

class Base {
public:
    Base() { std::cout << "Base Constructor" << std::endl; }
    Base(const Base& other) { std::cout << "Base Copy Constructor" << std::endl; }
    Base& operator=(const Base& other) { 
        std::cout << "Base Copy Assignment Operator" << std::endl; 
        return *this; 
    }
    Base(Base&& other) noexcept { 
        std::cout << "Base Move Constructor" << std::endl; 
    }
    Base& operator=(Base&& other) noexcept { 
        std::cout << "Base Move Assignment Operator" << std::endl; 
        return *this; 
    }
    virtual ~Base() { std::cout << "Base Destructor" << std::endl; }
};

class Derived : public Base {
public:
    Derived() { 
        std::cout << "Derived Constructor" << std::endl; 
    }
    Derived(const Derived& other) : Base(other) { 
        std::cout << "Derived Copy Constructor" << std::endl; 
    }
    Derived& operator=(const Derived& other) {
        Base::operator=(other);
        std::cout << "Derived Copy Assignment Operator" << std::endl;
        return *this;
    }
    Derived(Derived&& other) noexcept : Base(std::move(other)) { 
        std::cout << "Derived Move Constructor" << std::endl; 
    }
    Derived& operator=(Derived&& other) noexcept {
        Base::operator=(std::move(other));
        std::cout << "Derived Move Assignment Operator" << std::endl;
        return *this;
    }
    ~Derived() { 
        std::cout << "Derived Destructor" << std::endl; 
    }
};
```

