---
title: C++异常机制
date: 2017-10-15
categories: C++
tags: C++
---

&emsp;&emsp;C++异常机制用于处理程序中的异常事件，是一种用来有效处理运行错误的非常强大且灵活的工具。
<!--more-->

### 概述
&emsp;&emsp;C++的异常情况主要分为两种，一种是编译时的语法错误，另一种是运行时异常，例如访问越界，内存不足等。异常机制是专门用于处理运行时异常。异常事件在C++中表示为异常对象。异常事件发生时，程序使用throw关键字抛出异常表达式，抛出点称为异常出现点，由操作系统为程序设置当前异常对象，然后执行程序的当前异常处理代码块，在包含了异常出现点的最内层的try块，依次匹配catch语句中的异常对象。若匹配成功，则执行catch块内的异常处理语句，然后接着执行try...catch...块之后的代码。如果在当前的try...catch...块内找不到匹配该异常对象的catch语句,则由更外层的try...catch...块来处理该异常；如果当前函数内所有的try...catch...块都不能匹配该异常，则递归回退到调用栈的上一层去处理该异常。如果一直退到主函数main()都不能处理该异常，则调用系统函数terminate()终止程序。

### try、throw、catch关键字
&emsp;&emsp;throw与抛出表达式构成了throw语句，throw语句必须包含在try块中，也可以是被包含在调用栈的外层函数的try块中。如果在 try 语句块的程序段中（包括在其中调用的函数）发现了异常，且抛弃了该异常，则这个异常就可以被 try 语句块后的某个catch语句所捕获并处理，捕获和处理的条件是被抛弃的异常的类型与catch语句的异常类型相匹配。由于C++使用数据类型来区分不同的异常，因此在判断异常时，throw语句中的表达式的值就没有实际意义，而表达式的类型就特别重要。
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

异常处理示例：处理除数为0的异常。除数为0的异常可以用try/catch语句来捕获异常，并使用throw语句来抛出异常，从而实现异常处理。
```cpp
#include<iostream>       
#include<cstdlib>
using namespace std;  
 
double func(double x, double y)
{  
    if(y==0)  
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
         cerr<<"error of dividing zero.\n";  
         exit(1);             //异常退出程序  
    }  
    cout << "The result of x / y is : " << res << endl;
}
```

### C++异常标准
&emsp;&emsp;C++标准库提供了一组异常类，用户可以在程序中使用这些标准的异常，也可以通过继承和重载 exception 类来定义新的异常。

异常                        |描述
:---:                       | :---:
std::exception              |该异常是所有标准 C++ 异常的父类。
std::bad_alloc              |该异常可以通过 new 抛出。
std::bad_cast               |该异常可以通过 dynamic_cast 抛出。
std::bad_exception          |这在处理 C++ 程序中无法预期的异常时非常有用。
std::bad_typeid             |该异常可以通过 typeid 抛出。
std::logic_error            |理论上可以通过读取代码来检测到的异常。
std::domain_error           |当使用了一个无效的数学域时，会抛出该异常。
std::invalid_argument       |当使用了无效的参数时，会抛出该异常。
std::length_error           |当创建了太长的 std::string 时，会抛出该异常。
std::out_of_range           |该异常可以通过方法抛出，例如 std::vector 和 std::bitset<>::operator。
std::runtime_error          |理论上不可以通过读取代码来检测到的异常。
std::overflow_error         |当发生数学上溢时，会抛出该异常。
std::range_error            |当尝试存储超出范围的值时，会抛出该异常。
std::underflow_error        |当发生数学下溢时，会抛出该异常。

