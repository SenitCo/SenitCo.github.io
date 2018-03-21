---
title: C++异常机制
date: 2017-10-15
categories: C++
tags: C++
---

&emsp;&emsp;C++异常机制用于处理程序中的异常事件，是一种有效处理运行错误的强大且灵活的工具。
<!--more-->

### C++异常机制概述
&emsp;&emsp;C++的异常情况主要分为两种，一种是编译时的语法错误，另一种是运行时异常，例如访问越界，内存不足等。异常机制专门用于处理运行时异常。异常事件在C++中表示为异常对象，异常事件发生时，程序使用throw关键字抛出异常对象，抛出点称为异常出现点，由操作系统为程序设置当前异常类型，然后执行程序的当前异常处理代码块，在包含了异常出现点的最内层的try块，依次匹配catch语句中的异常对象。若匹配成功，则执行catch块内的异常处理语句，然后接着执行try...catch...块之后的代码。如果在当前的try...catch...块内找不到匹配该异常对象的catch语句,则由更外层的try...catch...块来处理该异常；如果当前函数内所有的try...catch...块都不能匹配该异常，则递归回退到调用栈的上一层去处理该异常。如果一直回退到主函数main()都不能处理该异常，则调用系统函数terminate()终止程序。

### try、throw、catch关键字
&emsp;&emsp;throw与抛出表达式构成了throw语句，throw语句必须包含在try块中，也可以是被包含在调用栈的外层函数的try块中。如果在 try 语句块的程序段中（包括在其中调用的函数）发现了且抛出了该异常，则这个异常就可能被 try 语句块后的某个catch语句所捕获并处理，捕获和处理的条件是被抛出的异常对象的类型与catch语句的异常类型相匹配。
&emsp;&emsp;catch语句匹配被抛出的异常对象。如果catch语句的参数是引用类型，则该参数可直接作用于异常对象，即参数的改变也会改变异常对象，而且在catch中重新抛出异常时会继续传递这种改变。如果catch参数是传值的，则复制构造函数将依据异常对象来构造catch参数对象。在该catch语句结束的时候，先析构catch参数对象，然后再析构异常对象。
```cpp
try 
{  
    if(满足异常条件)
        throw 异常对象;  
}  
catch(类型名 [形参名]) // 捕获特定类型的异常  
{  
 
}  
catch(类型名 [形参名]) // 捕获特定类型的异常  
{  
 
}  
catch(...)    // 三个点则表示捕获所有类型的异常  
{  

}
```

异常处理示例：处理除数为0的异常，用try/catch语句来捕获异常，并使用throw语句来抛出异常。
```cpp
#include<iostream>       
#include<cstdlib>
using namespace std;  
 
double func(double x, double y)
{  
    if(y == 0)  
    {  
        throw y;     //除数为0，抛出异常  
    }  
    return x / y;     //否则返回两个数的商  
}  
 
void main()  
{  
    double res;  
    try  //定义异常  
    {   
        res = fuc(4, 0);      //出现异常，函数内部会抛出异常  
    }  
    catch(double)             //捕获并处理异常  
    {  
         cerr << "error of dividing zero.\n";  
         exit(1);             //异常退出程序  
    }  
    cout << "The result of x / y is : " << res << endl;
}
```

### C++标准异常
&emsp;&emsp;C++标准库提供了一组异常类，用户可以在程序中使用这些标准的异常，也可以通过继承和重载 exception 类来定义新的异常。

<img src="https://i.loli.net/2018/03/21/5ab25f01bf173.jpg" alt="cpp_exceptions.jpg" />

下表是对上图层次结构中每个标准异常类的简单说明：

异常                        |描述
:---:                       | :---:
std::exception              |该异常是所有标准 C++ 异常的父类
std::bad_alloc              |该异常可以通过 new 抛出
std::bad_cast               |该异常可以通过 dynamic_cast 抛出
std::bad_exception          |这在处理 C++ 程序中无法预期的异常时非常有用
std::bad_typeid             |该异常可以通过 typeid 抛出
std::logic_error            |理论上可以通过读取代码来检测到的异常
std::domain_error           |当使用了一个无效的数学域时，会抛出该异常
std::invalid_argument       |当使用了无效的参数时，会抛出该异常
std::length_error           |当创建了太长的 std::string 时，会抛出该异常
std::out_of_range           |该异常可以通过方法抛出，例如 std::vector 和 std::bitset<>::operator
std::runtime_error          |理论上不可以通过读取代码来检测到的异常
std::overflow_error         |当发生数学上溢时，会抛出该异常
std::range_error            |当尝试存储超出范围的值时，会抛出该异常
std::underflow_error        |当发生数学下溢时，会抛出该异常

