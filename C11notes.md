# 第一章
历史、简介、总结，可以在看完整本书之后回头看

# 第二章
## 2.1 保持与C99兼容
1. C99预定义宏
2. __func__预定义标识符
3. _Pragma操作符
4. 不定参数宏定义以及__VA_ARGVS__
5. 宽窄字符串链接
### 2.1.1 预定义宏
一些可以检查机器环境对C标准和C库的支持情况的宏，C++11也支持。
这些宏通常用于多目标平台的代码，通过预处理指令犹如
~~~c
#ifdef 
//..
#endif
~~~
来实现支持多平台。

### 2.1.2 __func__预定义宏
返回函数名字的宏，**不能将__func__作为函数参数的默认值，因为函数声明时，__func__还没被定义**

### 2.1.3 _Pragma操作符
和预处理指令#pragma功能相同，但是_Pragma是一个操作符，使用方跟sizeof等操作符一样
~~~c
#pragma once
//效果相同
_Pragma ("once")
~~~
主要优势是可以在宏中使用，提高代码的可读性和可维护性，因为可以在一个地方定义所有的pragma 指令，并在需要时调用它们。

~~~c
#define ENABLE_LOOP_UNROLLING

#ifdef ENABLE_LOOP_UNROLLING
#define LOOP_UNROLL _Pragma("GCC unroll 4")
#else
#define LOOP_UNROLL
#endif

void loopFunction() {
    LOOP_UNROLL
    for(int i = 0; i < 100; i++) {
        // Loop body intended for unrolling
    }
}
~~~

使用 #pragma 实现同样的功能远不如使用 _Pragma 的方法在项目中灵活和可重用

~~~c
#define ENABLE_LOOP_UNROLLING

void loopFunction() {
    #ifdef ENABLE_LOOP_UNROLLING
    #pragma GCC unroll 4
    #endif
    for(int i = 0; i < 100; i++) {
        // 循环体，意图进行展开
    }
}
~~~

### 2.1.4 变长参数的宏定义以及__VA_ARGS__
使用方法
~~~c
#include  <stdio.h>
#define LOG(...) \{\
    fprintf(stderr, "%s: Line %d:\t", __FILE__, __LINE__);\
    fprintf(stderr, __VA_AGRVS__);\
    fprintf(stderr, "\n");\
}
~~~

### 2.1.5 宽窄字符串的链接
支持了char和wchar_t的转换，std::wstring_convert 和 std::codecvt，但c++17以后已被弃用

## 2.2 longlong整型
可以通过查看<climits>中的宏LLONG_MIN,LLONG_MAX和ULLONG_MAX，查看平台支持的long long大小。

## 2.3 扩展的整型
C++11中只定义了5种标准的有符号整型
1. signed char
2. short int
3. int
4. long int 
5. long long int

因为这5种适用性有限，编译器会自行扩展一些整型。C+11规定的是扩展的整型必须和标准类型一样，有符号类型和无符号类型占用同样大小的内存空间。整型间的隐式转换被称为**整型的提升**，提升遵循以下原则从低等级整型到高等级整型。
1. 长度越大等级越高。
2. 同等长度，标准整型高于扩展类型。
3. 相同大小，有符号高于无符号。

## 2.4 宏__cplusplus
宏__cplusplus被预定义为201103L

## 2.5 静态断言
### 2.5.1 断言：运行时与预处理时
~~~c
assert(expr)
~~~
运行时断言，exper，假如不为真则立刻退出程序。\#define和\#error配合使用可以实现预处理断言
~~~c
#ifndef condition
#error "mess"
#endif
~~~
如果 condition为假，就会在编译的时候引发错误并输出mess

### 2.5.2 静态断言和static_assert
assert只在运行时有效，\#error只有在编译器预处理时才起作用。C++11引入static_assert，第一个参数必须是在编译时期可以计算的表达式，即常量表达式。
~~~c
template <typename t, typename u>
int bit_cont(t & a, u & b)
{
    static_assert(sizeof(a) == sizeof(b), "the parameters of bit_copy myust have same width.");
};
~~~
会在编译时进行检查。

