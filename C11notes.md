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

### 3.5.3 POD类型
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