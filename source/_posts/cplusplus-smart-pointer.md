---
title: C++智能指针
date: 2017-10-10
categories: C++
tags: C++
---

C++中智能指针（auto_ptr、unique_ptr、shared_ptr）的简单总结。
<!--more-->

### 智能指针的设计思想
    先看一个简单的例子：
```cpp
void func(std::string & str)
{
    std::string * ps = new std::string(str);
    ...
    if (weird_thing())
        throw exception();
    str = *ps; 
    delete ps;
    return;
}
```
当出现异常时（weird_thing()返回true），delete将不被执行，因此将导致内存泄露。如何避免这种问题？直接在throw exception();之前加上delete ps;。这样确实可行，但更多时候开发者都会忘记在适当的地方加上delete语句。换个角度想，如果func()函数终止（不管是正常终止还是异常终止），局部变量都会自动从栈内存中删除，因此指针ps占据的内存将被释放，如果ps指向的内存也能被自动释放，这样就不会出现内存泄漏的问题，析构函数确实有这个功能，如果ps有一个析构函数，该析构函数将在ps过期时自动释放它指向的内存。但是问题在于ps只是一个常规指针，不是有析构凼数的类对象指针。因此引出了智能指针的设计思想：将常规指针封装成类（模板类，以适应不同类型的需求），并在析构函数里编写delete语句删除指针指向的内存空间。这样就无需程序员显式调用delete语句释放内存，也不会因为程序异常退出导致内存泄漏。

### 智能指针的简单介绍
&emsp;&emsp;C++一共提供了四种智能指针：auto_ptr、unique_ptr、shared_ptr和weak_ptr（此处只介绍前三种）。auto_ptr是C++98提供的解决方案，C+11已将其摒弃，并提供了另外两种解决方案。然而，虽然auto_ptr被摒弃，但已使用了多年；如果你所用的编译器不支持其他两种解决力案，auto_ptr将是唯一的选择。
- 所有的智能指针都有一个explicit构造函数，以指针作为参数，因此不能自动将指针转换为智能指针对象，必须显示调用
```cpp
shared_ptr<double> pd; 
double *p_reg = new double;
pd = p_reg;                               // not allowed (implicit conversion)
pd = shared_ptr<double>(p_reg);           // allowed (explicit conversion)

shared_ptr<double> pshared = p_reg;       // not allowed (implicit conversion)
shared_ptr<double> pshared(p_reg);        // allowed (explicit conversion)

shared_ptr<double> ps = new double(10.0);   // not allowed (implicit conversion)
shared_ptr<double> ps(new double(10.0));    // allowed (explicit conversion)
```
- 应避免将存储在栈区的普通变量或对象的地址作为智能指针的初始化参数
```cpp
int number = 10;
shared_ptr<int> pvac(&number);   // pvac过期时，程序将把delete运算符用于非堆内存，这是错误的
```

### auto_ptr
&emsp;&emsp;对于下面的赋值语句，可以看出，两个常规指针指向同一个对象，这样会导致错误的结果，因为程序试图销毁同一个对象两次，除非程序员特别注意只delete一个指针，但这个过程是不可控的，也应尽量避免在程序中留下这种隐患。
```cpp
string *p1 = new string("Hello World!");
string *p2 = p1;
...
delete p1;
delete p2;
```
要避免这种问题，有多种方法：
- 在复制指针的时候采用深拷贝。这样两个指针将指向不同的对象，其中一个对象时另一个对象的副本。缺点是浪费空间，所有智能指针都未采用此方案。
- 建立所有权概念。对于特定的对象，只有一个智能指针可拥有，在进行复制操作时，会转让对象的所有权，这样只有拥有该对象的智能指针在生命周期结束时会自动销毁对象。这就是auto_ptr和unique_ptr的策略，但unique_ptr的策略更严格。
- 创建智能程度更高的指针，跟踪引用（指向）特定对象的智能指针数，称为引用计数。例如赋值时引用计数加一，而指针过期时计数减一，当减为0时才调用delete。这是shared_ptr采用的策略。