## 2.6 noexcept修饰符与noexcept操作符
~~~c
void func() noexcept;
void func() noexcept(常量表达式);
~~~
表达式结果会被转换为bool值，true则表示函数不会抛出异常。不带表达式则默认为true。声明为noexcept的函数抛出异常会直接调用std::terminate终结程序的执行。

个人看法：
1. 默认未声明的noexcept的函数都会抛出异常；
2. 在 C++ 中，默认情况下，如果未捕获异常且没有调用 std::terminate() 强制终止程序，异常会在调用栈中不断传播，可能导致程序不稳定或终止。

GPT总结使用noexcept的优点：
1. 明确意图和文档化：表明函数不应该抛出异常，增强代码的可读性和可维护性。
2. 编译期检查：通过 noexcept 声明，可以在编译期发现潜在的异常抛出问题，提前捕获和修复错误。
3. 性能优化：有助于编译器进行优化，减少异常处理的开销，提高代码的执行效率。
通过合理使用 noexcept，可以编写更可靠和高效的 C++ 代码。

GPT总结使用noexcept的场景
~~~
是否使用 noexcept 不仅仅是性能方面的考虑，还涉及到代码的意图表达、错误处理策略和安全性。在设计函数时，如果确定函数不会抛出异常，声明 noexcept 是一个好的实践，可以增加代码的清晰度和可靠性。如果需要异常传播和处理，则可以不声明 noexcept。两种情况下的错误查找方式虽然不同，但都有各自的调试手段和工具，实际应用中需要根据具体需求进行权衡。
~~~

虚构函数为什么默认是noexcept的
1. 双重异常会导致程序终止：如果一个析构函数抛出异常，而在析构函数执行的同时，另一个异常正在传播中，这将导致所谓的“双重异常”或“嵌套异常”情况。在这种情况下，C++标准会调用 std::terminate()，从而导致程序不正常终止
2. 资源管理中的问题：析构函数的主要目的是释放对象所持有的资源。这通常包括关闭文件、释放内存等。如果析构函数抛出异常，可能会导致资源未得到正确释放，从而引发资源泄漏问题。
3. 编译器优化和库设计：很多库和框架（包括标准库容器）假定析构函数不会抛出异常，这使得库的实现可以利用这一假定进行优化。如果析构函数抛出异常，这些优化可能会失效，并引起未定义的行为。

个人总结：
~~~
如果函数A确定不抛出异常，就要声明noexcept。那么就算A有不可预见的异常产生，我们也可以避免在堆栈展开的过程中触发额外的不可预测的另外的异常。如果函数A可能会抛出异常，我们就应该再适当的区块设计好异常处理。
~~~

## 2.7 快速初始化成员变量
1. 初始化列表效果优先于就地初始化
2. 可以利用就地初始化来避免在构造函数中的重复代码

## 2.8 非静态成员的sizeof
~~~c
struct People
{
    int hand;
    static People * all;
}

sizeof(People::all); //合法
sizeof(People::hand); //合法
~~~
可以对非静态成员变量使用sizeof

## 2.9 扩展的friend语法
声明一个类为另一个类的友元的时候，不再需要class关键字，所以可以为类模板声明友元
~~~c
class P;
template <typename T>class People
{
    friend T;
}

People<P> PP; //类型P是People类型的友元
People<int> PP; //int类型的话，friend被忽略
~~~

## 2.10 final/override控制
final用于继承关系的中途的终止，override用于描述该函数必须重载其基类中的同名函数。

## 2.11 模板函数的默认模板参数
函数模板在调用时会有实际的函数参数，这些参数可以帮助编译器推导出模板参数的类型。类模板在实例化时没有实际的函数参数可供编译器使用进行推导，因此所有的默认模板参数必须按顺序指定。