下面是通过继承和重载 exception 类来定义新异常类的简单示例：
```cpp
#include <iostream>
#include <exception>
using namespace std;

class MyException : public exception
{
    const char* what() const   //what()是异常类提供的一个公共方法，它能被所有子异常类重载。
    {
        return "C++ Exception";
    }
};
 
int main()
{
    try
    {
        throw MyException();
    }
    catch(MyException& e)
    {
        std::cout << "MyException caught" << std::endl;
        std::cout << e.what() << std::endl;
    }
    catch(std::exception& e)
    {
        std::cout << e.what() << std::endl;
    }
}
```

### 异常声明
C++98中可使用throw作为函数的异常声明，例如
```cpp
void func() throw(int ,double ) {...} 
void func() throw(){...}
```
前者表示func()函数可能会抛出int或double类型的异常，后者表示func()函数不会抛出异常。C++11中已经摒弃了这种做法，不会抛出异常的声明由noexcept替代。
```cpp
void func() noexcept {...} //等价于void func() throw(){...}
```

### 异常机制与构造函数、析构函数
&emsp;&emsp;构造函数没有返回值，应该使用异常机制来报告发生的问题。但构造函数抛出异常表明构造函数没有执行完，不会生成特定的对象，对应的析构函数也不会自动被调用。因此在构造函数中抛出异常之前，需要先释放已经申请的资源。
C++类构造函数初始化列表的异常机制，称为function-try block。一般形式为：
```cpp
myClass::myClass(type t) try: val(初始化值)
{ 
    /*构造函数的函数体 */
} 
catch(exception& err) 
{
    /* 构造函数的异常处理部分 */ 
}
```
&emsp;&emsp;C++不禁止析构函数向外界抛出异常，但一般来说析构函数不应该向外界函数抛出异常。析构函数中向函数外抛出异常，将直接调用terminator()系统函数终止程序。如果一个析构函数内部抛出了异常，就应该在析构函数的内部捕获并处理该异常，不能让异常被抛出析构函数之外。可以如此处理：
- 若析构函数内部抛出异常，调用std::abort()来终止程序。
- 在析构函数中catch捕获异常并作处理。
关于异常与析构函数的详细论述，可参考Effective C++的条款08：别让异常逃离析构函数。

### 异常处理需注意的问题
- 如果抛出的异常在这一层try...catch...中没有被捕获，则会栈展开（栈解退）逐层往上传递，直至回退到主函数main()都不能处理该异常，则调用系统函数terminate()终止程序。
- 一般在异常抛出后资源可以正常被释放，但注意如果在类的构造函数中抛出异常，系统是不会调用它的析构函数的，处理方法是：在构造函数中抛出异常之前，先删除申请的资源。
- 异常处理仅仅通过类型而不是通过值来匹配的，所以catch块的参数可以没有参数名称，只需要参数类型。
- 函数原型中的异常说明要与实现中的异常说明一致，否则容易引起异常冲突。
- 在利用throw语句抛出异常对象时，throw会先通过拷贝构造函数构造一个新对象，然后将新对象传递给catch参数。如果catch参数采用值传递，会再一次调用拷贝构造函数，给catch参数赋值；如果是传引用（或传地址/指针），则无需赋值处理，这样不仅会提高效率，还可以利用对象的多态性。
- 异常抛出的新对象并非创建在函数栈上，而是创建在专用的异常栈上，因此它才可以跨接多个函数而传递到上层，当异常对象与catch语句成功匹配上后，在该catch语句的结束处被自动析构。所有从try到throw语句之间的局部对象的析构函数将被自动调用。但如果一直上溯到main函数后还没有找到匹配的catch块，那么系统调用terminate()终止整个程序，这种情况下不能保证所有局部对象会被正确地销毁。
- 派生类的异常捕获要放到父类异常扑获的前面，否则，派生类的异常无法被捕获。编写异常说明时，要确保派生类成员函数的异常说明和基类成员函数的异常说明一致，即派生类改写的虚函数的异常说明至少要和对应的基类虚函数的异常说明相同，甚至更加严格，更特殊。
- 在栈展开的过程中，会依次调用局部对象的析构函数释放资源。为了避免内存泄漏的情况，应该采用RAII机制（Resource acquisition is initialization，资源获取即初始化），即以对象管理资源，把资源数据用对象封装起来。程序发生异常，执行栈展开时，封装了资源的对象会自动调用其析构函数以释放资源。C++中的智能指针便符合RAII机制。

### reference
- http://www.cnblogs.com/QG-whz/p/5136883.html
- http://huqunxing.site/2016/09/18/CPP-exception/
- http://baiy.cn/doc/cpp/inside_exception.htm
- http://blog.csdn.net/zhangyifei216/article/details/50410314