同样的策略适用于赋值运算符（重载）和复制构造函数。但是auto_ptr存在一定的弊端，这也是在后面的版本中将其摒弃的原因。举个例子：
```cpp
#include <iostream>
#include <string>
#include <memory>   //包含智能指针的头文件
using namespace std;

int main()
{
    auto_ptr<string> str[5] = {
        auto_ptr<string> (new string("What"));
        auto_ptr<string> (new string("Why"));
        auto_ptr<string> (new string("Where"));
        auto_ptr<string> (new string("when"));
        auto_ptr<string> (new string("How"));
    }

    auto_ptr<string> s;
    s = str[2];	//str[2]所指对象的所有权转让给了s
    if(str[2].get() == NULL)    //此处判断条件为真
        cout << "The pointer is null!" << endl;

    for(int i = 0; i < 5; i++)
        cout << *str[i]<< endl;
    return 0;
}
```
上面程序运行会崩溃，因为str[2]转让所有权后（对应的裸指针）变成空指针了，因此输出访问空指针必然会崩溃。如果将auto_ptr换成shared_ptr或unique_ptr后，程序不会崩溃。
- 使用shared_ptr时运行正常，因为shared_ptr采用引用计数，s与str[2]指向同一块内存，同一个对象，在释放空间时会先判断引用计数值的大小，不会出现多次删除同一个对象的情况。
- 使用unique_ptr时编译出错，与auto_ptr一样，unique_ptr也采用所有权模型，但在使用unique时，程序不会等待运行阶段崩溃，而会在编译期就报错。

之所以要摒弃auto_ptr，就是要避免潜在的内存崩溃问题。

### unique_ptr
&emsp;&emsp;unique_ptr和auto_ptr一样，一个对象只能被一个智能指针所占有，这样可防止多个指针试图销毁同一个对象。但在进行复制操作后，auto_ptr可能存在潜在的内存崩溃问题，如果访问失去对象所有权的指针的话，而unique_ptr则会在编译期提前报错。此外，unique_ptr还有其他优势，例如下面的例子
```cpp
unique_ptr<string> func(const char* s)
{
    unique_ptr<string> temp(new string(s));
    return temp;
}

unique_ptr<string> ps = func("Hello World");
```
func()返回一个临时的unique_ptr，ps接管了临时unique_ptr所指的对象，而返回时该临时unique_ptr被销毁，后面没有机会通过该指针来访问无效的数据。也就是说这种赋值是没有问题的，实际上，编译器也允许unique_ptr的这种赋值。
简单总结，当程序试图将一个unique_ptr赋值给另一个时，如果源unique_ptr是个临时右值，编译器允许这么做；如果源unique_ptr将存在一段时间，编译器将禁止这种做法。
```cpp
unique_ptr<string> ps1(new string("Hello World"));	
unique_ptr<string> ps2 = ps1;   //调用拷贝构造函数 #1 not allowed
/*
unique_ptr<string> ps2;
ps2 = ps1；  //赋值运算符重载（operator=）
*/

unique_ptr<string> ps3;	    //调用默认（无参）构造函数
ps3 = unique_ptr<string>(new string("Hello World"));    //先调用有参构造函数创建临时对象，再调用赋值运算符	#2 allowed
```
前面已经提到，第一种情况#1编译器是会报错的，转让对象所有权留下了空指针，而第二种情况#2是先创建一个unique_ptr临时对象，然后将所指对象的所有权转让给ps3。这种随情况而异的行为表明，unique_ptr从安全性和实用性方面都优于auto_ptr。
如果确实想对unique_ptr对象执行普通的赋值操作，可借助移动语义实现（将左值转换为右值，std::move()）,能够直接将一个unique_ptr赋给另一个，使用move()后，原来的指针转让所有权成为空指针，可对其重新赋值后再使用。
```cpp
unique_ptr<string> ps1 = unique<string>(new string("Hello World!"));    //等价于unique_ptr<string> ps1(new string("Hello world!"))，直接调用有参构造函数
uinque_ptr<string> ps2 = std::move(ps1);
if(ps1 == NULL) //此处判断条件为真，与auto_ptr必须通过get()成员函数返回指针略有不同，ps1可直接和指针进行比较（应该是重载了operator==），也可通过get()函数返回指针
    cout << "The pointer is null!" << endl;
```