## 2.12 外部模板
### 2.12.1 为什么需要外部模板
如果一个项目中的两个文件a.c和b.c都定义了 
~~~c
int i；
~~~
那么链接器在链接a.o和b.o的时候就会出现错误，因为无法决定相同的符号是否需要合并。在其中一个声明中加上extern即可。
~~~c
extern int i;
~~~
如果在test.h中有一个模板函数
~~~c
template<typename T>void fun(T){};
~~~
在test1.cpp中
~~~
#include"test.h"
void test1(){ fun(3);}
~~~
在test2.cpp中
~~~
#include"test.h"
void test2(){ fun(4);}
~~~
编译器会在test1.o和test2.o中都实例化出一个func<int>(int)函数，链接的时候跟变量不同，连接器会删除冗余。如果项目中大量使用模板，就会有极大的工作量是用于移除重复的实例代码。
### 2.12.2 显式的实例化与外部模板的声明
在test1.cpp中
~~~
#include"test.h"
template void func<int>(int);
void test1(){ fun(3);}
~~~
在test2.cpp中
~~~
#include"test.h"
extern template void func<int>(int);
void test2(){ fun(4);}
~~~

### 2.13 局部和匿名类型作模板实参
如题

# 第三章
## 3.1 继承构造函数
~~~c
struct A{
    A(int i) {}
    A(double d, int i) {}
    A(float f, int i, const char * c) {}
    //..
}

struct B : A{
    using A::A; //这样可以直接将基类的构造函数继承
    //..
    virtual void ExtraInterface(){}
}
~~~

基类构造函数如果有默认参数，会生成多个版本的构造函数，都会被一一继承

~~~c
struct A{
    A(int a =3 ){}
    //..
};

struct B:A{
    using A:A;
    //..
}
~~~

对于A的构造函数有：
1. A()：不提供参数，使用默认值，等价于 A(3)。这是因为a参数有一个默认值。
2. A(int a)：只提供一个参数，等价于直接使用该参数值调用A(a)。

对于B的构造函数，有利用using A::A;语法继承A的构造函数，B的构造函数包括：
1. B()：这实际上是通过调用基类的A(int a = 3)构造函数来实现的，因此B()在没有提供参数时等价于B:A(3)。
2. B(int a)：这通过调用基类的A(int a)构造函数来实现，在提供一个参数时相当于B:A(a)。

以上不包括编译器默认生成。一旦使用了继承构造，编译器就不会在为派生类生成默认构造函数。

## 3.2 委派构造函数
~~~c++
class Info
{
    public:
        Info() { InitRest(); } //基准版本 称为目标构造函数
        Info(int i) : Info() { type = i; }  //称为委派构造函数
        Info(char a) : Info() { name = a; } //不能跟初始化列表混用
        //Info(int i) : Info(), type(i) { } 错误
    private:
        void InitRest() { /**/}
        int type{1};
        char name {'a'};
}
~~~
构造函数不能同时“委派”和使用初始化列表，目标构造函数总是先于委派构造函数执行，注意不要形成委托环（指A委托B，B委托C，C又委托A之类的情况）。

实际应用举例
~~~c++
class TDConstruced
{
    private:
    template<class T> TDConstruced(T first, T last): l(first, last){}
    list<int> l;
    public:
    TDConstruced(vector<short> &v):TDConstruced(v.begin(), v.end()) {}
    TDConstruced(deque<int> &d):TDConstruced(d.begin(), d.end()) {}
}
~~~

## 3.3 右值引用：移动语义和完美转发
### 3.3.1 指针成员与拷贝构造
当类中包含指针成员的话，要避免浅构造。
~~~c++
class Test
{
    public:
    int * d;
    Test():d(new int (0)) {}
    ~Test(){ delete d; }
}

void Error()
{
    Test a;
    Test b(a);
} //Error作用域结束的时候，a和b的析构函数被调用。因为默认生成的拷贝构造函数只会执行浅拷贝，即a.d和b.d都指向同一块内存。按着顺序，b的析构函数先被调用，在完成内存释放后，a.d就成了一个悬挂指针，因为其不再指向有效的内存了，在悬挂指针上释放内存会导致严重的错误。
~~~

### 3.3.2 移动语义
要注意的点
1. C++标准明确了在返回局部对象时使用移动构造函数而不是拷贝构造函数的规则。如果一个类有一个可访问的移动构造函数（且没有禁用），且返回的对象类型与函数返回类型相匹配（或者有适当的转换构造函数），那么在返回对象时将使用移动构造函数。
2. 编译器会尝试优化返回对象的过程，以避免额外的拷贝或移动。这称为返回值优化（RVO）和命名返回值优化（NRVO）。当编译器可以确定返回的对象直接用于初始化一个对象时，它可直接在目标位置构造返回值对象，从而避免拷贝或移动。

### 3.3.3 左值、右值与右值引用
1. 右值分为将亡值和纯右值。
2. 右值引用和左值引用都需要声明的时候立刻初始化，因为引用类型本身不拥有所绑定对象的内存，只是该对象的一个别名。
3. 以下例子中即使AcceptRvalueRef函数的参数Copyable && s被声明为右值引用，但在函数体内部，s实际上是一个具名变量，按照C++的规则，它被视作一个左值。
~~~c++
struct  Copyable
{
    Copyable(){}
    Copyable(const Copyable &o)
    {
        cout << "Copied" << endl;
    }
    Copyable(Copyable && o)
    {
        cout << "Moved" << endl;
    }
};

void AcceptRvalueRef(Copyable && s)
{
    Copyable news  = std::move(s);
}
~~~

4. 可以通过is_rvalue_reference等模板类进行判断类型。

### 3.3.4 std::move：强制转化为右值
std::move基本相当于static_cast<T&&>(lvalue)，是一个类型转换。为了保证移动语义的传递，在编写移动构造函数的时候，要记得使用std:move转换用于形如堆内存、文件句柄等资源的成员称为右值。如果成员支持移动构造，就可以实现其移动语义。而如果成员不支持移动构造，接收常量左值的构造函数也能实现拷贝构造

### 3.3.5 移动语义的一些问题
1. 可以通过is_move_constructible等来判断一个类型是否可以移动
2. 尽量编写不抛出异常的移动构造函数，避免移动语义未完成，一些指针会称为悬挂指针
3. move_if_noexcept模板函数代替move函数，目标类没有使用noexcept修饰移动构造函数的时候返回左值引用

### 3.3.6 完美转发
1. 引用折叠的规则就是有左就左
2. 使用方法和场景之一就是，当目标函数A对参数为左值和右值分别由不同的重载，而当我们通过函数B去调用函数A的时候，没使用完美转发情况下，我们无论使用std::move将参数传给B，还是直接将参数传给B，都是会调用A的左值版本。但如果B在调用A的时候使用了完美转发，就可以根据我们传给B的参数是左值还是右值来调用A的不同版本了。
~~~c++
#include <iostream>
#include <utility>

void A(int& x) {
    std::cout << "A(int&): Called with lvalue\n";
}

void A(int&& x) {
    std::cout << "A(int&&): Called with rvalue\n";
}

// 没有使用完美转发的函数B
template<typename T>
void B_no_forward(T x) {
    A(x); // 这里总是调用A的左值版本
}

// 使用完美转发的函数B
template<typename T>
void B_with_forward(T&& x) {
    A(std::forward<T>(x)); // 这里根据x的实际类型调用A的相应版本
}

int main() {
    int i = 42;
    std::cout << "Without perfect forwarding:\n";
    B_no_forward(i);            // 传递左值，会调用A(int&)
    B_no_forward(42);           // 传递右值，但x是左值引用类型，调用A(int&)

    std::cout << "\nWith perfect forwarding:\n";
    B_with_forward(i);          // 传递左值，会调用A(int&)
    B_with_forward(42);         // 传递右值，会调用A(int&&)
    B_with_forward(std::move(i)); // 显式传递右值，会调用A(int&&)
}
~~~
3. 通过使用完美转发，我们可以避免定义多个重载函数，而只需一个模板函数。
~~~c++
#include <iostream>
#include <utility>

template <typename T>
void process(T&& x) {
    if constexpr (std::is_lvalue_reference_v<T>) {
        std::cout << "process: Called with lvalue\n";
    } else {
        std::cout << "process: Called with rvalue\n";
    }
}

template <typename T>
void forwardToProcess(T&& x) {
    process(std::forward<T>(x)); // 使用 std::forward 进行完美转发
}

int main() {
    int a = 10;
    const int b = 20;

    forwardToProcess(a);       // 调用 process(int&)
    forwardToProcess(30);      // 调用 process(int&&)
    forwardToProcess(b);       // 调用 process(const int&)
    forwardToProcess(40);      // 调用 process(const int&&)
}
~~~