### shared_ptr
&emsp;&emsp;shared_ptr是通过引用计数来共享同一个对象的智能指针，也就是说多个shared_ptr指针可指向同一个对象，通过引用计数的方式可避免多次释放内存。但在使用shared_ptr指针时也存在一些陷阱。见下面例子：
```cpp
// #1 正确用法
std::shared_ptr<int> ps1(new int(0));
std::shared_ptr<int> ps2 = ps1;

// #2 错误用法
int *p = new int(0);
std::shared_ptr<int> ps3(p);
std::shared_ptr<int> ps4(p);
```
第一个例子中是两个shared_ptr指针共享一个对象，ps1和ps2是有关联的，引用计数为2，最后一个负责释放new的变量；第二个例子中是两个独立的shared_ptr指针指向了同一个对象，ps3和ps4是没有关联的，他们并不知道对方的存在，因此会争相去释放p指针，导致重复释放。一个裸指针只能用来初始化一个shared_ptr指针，如果要让对象在多个指针间共享，需通过shared_ptr指针之间的拷贝，而不能直接拷贝原始指针。
&emsp;&emsp;使用智能指针难以避免的场景之一就是需要把 this 当成一个 shared_ptr 传递到其他函数中去，不能简单粗暴的用 this 指针构造一个 shared_ptr，因为那样做会导致两个独立的 shared_ptr 指向同一份内存资源，就像第二个例子那样
```cpp
class Widget 
{
    std::shared_ptr<Widget> GetPtr() 
    {
        return shared_ptr<Widget>(this); // 错误
    }
};

int main() 
{
    auto widget = std::shared_ptr<Widget>();
    widget->GetPtr();
}
```
正确的做法是继承模板类 std::enable_shared_from_this，然后调用它的 shared_from_this 成员函数，这种把自己作为基类的模板参数的做法称为——奇异递归模板模式
```cpp
class Widget : public std::enable_shared_from_this<Widget> 
{
    std::shared_ptr<Widget> GetPtr()
    {
        return shared_from_this();
    }
};

int main() 
{
    auto widget = std::shared_ptr<Widget>();
    widget->GetPtr();
}
```
&emsp;&emsp;在裸指针被初始化给多个shared_ptr的异常场景下，shared_from_this返回的对象将会增加哪个shared_ptr的引用计数呢？ 对于这种未定义的行为通常答案是由编译器决定。可以看出裸指针通过shared_from_this返回的对象与最近一个初始化的share_ptr相关联。
```cpp
class Widget: public std::enable_shared_from_this<Widget>
{
public:
    std::shared_ptr<Widget> xxx()
    {
        return shared_from_this();
    }
};

int main(int argc, char** argv) 
{
    Widget *p = new Widget();
    std::shared_ptr<Widget> one1(p);
    std::shared_ptr<Widget> one2(one1);
    std::shared_ptr<Widget> one3(one1);
    std::cout << one1.use_count() << std::endl; //3
    std::cout << one2.use_count() << std::endl; //3
    std::cout << one3.use_count() << std::endl; //3

    std::shared_ptr<Widget> two1(p);    //这种用一个裸指针初始化多个shared_ptr指针的行为实际上是不允许的
    std::cout << two1.use_count() << std::endl; //1
    std::shared_ptr<Widget> guess = p->xxx();
    std::cout << one1.use_count() << std::endl; //3
    std::cout << two1.use_count() << std::endl; //2
    std::cout << guess.use_count() << std::endl; //2
    return 0; // crash at end
}
```