4. 额外注意：在模板函数中，除非明确使用引用类型，否则传递的参数会被复制，变成右值。

## 3.4 显式转换操作符
1. 使用explicit关键字保证对象不能被隐式构造。
2. operator bool()是一个操作符，它是一种用户定义的类型转换运算符，用于将类类型的对象转换为 bool 类型。bool可以替换成别的类型，也是另外的操作符。

## 3.5 列别初始化
### 3.5.1 列表初始化
1. 支持大括号 
2. 包含initializer_list就可以支持初始化列表，函数的参数列表也行

### 3.5.2 防止类型收窄
1. 使用初始化列表的数据编译器会检查是否发生类型收窄

### 3.6 POD类型
POD为两个基本概念的合集平凡的和标准布局的，平凡的很容易理解，注意书上例子Nontrivial2，即使构造函数被声明为=default，只要它的声明在类体外部，这个类型就不再是平凡的。

标准布局的三条规则
1. 所有非静态成员拥有相同的访问权限
2. 在类或者结构体继承时，满足以下两种情况之一：
    * 派生类中非静态成员，且只有一个仅包含静态成员的基类。
    * 基类有非静态成员，而派生类没有非静态成员

    实际上可以说是只要非静态成员出现继承关系网中的两个类就会被判断为不是标准布局
3. 类中第一个非静态成员的类型与其基类不同，因为C++标准要求类型相同的对象必须地址不同
4. 没有虚函数和虚基类
5. 所有非静态数据成员均符合标准布局，其基类也符合标准布局。

## 3.7 非受限联合体
P109要保证析构的是T为string对象，这里说明了如果使用t.n，就会在设置 t.n 的值时破坏了已经构造好的 t.s 对象，而在 union 析构时试图销毁这个已被破坏的 std::string 对象，会导致未定义行为。

## 3.8 用户自定义字面量
1. 字面为整型数、浮点熟、字符串和字符的情况下，参数各有不同，但都固定
2. operator"" 与用户自定义后缀之间必须有空格，且建议以下划线开始

## 3.9 内联名字空间
1. 内联的名字空间允许程序员在父名字空间定义或特化子名字空间的模板。
2. 还可以用于版本管理
~~~c++
namespace FUNC
{
    inline namespace version1
    {
        void func() {};
    }
    namespace version2
    {
        void func() {};
    }
}
~~~
这样默认的FUNC::func()就是version1的，而且在想要使用version2的时候可以，FUNC::version2::func()

## 3.10 模板的别名
using比typedef强大
~~~c++
template<typename T> using strmap = map<T, char*>;
strmap<int> numstrmap;
~~~

## 3.11 一般化的SFINAE规则
substitution failure is not an error，匹配失败不是错误。指重载模板的参数进行展开的时候，遇到类型不匹配不会报错，而是会接着找合适的。

# 4 新手易用，老兵易学
## 4.1 右尖括号>的改进
编译器会智能检测是否为右移操作

## 4.2 auto类型推导
### 4.2.1 静态类型、动态类型与类型推导
auto并非“类型”声明，而是一个类型声明时的“占位符”，编译器会在编译时期将auto替换为变量实际的类型

### 4.2.2 auto的优势
1. 简化代码
2. 避免部分类型声明的错误
3. 一定程度支持泛型

## 4.2.3 auto的使用细则
1. 如果要auto声明的变量是另一个变量的引用的话，必须使用auto &
2. 只有声明为auto & 和auto *的变量能带走cv限制符
3. auto 不能是函数形参
4. 结构体中非静态成员变量不能是auto
5. 不能声明 auto数组
6. 实例化模板的时候不能用auto

## 4.3 decltype
### typeid 与decltype
auto和decltype类型推导都是在编译时进行的

### decltype的应用
略

### decltype推导四规则
decltype(e)，**依序**判断以下四规则
1. 如果e是一个没有带括号的标记表达式或类成员访问表达式，decltype(e)就是e所命名的实体的类型。如果e是一个被重载的函数，编译时报错
2. 如果e是将亡值，类型为T，decltype(e)为T&&
3. 如果e是左值，类型为T，decltype(e)为T&
4. e的类型是T，则decltype(e)为T

按顺序判断很重要，标记表达式为形如：int arr[3];中的arr，或者double ff；中的ff

### 4.3.4 cv限制符的继承与冗余的符号
decltype能“带走”表达式的cv限制符，*不会被忽略。

## 4.4 追踪返回类型

### 4.4.1 追踪返回类型的引入
~~~c++
template<typename T1, typename T2>
auto sum(T1 & t1, T2 & t2) -> decltype(t1 + t2)
{
    return t1 + t2;
}
~~~

### 4.4.2 使用追踪返回类型的函数
可以省略作用域

## 4.5 基于范围的for循环
略

# 5 提高类型安全
## 5.1 强类型枚举
### 5.1.1 枚举：分门别类与数值的名字
因为编译器会默认为枚举值赋值，所以需要数值的名字的时候，可以使用，但并不推举
~~~c
 enum {zero, first, second};
~~~
最好是使用静态常量
### 5.1.2 有缺陷的枚举类型
1. namespace也无法分割enum全局可见
2. 总是隐式被转换成整型
3. 所占空间大小也是一个“不确定量”

### 5.1.3 强类型枚举以及C++11对原有枚举类型的扩展
相比起以前
1. 强作用域，强类型枚举成员的名称不会被输出到其父作用域空间。
2. 转换限制，强类型枚举成员的值不可以与整型隐式地相互转换。
3. 可以指定底层类型。默认为int，但可以显式地指定底层类型。
~~~c++
enum class Type: char{fir, sec, thir};
~~~

扩展的地方，保证向后兼容。
~~~c++
enum Type:char { fir, sec, thir}; //合法了
Type t1 = fir;
Type t2 = Type::sec;//也合法
~~~
enum class的成员没有公私之分，也没有模板，enum struct也一样的。

## 5.2 堆内存管理：智能指针与垃圾回收
### 5.2.1 显式内存管理
会出现运行程序占用内存越来越多的原因可以归为：
1. 野指针
2. 重复释放
3. 内存泄漏

### 5.2.2 c++11的智能指针
weak_ptr可用于检查share_ptr的有效性

### 5.2.3 垃圾回收的分类
weak_ptr可用于解决share_ptr引起的环形引用问题

### 5.2.4 C++与垃圾回收
因为指针过于强大，可能会导致不安全
1. static_cast：用于相关类型之间的安全转换，编译时检查。
2. dynamic_cast：用于继承层次结构中的安全向下转换，运行时检查。
3. const_cast：用于添加或移除 const 或 volatile 属性。
4. reinterpret_cast：用于不进行类型检查的强制转换，适用于低级别编程。

### 5.2.5 C++11与最小垃圾回收支持
安全派生的指针是指向new出来的对象或其子对象的指针，安全派生指针的操作包括：
1. 在解引用基础上的引用，比如：&*p。
2. 定义明确的指针操作，比如：p+1。
3. 定义明确的指针转换，比如:static_cast<void *> p。
4. 指针和整型之间的reinterpret_cast，比如reinterpret_cast<intptr_t> p。

### 5.2.6 垃圾回收的兼容性
略

# 6 提高性能及操作硬件的能力
## 6.1 常量表达式
### 6.1.1 运行时常量性与编译时常量性
通过constexpt可以使编译器在编译的时候就进行值计算

### 6.1.2 常量表达式函数
常量表达式函数必须：
1. 函数体只有单一的return返回语句。
2. 函数必须返回值，不能是void。
3. 在使用前必须已有定义。
4. return返回语句表达式中不能使用非常量表达式的函数、全局数据，且必须是一个常量表达式。

### 6.1.3 常量表达式值
自定义类型
~~~c++
struct mytype
{
    constexpr mytyoe(int x) : i(x) {}
    int i;
}
~~~
1. 函数体必须为空
2. 初始化列表只能由常量表达式来赋值

### 6.1.4 常量表达式的其他应用
constexpr应用于模板函数的时候，如果实例化结果不满足常量表达式的需求的话，会忽略constexpr。constexpr元编程在编译器支持的情况下，可以在编译时候就完成一些计算工作，因为constexpr支持浮点数运算，支持三元表达式、逗号表达式。