### 智能指针管理动态数组
C++11标准库提供了一个可以管理new分配的数组的unique_ptr版本，使用方式如下：
```cpp
unique_ptr<Object[]> spArray(new Object[2]);
```
类型说明符中的方括号<Object[]>指出spArray指向一个Object数组，当spArray销毁它管理的指针时，会自动使用delete[]。由于spArray指向一个数组，所以不能直接使用”.”和”->”成员运算符。这时，可以使用下标运算符[]来访问数组中的元素:spArray[i]  返回spArray在数组中位置i处的对象，其中spArray指向一个数组。
注意：不能使用智能指针管理静态数组，下面这种用法是错误的
```cpp
int num[3] = {1, 2, 3};
unique_ptr<int[]> sp(num);
```

与unique_ptr不同，shared_ptr不支持直接管理动态数组，也不支持[]运算符。如果希望使用shared_ptr来管理动态数组，可以通过指定delete []作为删除器(不指定的话默认使用delete作为删除器)：
```cpp
//下面两种方法均可
shared_ptr<Object>sp(newObject[2], [](int *p){delete[] p;});
shared_ptr<int> p(new int[10],std::default_delete<int[]>());
```
由于不支持下标运算符[]，因此，为了访问元素，必须用get获取一个内置指针，然后通过它来访问数组元素，如下所示：
```cpp
for(size_ti = 0; i < 2; ++i)
{
    *(sp.get() + i) = i;
}
```


### 如何选择智能指针
- 如果程序要使用多个指向同一个对象的指针，应该选择shared_ptr。例如STL容器中包含指针，很多STL算法都支持复制和赋值操作，这些操作可用于shared_ptr，而不能用于unique_ptr（编译器warning警告）和auto_ptr（行为不确定）。
- 如果程序不需要多个指向同一个对象的指针，则可使用unique_ptr，可省去显式调用delete销毁对象的过程；而且unique_ptr可通过传引用的方式作为函数的参数，传值则不允许。
- 尽量不要使用auto_ptr指针。

```cpp
unique_ptr<int> make_int(int n)
{
    return unique_ptr<int>(new int(n));	
}

void show(unique_ptr<int> &p)	//形参为引用是允许的，传值则不允许
{
    cout << *a << ' ';
}

int main()
{
    ...
    vector<unique_ptr<int> > vp(size);
    for(int i = 0; i < vp.size(); i++)
        vp[i] = make_int(rand() % 1000);       // copy temporary unique_ptr
    vp.push_back(make_int(rand() % 1000));     // ok because arg is temporary
    for_each(vp.begin(), vp.end(), show);      // use for_each()
    ...
}
```
其中push_back调用没有问题，因为它返回一个临时unique_ptr，该unique_ptr被赋给vp中的一个unique_ptr。另外，如果按值而不是按引用给show()传递对象，for_each()将非法，因为这将导致使用一个来自vp的非临时unique_ptr初始化p，而这是不允许的，编译器会报错。

在unique_ptr为右值时，可将其赋给shared_ptr，这与将一个unique_ptr赋给另一个指针需要满足的条件相同。
```cpp
unique_ptr<int> p1(make_int(rand() % 1000));   // ok
shared_ptr<int> p2(p1);                        // not allowed, p1 as lvalue
shared_ptr<int> p3(make_int(rand() % 1000));   // ok
```
模板shared_ptr包含一个显式构造函数，可用于将右值unique_ptr转换为shared_ptr。shared_ptr将接管原来归unique_ptr所有的对象。在满足unique_ptr要求的条件时，也可使用auto_ptr，但unique_ptr是更好的选择。如果你的编译器没有unique_ptr，可考虑使用Boost库提供的scoped_ptr，它与unique_ptr类似。

### reference
- http://yizhi.ren/2016/11/14/sharedptr/
- http://blog.guorongfei.com/2017/01/25/enbale-shared-from-this-implementaion/
- http://www.cnblogs.com/lanxuezaipiao/p/4132096.html