## 6.2 变长模板
### 6.2.1 变长函数和变长的模板参数
可以使用va_list相关参数来实现变长参数的使用。但使用C 风格的变长参数列表 (...) 来接受不定数量的参数的话，会导致函数“本身”不知道参数数量和类型。C++11中我们使用tuple模板来实现变长模板

### 6.2.2 变长模板：模板参数包和函数参数包
~~~c++
template<int... values> //values为模板参数包
struct sumint;

template<int fir, int ... res> //res为模板参数包
struct sumint<fir, res...>//为了使用模板参数包，需要进行解包，通过表达式res...，又称为包扩展，来完成解包
{
    static const int value = fir + sumint<res...>::value;
};

template<>
struct sumint<>
{
    static const int value = 0;
};

int main() 
{
    std::cout << "Sum of 1, 2, 3 is: " << sumint<1, 2, 3>::value << std::endl; // 输出: 6
    return 0;
}
~~~

### 6.2.3 变长模板：进阶
~~~c++
template<typename... A >class T: private B<A>... {};
template<typename... A >class T: private B<A...> {};
//上面两个表达式是不一样的，对同样实例化T<X, Y>
//第一行是
class T<X, Y>: private B<X>, private B<Y> {};
//第二行是
class T<X, Y>: private B<X, Y> {};
//相同的情况也会发生在模板函数中

template<class... A> 
int Vaargs(A... args)
{
    int size = sizeof...(A);//变长包的长度
    //...
}
~~~
可以使用组合的方式实现变长模板

## 6.3 原子类型与原子操作
### 6.3.1 并行编程、多线程与C++11
略
### 6.3.2 原子操作与C++11原子类型
1. 通过包含<cstdatomic>包含内置类型对应的原子类型类似于**atmoic_llong**，可以不再需要显式加互斥锁。
2. 还可以使用atomic类模板来定义出需要的原子类型。
~~~c++
std::atomic<T> t;
~~~
3. 原子类型属于**资源型**的数据，所以atomic模板类的拷贝、移动、赋值等构造函数都是被默认删除的。
4. atomic_flag是无锁的，可以使用以下代码做到线程等待的效果
~~~c++
atomic_flag lock = ATOMIC_FLAG_INIT;

void f(int n)
{
    while(lock.test_and_set(memory_order_acquire))
    {
        cout << "Waiting from thread " << n << endl;
        cout << "Thread " << n << " starts working" << endl;
    }
}

void g(int n)
{
    cout << "Thread " << n << " is going to start." << endl;
    lock.clear();
    cout << "Thread " << n << " starts working" << endl;
}
~~~

### 6.3.3 内存模型，顺序一致性与memory_order
1. 代码再线程中运行的顺序与程序员看到的代码顺序一直，被称为“顺序一致性”。
2. 内存模型通常指机器指令或汇编语言是以什么样的顺序被执行器处理的，顺序一致性只是其中一种。
3. 编译器会将代码前后移动，用以获得最佳的机器指令的排列和产生最近的运行时性能。
4. 程序员可以通过以下枚举值来控制执行顺序:
    - memory_order_relaxed : 不对执行顺序做任何保证
    - memory_order_acquire : 本线程中，所有后续的读操作必须在本条原子操作完成后执行
    - memory_order_release : 本线程中，所有之前的写操作完成后才能执行本条原子操作
    - memory_order_acq_rel : 同时包含memory_order_acquire和memory_order_release
    - memory_order_consume : 本线程中，所有后续的有关本原子类型的操作，必须在本条原子操作完成之后执行
    - memory_order_seq_cst : 全部存取都按顺序执行
5. 使用例子如下
~~~c++
#include <thread>
#include <atomic>
#include <iostream>
using namespace std;

atomic<int> a{0};
atomic<int> b{0};

void set()
{
    int t = 1 ;
    a.store(t, memory_order_relaxed);
    b.store(t, memory_order_relaxed);
}

void read()
{
    cout << "(" << a << ", " << b << ")" << endl;//可能会有多种输出，因为使用了memory_order_relaxed来写入a和b
}

int main()
{
    thread t1(set);
    thread t2(read);

    t1.join();
    t2.join();
    cout << "Got (" << a << ", " << b << ")" << endl;
    return 0;
}
~~~
6. 常见的内存顺序由：顺序一致、松散、release-acquire和release-consume

## 6.4 线程局部存储
通过thread_local修饰符，可以将变量声明为TLS变量，其值在线程开始时被初始化，而线程结束时不再有效。

## 6.5 快速退出：quick_exit与at_quick_exit
因为使用exit()函数退出的话，会先向线程发出一个信号，并等待线程结束再执行析构函数和使用atexit()注册的函数。这样的退出方式有时候可能会卡死在半路上导致无法退出。

所以C++11引入了quick_exit()和at_quick_exit()配合，可以不执行析构函数只是等程序终止。

# 7 为改变思考方式而改变
## 7.1 指针空值-nullptr
### 7.1.1 指针空值：从0到NULL，再到nullptr
### 7.1.2 nullptr和nullptr_t
### 7.1.3 一些关于nullptr的讨论
nullptr不能被取地址，被定义为一个右值常量。

## 7.2 默认函数的控制
### 7.2.1 类与默认函数
### 7.2.2 "=default"和"=deleted"
可以通过=deleted来禁止类型转换、禁止使用new对象和禁止析构函数，禁止析构函数可以用于构建单例模式。
~~~c++
void test(int i) {};
void test(char) = delete; 

class noheapalloc
{
    public:
        void * operator new(std::size_t) = delete;
};

extern void* p;
class nostackalloc
{
    public:
        ~nostackalloc() = delete;
};

int main()
{
    test(3);
    test('a'); //编译失败

    noheapalloc nha;
    noheapalloc * pnha = new noheapalloc; //编译失败
    
    nostackalloc nsa;//编译失败
    new (p) nostackalloc(); //placement new 假设p无需调用析构函数
}
~~~

## 7.3 lambda函数
### 7.3.1 lambda的一些历史
### 7.3.2 C++11中的lambda函数
### 7.3.3 lambda与仿函数
1. lambda函数可以被视为一个类有一个operator()成员函数，捕获的对象为类内成员，并且operator()默认为const。可以使用mutable修饰，在使用了mutable修饰之后，参数列表即使为空也不可省略。
2. 捕捉列表不允许变量重复传递。
3. 在块作用域外的lambda函数捕捉列表必须为空。
4. 比起普通函数，lambda函数和仿函数的优势是可以具有初始状态

### 7.3.4 lambda的基础使用
1. lambda函数对比函数外定义的static inline函数还是对比自定义的宏，不会又实际运行时的优势，但不差。
2. lambda一个优势明显的使用场景，在一个复杂运算的函数中，会有大量的局部状态，这时候如果需要一些固定的操作形如打印、计算，这些操作往往不需要与其他代码共享，却需要在一个函数中多次重用。
3. lambda函数的好处就是会随着父作用域结束而结束，不会污染命名空间，可读性也更好。

### 7.3.5 关于lambda的一些问题及有趣的实验
1. lambda函数如需要捕捉的值称为lambda函数的常量的话，可以使用按值传递的方式捕捉，反之使用按引用传递。
2. lambda不是函数指针类型也不是自定义类型，在C++11中被定义为一种名为“闭包”的类。允许lambda表达式向函数指针的转换，前提是lambda没有捕捉变量，并且函数指针的原型跟lambda函数有相同的方式。
### 7.3.6 lambda与STL
### 7.3.7 更多的一些关于lambda的讨论

# 8 融入实际应用
## 8.1 对齐支持
### 8.1.1 数据对齐
### 8.1.2 C++11的alignof和alignas
对齐方式的查看alignof，设定alignas，STL库函数std::align，STL库模板类型aligned_storage和aligned_union，都可以帮助程序员实现对齐。

## 8.2 通用属性
### 8.2.1 语言扩展到通用属性
### 8.2.2 C++11的通用属性
### 8.2.3 预定义的通用属性
1. [[noreturn]]用于标识哪些不会将控制流返回给原调用函数的函数，例子有：终止应用程序语句的函数、有无限循环语句的函数。
2. [[carries_dependecy]]用于弱内存模型并且很关心并行程序的执行性能时优化。

## 8.3 Unicode支持
### 8.3.1 字符集、编码和Unicode