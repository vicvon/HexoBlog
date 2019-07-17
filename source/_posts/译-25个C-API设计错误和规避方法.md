---
title: '[译]25个C++ API设计错误和规避方法.md'
date: 2019-07-17 22:38:30
tags:
categories:
---

### [译]25个C++ API设计错误和规避方法

[原文链接](https://www.acodersjourney.com/top-25-cplusplus-api-design-mistakes-and-how-to-avoid-them/)

对于许多C++开发者来说，他们可能把API设计的优先级排到第三或者第四。大多数拥抱C++的开发者都是因为它的原始能力和控制权。因此，性能和优化占据了他们百分之八十的时间。

当然，每一位C++开发者都考虑头文件设计的各个方面，但是API的设计不仅仅只有头文件的设计。事实上，我强烈推荐每一位开发者多思考API的设计，无论是面向公共的还是面向内部的，它都能节约维护成本，提供平滑升级路径并且帮你客户省去麻烦。

以下列出的许多错误都是我的个人经验和我从Martin Reddy的精彩书籍《C ++ API Design》（我强烈推荐的书）中学到的东西的结合。如果你真的想深入了解c++ API设计，你应该读这本书，然后使用下面的列表作为更多的清单来强制执行代码审查。



#### 错误1：没有将API放入命名空间

**为什么这是个错误？**

因为你不知道哪个代码会使用你的API，尤其是外部API。如果你没有把你的API函数放入命名空间，它可能会导致你的API命名和其他API命名冲突。

让我们来看一个简单的API和客户端类的使用

```c++
//API - In Location.h
class vector
{
public:
  vector(double x, double y, double z);
private:
  double xCoordinate;
  double yCoordinate;
  double zCoordinate;
};
//Client Program
#include "stdafx.h"
#include "Location.h"
#include <vector>
using namespace std;
int main()
{
  vector<int> myVector;
  myVector.push_back(99);
  return 0;
}
```

如果有人试图在一个项目中使用这个类并且也使用了`std::vector`，他会得到一个错误`error C2872: ‘vector’: ambiguous symbol`。这是因为编译器不能决定客户端代码中使用哪个`vector`，是`std::vector`还是在`Location.h`中定义的`vector`对象。

**如何解决这个问题？**

总是将你的API放在一个定制的命名空间中

```c++
//API
namespace LocationAPI
{
  class vector
  {
  public:
    vector(double x, double y, double z);
  private:
    double xCoordinate;
    double yCoordinate;
    double zCoordinate;
  };
}
```

另一个选择是在你公共API符号前加一个唯一的前缀。如果遵守这个约定，我们将调用我们的类`Ivector`而不是`vector`。`OpenGL`和`QT`就是使用的这个方法。

在我看来，如果你在开发一个纯C的API，这样做是合理的。它有一个麻烦的事就是确保所有你的公共符号是唯一的。如果你使用的是C++，你应该组织你的API到一个命名空间中，让编译器为你做繁重的工作。

我还强烈鼓励使用嵌套的命名空间进行功能分组或者分离公共API和内部API。一个很好的列子就是BOOST库，它使用了嵌套命名空间。例如，在根boost命名空间内，`boost::variant`包含了公共的BOOST Variant API并且`boost::detail::variant`包含了内部的API。

#### 错误2：在你公共API头全局作用域内包含了using namespace

**为什么这是个错误？**

因为这将导致被引用的命名空间中的所有符号变成全局命名空间中可见，并且抵消了第一点使用命名空间的好处。

另外：

1. 头文件的使用者不可能取消命名空间包含，因此他们被强制接收你命名空间的决定，这是不可取的。
2. 它显著增加了命名冲突的机会，这也是第一点要解决的。
3. 当引入新版本库后，有可能是程序工作版本编译失败。如果新版本引入的名字和程序正在使用的库中的名字冲突就会导致这种情况发生。
4. 在你包含的头文件中using namespace部分代码从引入的那一点开始生效，意味着任何出现在这之前的代码可能区别对待与出现在这之后的代码。

**如何解决这个问题？**

1. 避免使用using namespace声明在你的头文件中。如果你确实需要一些命名空间下的对象，请在你头文件中使用完整名字(如，std::cout)。

```c++
//File:MyHeader.h:
class MyClass
{   
private:
    Microsoft::WRL::ComPtr _parent;
    Microsoft::WRL::ComPtr _child;
}
```

2. 如果上面推荐的第一条建议导致太多的代码，将using namespace的使用限制在头文件中定义的类或者命名空间下。另一点是像下面这样使用域别名。

```c++
//File:MyHeader.h:
class MyClass
{
namespace wrl = Microsoft::WRL; // note the aliasing here !
private:
    wrl::ComPtr _parent;
    wrl::ComPtr _child;
}
```

其他c++头文件问题，请参考[Top 10 C++ header file mistakes and how to fix them](https://www.acodersjourney.com/top-10-c-header-file-mistakes-and-how-to-fix-them/)

#### 错误3：忽略三法则

**什么是三法则**

三法则是，如果一个类定义了析构函数，拷贝构造函数或赋值构造函数，那么应该显示的定义这3个函数，而不是使用他们的默认实现。

**为什么忽略三法则是个错误？**

如果你定义了他们中的一个，可能你的类正管理一个资源(内存，文件句柄，套接字)。从而：

1. 如果你编写/禁用拷贝构造函数或赋值构造函数，你可能需要对另一个做相同的事: 如果一个处理了“特殊的”工作，另一个也可能需要做同样的事情，因为两个函数应该有相似的效果。
2. 如果现实的写了拷贝函数，你可能需要写析构函数：如果在拷贝构造中的“特殊”工作是分配或者复制一些资源（内存，文件，套接字），你需要在析构函数中释放。
3. 如果你显示的写了析构函数，你可能需要显示写或者禁用拷贝：如果你必须写一个nontrivial析构函数，因为你需要手动释放一个对象持有的资源。如果是这样，很可能这些资源要求仔细复制，并且你需要注意对象拷贝和赋值的方法，或者完全禁用拷贝。

下面让我们看一个例子，在下面的API中，我们有一个`int*`的资源被`MyArray`类管理。我们给类创建了一个析构函数，因为当我们销毁对象后我们需要释放`int*`的内存。到目前为主都很好。

现在我们假设你的API客户端如下使用它：

```c++
int main()
{
  int vals[4] = { 1, 2, 3, 4 };
  MyArray a1(4, vals); // Object on stack - will call destructor once out of scope
  MyArray a2(a1); // DANGER !!! - We're copyin the reference to the same object
  return 0;
}
```

**发生了什么？**

客户端在栈上通过构造函数创建了一个类的实例a1。然后通过从a1拷贝创建了实例a2。当a1出了作用域，析构函数将删除类内部的`int*`的内存。但是当a2出作用域后，它调用析构函数并且尝试释放`int*`内存，这导致double free堆被破坏。

因为我们没有提供拷贝构造函数，并且没有标记我们的API是不可拷贝的，没有办法让客户端知道他不应该拷贝`MyArray`对象。

**如何解决这个问题？**

我们可以做以下事情：

1. 提供拷贝构造函数，实现对底层资源的深拷贝
2. 使类成为不可拷贝的，通过delete拷贝构造和赋值构造
3. 最后提供API文档信息。

这是通过提供拷贝和赋值构造修复上面问题的代码：

```c++
// File: RuleOfThree.h
class MyArray
{
private:
  int size;
  int* vals;
public:
  ~MyArray();
  MyArray(int s, int* v);
  MyArray(const MyArray& a); // Copy Constructor
  MyArray& operator=(const MyArray& a); // Copy assignment operator
};
// Copy constructor
MyArray::MyArray(const MyArray &v)
{
  size = v.size;
  vals = new int[v.size];
  std::copy(v.vals, v.vals + size, checked_array_iterator<int*>(vals, size));
}
// Copy Assignment operator
MyArray& MyArray::operator =(const MyArray &v)
{
  if (&v != this)
  {
    size = v.size;
    vals = new int[v.size];
    std::copy(v.vals, v.vals + size, checked_array_iterator<int*>(vals, size));
  }
  return *this;
}
```

第二个方式修复这个问题是使类变成不可拷贝的通过delete拷贝和赋值构造。

```c++
// File: RuleOfThree.h
class MyArray
{
private:
  int size;
  int* vals;
public:
  ~MyArray();
  MyArray(int s, int* v);
  MyArray(const MyArray& a) = delete;
  MyArray& operator=(const MyArray& a) = delete;
};
```

现在当客户端尝试拷贝类，他将收到一个变异错误：`error C2280: ‘MyArray::MyArray(const MyArray &)’: attempting to reference a deleted function`

**C++11 附录**

三法则已经转变为5法则，增加移动构造和移动赋值操作。因此在我们的例子中，如果我们使类成为不可拷贝和不可移动，我们必须将移动构造和移动赋值标记为delete。

```c++
class MyArray
{
private:
  int size;
  int* vals;
public:
  ~MyArray();
  MyArray(int s, int* v);
  //The class is Non-Copyable
  MyArray(const MyArray& a) = delete;
  MyArray& operator=(const MyArray& a) = delete;
  // The class is non-movable
  MyArray(MyArray&& a) = delete;
  MyArray& operator=(MyArray&& a) = delete;
};
```

***附加警告***：如果顶一个一个拷贝构造（包括标记为delete），类没有创建移动构造。如果你的类仅仅包含简单的数据类型，并且你计划使用隐式生成的移动构造，那么如果你定义拷贝构造它将是不可能的。这种情况你必须显示的定义移动构造。

#### 错误4：没有在你的API中标记移动构造和移动赋值为noexcept

一般来说，移动操作不会抛出异常。你基本上是从源对象中窃取一堆指针到目标对象，理论上不应该抛出异常。

**为什么这是个错误？**

如果移动构造不破坏它强大的异常安全保证，一个stl容器再调整大小时仅仅使用移动构造函数。例如，`std::vector`不将使用你API对象的移动构造函数，如果移动构造会抛出异常。这是因为如果在移动操作时抛出异常，那被处理的数据可能会丢失，而一个拷贝构造的原始数据不会被修改。

因此，如果在你API中移动构造和移动赋值没有标记为noexcept，它将可能引起性能问题，如果客户端计划使用stl容器的话。[这篇文章](http://www.hlsl.co.uk/blog/2017/12/1/c-noexcept-and-move-constructors-effect-on-performance-in-stl-containers)展示了一个类不能被移动和可以被移动相比，赋值到vector有2倍的时间消耗，并且经历不可预测的内存波动。

**如何解决这个问题？**

简单的标记移动构造和移动赋值为noexcept

```c++
class Tool
{
public:
  Tool(Tool &&) noexcept;
};
```

#### 错误5：没有将不可抛出异常的API标记为noexcept

**为什么这是一个API设计错误？**

将API标记为noexcept有多重好处，包括编译器优化，例如移动构造的优化。然而，从API设计角度看，如果你的API真的不抛出异常，它将减少你客户端代码的复杂度，因为他们不需要有多个try/catch块在他们代码里。这里还有2个额外的好处：

1. 客户端不需要写单元测试，去测试异常代码路径
2. 客户端软件的代码覆盖率可能变高，因为减少代码复杂度

**如何修复这个问题？**

仅仅标记不抛异常的API为noexcept

#### 错误6：没有标记单个参数的构造函数为explicit

**为什么这是个API设计错误？**

编译被允许做一次隐式转换，将参数解析为函数。这意味着编译器可以使用单参数构造函数转换从一种类型到另一个类型，以获得正确的参数类型。

例如，如果我们在我们的API中有如下的单参数构造函数：

```c++
namespace LocationAPI
{
  class vector
  {
  public:
    vector(double x);
    // .....
  };
}
```

我们能通过下面的代码调用：

```c++
LocationAPI::vector myVect = 21.0;
```

这将调用vector的单参数构造函数通过将21.0转换为参数。然而，这种隐式的行为令人困惑，不直观，并且大多数情况下是意外的。

另一个不需要隐式转换的例子，考虑下面的函数签名：

```c++
void CheckXCoordinate(const LocationAPI::vector &coord, double xCoord);
```

如果没有声明单参数构造函数为explicit，我们能像下面这样调用函数：

```c++
CheckXCoordinate(20.0, 20.0);
```

这将削弱你API的类型安全，因为现在编译器不强制第一个参数为显示的vector对象。

结果是，用户有可能忘记参数的顺序，并且错误的顺序传递他们。

**如何解决这个问题？**

这就是为什么你应该使用explicit关键字修饰单参数构造函数，除非你知道你想支持隐式转换。

```c++
class vector
{
public:
  explicit vector(double x);
  //.....
}
```

#### 错误7：没有标记只读的数据或方法为const

**为什么这是个错误？**

有时，你的API会将你的客户端的一些数据结构作为输入。标记方法和参数为const，这告诉客户端你将以只读的方式使用数据。相反地，如果你的API方法和参数没有被标记为const，你的客户端可能传给你一个数据的拷贝，因为你没有保证不修改数据。根据客户端代码调用API的频率，性能的影响可以从轻微到严重。

**如何解决这个问题？**

当你的API需要以只读的方式使用客户端数据，标记API的方法和参数为const。

我们假设你需要一个函数仅仅检查两个坐标是不是相同：

```C++
//Don't do this:
bool AreCoordinatesSame(vector& vect1, vector& vect2);
```

相反，标记方法为const，这就告诉客户端你不会修改客户端传来的vector对象。

```c++
bool AreCoordinatesSame(vector& vect1, vector& vect2) const;
```

const的正确性是一个大的话题，请参考一本好的c++的书或者读一下[这个文章](https://isocpp.org/wiki/faq/const-correctness)的FAQ章节

#### 错误8：通过const引用返回API内部数据

**为什么这是个错误？**

从表面上看，通过const引用返回一个对象似乎是双赢的，这是因为：

1. 避免了不必要的拷贝
2. 因为是const引用，所以客户端无法修改数据

然而，这可能导致一些严重问题，

1. 如果对象在内部释放了，这时客户端使用的引用怎么办？
2. 客户端抛弃了对象的常量性并且修改它，怎么办？

**如何解决这个问题？**

遵循以下三步：

1. 首先，不要通过更好的设计暴露API内部对象
2. 如果第一条代价太大，考虑通过值传递返回对象
3. 如果它是一个堆上分配的对象，考虑返回一个智能指针，确保即使你释放了也能保证对象可以访问

#### 错误9：当使用隐式模板实例化时，通过模板实现细节混淆公共头文件

在隐式实例化中，你模板的内部代码不得不放入头文件中。没有办法绕开它。然而，你可以分离模板声明（你的API用户将引用的）和模板实例，通过将模板实例放在另一个头文件中。

```c++
// File: Stack.h ( Public interface)
#pragma once
#ifndef STACK_H
#define STACK_H
#include <vector>
template <typename T>
class Stack
{
public:
  void Push(T val);
  T Pop();
  bool IsEmpty() const;
private:
  std::vector<T> mStack;
};
typedef Stack<int> IntStack;
typedef Stack<double> DoubleStack;
typedef Stack<std::string> StringStack;
// isolate all implementation details within a separate header
#include "stack_priv.h"
#endif
```

```c++
// File: Stack_priv.h ( hides implementation details of the Stack class)
#pragma once
#ifndef STACK_PRIV_H
#define STACK_PRIV_H
template <typename T>
void Stack<T>::Push(T val)
{
  mStack.push_back(val);
}
template <typename T>
T Stack<T>::Pop()
{
  if (IsEmpty())
  {
    return T();
  }
  T val = mStack.back();
  mStack.pop_back();
  return val;
}
template <typename T>
bool Stack<T>::IsEmpty() const
{
  return mStack.empty();
}
#endif
```

这个技术被用在许多高质量的基于模板的API中，例如boost。它的好处就是保持主要的公共头文件和实现细节分开。

#### 错误10：当使用场景已知的情况不使用显示模板实例化

**为什么这是个错误？**

从API设计角度，隐式实例化有如下问题：

1. 编译器有责任延迟实例化在正确的位置，并且确保仅仅只有一份代码拷贝，防止符号重复的链接错误。这会浪费你客户端编译和链接的时间。
2. 你内部代码逻辑现在暴露出来了，这不是一个好的主意
3. 客户端可以使用任意的类型实例化你的模板，这些类型都是你没有测试过的，并且会运行失败。

**如何解决这个问题？**

如果你知道你的模板仅仅被用在int，double和string类型，你可以显示实例化生成这3个类型的模板特化。它缩短你客户端编译的时间，和你未经测试的类型分离，保持你模板代码逻辑因此在你的cpp文件中。

要做到这点很简单，只需要三步：

***Step 1:*** 将stack模板实现移到cpp文件中

在这点上，我们尝试实例化并且使用push方法，

```c++
Stack<int> myStack;
myStack.Push(31);
```

我们将得到一个链接错误：

```c++
error LNK2001: unresolved external symbol "public: void __thiscall Stack<int>::Push(int)" (?Push@?$Stack@H@@QAEXH@Z)
```

这个链接错误是告诉我们，它可能没有找到push方法。因为我们没有实例化它。

***Step 2:*** 创建一个模板实例（int，double，string类型）在你cpp文件底部：

```c++
// explicit template instantiations
template class Stack<int>;
template class Stack<double>;
template class Stack<std::string>;
```

现在你可以编译和运行stack代码。

***Step 3:*** 告诉你API客户端你支持3个类型的特化，并且将下面的typedef放在你头文件结尾：

```c++
typedef Stack<int> IntStack;
typedef Stack<double> DoubleStack;
typedef Stack<std::string> StringStack;
```

***警告:*** 如果你已经显示特化了，客户端将无法创建更多的特化（并且编译器也无法创建隐式实例），因为实现细节隐藏在你的cpp文件中。请确保这是你API预期的。

#### 错误11：暴露内部值在默认函数参数中

**为什么这是个问题？**

默认参数常常用来在新版本中扩展API的功能，以便不会破坏API的向后兼容性。

例如，你发布了一个API，

```c++
//Constructor
Circle(double x, double y);
```

后来你决定指定一个radius作为参数很有用。因此你新版本的API使用radius作为第三个参数，然而，你不想破坏已存在的客户端，所以你将radius作为默认参数：

```c++
// New API constructor
Circle(double x, double y, double radius=10.0);
```

用这种方法，任何使用你API传了x和y坐标的客户端都可以使用它。这种方式听起来很好。

然而，它面临了多个问题：

1. 它将破坏二进制兼容性（ABI），因为方法的符号命名被改变
2. 默认值将编译进你的客户端程序中。这意味着你的客户端必须重新编译他们的代码，如果你发布新版本API并且默认值变了
3. 多个默认参数可能导致客户端使用你API造成错误。例如，如果你为所有参数都提供默认参数，客户端可能错误的使用一个组合，这个组合没有一点逻辑关系，像下面这样提供了x的值，没有提供y的值

```c++
Circle(double x=0, double y=0, double radius=10.0);
Circle c2(2.3); // Does it make sense to have an x value without an Y value? May or may not !
```

4. 最后，当你没有显示指定radius的值时，你就暴露了API的行为。这将是糟糕的，因为如果你后期增加对不同单位的支持，让用户在米，厘米，毫米之间切换。这种情况默认的radius的值10.0将不适用所有的单位。

**如何解决这个问题？**

提供多个重载版本而不是使用默认参数，例如

```c++
Circle();
Circle(double x, double y);
Circle(double x, double y, double radius);
```

前两个构造函数的实现可以使用没有指定默认值的属性。重要的是，这些默认值在cpp文件中被指定，并且没有暴露在头文件中。最后，后期的API版本可以改变这些值而对公共的接口没有任何影响。

**补充说明**

1. 不需要将所有的默认参数实例都转换为重载方法。特别地，如果默认参数代表一个非法值或者一个空值，例如定义null作为指针的默认值或者“”作为字符串的值，那么这种用法在API的版本间不可能改变。
2. 作为性能说明，你应该避免定义涉及构造临时对象的默认参数，因为这将通过值传递到方法中，这是有代价的。

#### 错误12：C++API使用#define

#define被用于在C代码中定义常量，如：

```c
#define GRAVITY 9.8f
```

**为什么这是个错误？**

在C++中，你不应该使用#define定义内部常量，有如下几个原因：

1. 使用#define在你公共头文件中将泄漏你的实现细节。
2. #define定义的常量不能进行类型检查，而且容易让我们被隐式转换和舍入错误影响。
3. #define语句是全局的不限制作用域，例如在单个类中。从而他们能污染你客户的全局命名空间。他们将不得不跳过多步通过#undef去除#define。但是找到正确的#undef位置总是很困难，因为有别的依赖。
4. #define没有访问控制。你不能定义#define为public，protected或者private。它总是public的。你不能使用#define指定一个智能被派生类访问的常量。
5. 上面的#define定义的GRVAITY符号会被预处理删除，因此它不会进入符号表。这在调试的时候会有巨大的问题，因为在你客户端使用你API调试代码时会隐藏有价值的信息，因为他们只能在调试器中看到9.8这个值，没有任何描述。

**如何解决这个问题？**

对于简单的常量，使用`static const`代替#define，如

```c++
static const float Gravity;
```

更好的是，如果在编译期知道这个值，使用constexpr

```c++
constexpr double Gravity = 9.81;
```

更多关于const和constexpr请参考[这篇文章](https://stackoverflow.com/questions/13346879/const-vs-constexpr-on-variables)。

在C代码中，有时#define用来定义网络状态：

```c
#define BATCHING 1
#define SENDING 2
#define WAITING 3
```

在c++中，我们可以使用枚举

```c++
enum class NetworkState { Batching, Sending, Waiting };  // enum class
```

#### 错误13：使用友元类

在c++中，友元是你的类授权给另一个类或者方法完全的访问权限的方法。友元类或者友元函数可以访问你的类中的所有protected和private成员。

虽然这是面向对象的设计和封装，这在实践中也很有用。如果你正在开发一个包含多个组件的大型系统，并且想暴露一个组件的功能给选择的客户端，友元会使事情变得简单。

事实上，.NET的[InternalsVisible]属性确实有相似作用。

然而，友元类不应该暴露在公共API中。

**为什么这是个错误？**

因为在公共API中使用友元就意味着允许客户端破坏你的封装，并且非预期的使用系统对象。 

即使我们放弃了内部发现API的一般问题，客户端也可能以非预期的方式使用API，使用他们的系统然后致电你的支持团队修复阿门以非预期方式使用API的问题。

***这是他们的问题吗？不！***这是你的问题，是你暴露了友元类。

**如何解决这个问题？**

避免在公共API中使用友元。它们通常是设计不佳的表现，并且允许客户端访问API的所有受保护和私有成员。

#### 错误14：没有避免不必要的头文件包含

**为什么这是个问题？**

不必要的头文件会增加编译时间。这不仅仅是浪费使用你API编译代码的开发人员时间，而且会因为自动化构建周期增加而增加成本，这可能需要每天构建几千遍。

另外，有庞大头文件将降低并行构建效率如incredibuild和fastbuild

**如何解决这个问题？**

1. 你的API应该只包含编译需要的头文件。使用前置声明是有用的，因为它减少了编译时间，能打破头文件的循环依赖。
2. 使用预编译头也能有效减少编译时间。

#### 错误15：使用前置声明声明外部类型(非你的)

**为什么这是个问题？**

使用前置声明声明不是你的对象会以不可预测的方式破坏客户端代码。例如，如果客户端决定使用一个不同版本的外部API头文件，那么如果你前置声明的外部对象被改变为typedef或者模板类，你的前置声明就被破坏了。

从另一个角度看，如果你从外部头文件前置声明一个类，你就锁定了你客户端使用外部头文件的版本，必须和你使用版本一致，他再也不能升级那个外部依赖了。

**如何解决这个问题？**

你应该只前置声明你自己API内的符号。也不要前置声明stl类型。[请看关于这个问题的延伸讨论](https://stackoverflow.com/questions/47801590/what-are-the-risks-to-massively-forward-declaration-classes-in-header-files)。

#### 错误16：头文件无法自编译

一个头文件应该有编译它自己的所有东西，它应该显示#include或者前置声明一个它编译需要的类型。

如果头文件没有它编译需要的所有内容，但是包含头文件的程序编译表明，由于包含顺序依赖，头文件以某种方式获得了它所需要的东西。这通常是因为另一个头文件在编译链之前，它提供了这个头文件缺失的内容。

如果包含顺序依赖关系改变，整个程序可能被破坏。C++编译器因为误导错误信息而臭名昭著，它可能不好定位到问题。

**如何解决这个问题？**

检查你的头文件通过一个隔离的方式编译他们，建立一个cpp文件仅仅包含你的头文件。如果它产生编译错误，那么需要包含头文件或者前置声明。对项目中的所有头文件以自底向上的方式重复。随着代码库变大和代码块的移动，这将有助于防止随机构建的中断。

#### 错误17：没有提供你API的版本信息

客户端应该能在编译时和运行时检查集成到他们系统中的API版本。如果信息缺失，他们将无法采取有效错误升级或者打补丁。

他们也很困难在不同平台上增加向后兼容性。

此外，产品的版本号是我们的工程师在回答用户问题时首先要问的。

#### 错误18：从开始就没有决定静态还是动态库实现

[你的客户端喜欢使用静态库还是动态库都应该决定你设计的选择](https://www.acodersjourney.com/cplusplus-static-vs-dynamic-libraries/)。例如：

1. 你可以在你的API接口中使用stl类型吗？如果你以静态库的方式发布可能会好点，但是如果是以动态库的方式，可能因为平台类型和编译器版本引起二进制问题。如果是DLL，可能喜欢扁平的C风格API。
2. 有多少功能会进入你的API？对于静态库，你不需要担心太多，因为仅仅只需要库文件被链接进可执行程序中。另一方面，对于DLL，尽管只有5%的功能被使用，整个DLL都被加载到低效的进程空间。如果你使用DLL，把功能分解到多个DLL中可能更好（例如一个math库，你可以从三角函数库中分离出微积分库）。

**如何解决这个问题？**

没有什么神奇之处，它归结起来就是清楚地收集旧需求，在早期讨论阶段确认你的客户需要静态库还是动态库。

#### 错误19：没有意识到ABI兼容性

根据维基百科对ABI的定义，ABI是两个二进制程序模块之间的接口。通常，其中一个模块是库或者操作系统的工具，另一个是用户执行的程序。

如果一个链接旧版库的程序链接了新版本库能继续运行并且不需要重新编译，则这个库是二进制兼容的。

二进制兼容可以节省很多麻烦。它使在特定平台上发布软件更容易。如果没有确保发布版本的二进制兼容性，人们将强制提供静态链接文件。静态二进制文件很糟糕，因为它们浪费资源(尤其是内存)不允许从库的bug修复和扩展中获益。windows子系统被打包成DLL集合是有原因的，这使得windows更新容易，可能不是真的，而是有别的原因。

例如，这是2个不同函数的name mangling(符号名被用来标识对象和库文件):

```c++
// version 1.0
void SetAudio(IAudio *audioStream) //[Name Mangling] ->_Z8SetAudioP5Audio
// version 1.1
void SetAudio(IAudio *audioStream, bool high_frequency = false) // [Name Mangling] ->_Z8SetAudioP5Audiob
```

这两个方法是源代码兼容的，但是不是二进制兼容的，他们的name mangling是不同的。这意味着1.1版本不能简单的替换1.0版本，因为1.0版本的name mangling没有被定义。

**如何解决ABI问题？**

首先，[熟悉ABI兼容性和破坏ABI的改变](https://www.acodersjourney.com/20-abi-breaking-changes/)。然后，按照Martin Reddy在他的书中提供的附加指导：

1. 使用原始C风格的API可以很简单的保证ABI兼容性，因为C语言没有继承，可选参数，重载，异常和模板这些特性。例如，使用`std::string`可能在不同编译器间没有ABI兼容。为了更好的利用这两个方面，你可以使用C++面向对象的方式开发API，然后提供一个用C语言包装C++的API。
2. 如果你确实有ABI不兼容的更改，你可以考虑在新的库中使用不同的name mangling以便不打破存在应用的ABI兼容性。这种方法被libz使用。在windows上1.1.4版本之前命名为ZLIB.DLL。然而，二进制不兼容的设置被用在之后的版本，所以库被命名为ZLIB1.DLL，这里的“1”显示了API的主版本号。
3. pimpl idom能保留你接口的二进制兼容，因为它移动所有在将来可能有修改的实现细节到cpp文件中，这些都不会影响h文件。
4. 你可以定义一个重载版本代替在现有版本上加参数。这保证了原来的符号继续有效，并且提供了一个新的。在你的cpp文件内，老版本可以通过调用新的重载版本实现。

#### 错误20：在已经发布的API类中添加纯虚方法

**为什么这是个错误？**

看如下代码：

```c++
class SubClassMe
{
  public:
    virtual ~SubClassMe();
    virtual void ExistingCall() = 0;
    virtual void NewCall() = 0; // added in new release of API
};
```

这个API让所有现存的客户端都改变了，因为现在他们必须实现这个新添加的方法，否则他们的派生类将编译失败。

**如何解决这个问题？**

简单修复就是在抽象类中新添加的方法都实现一个默认实现，保证他们是虚的而不是纯虚的。

```c++
class SubClassMe
{
  public:
    virtual ~SubClassMe();
    virtual void ExistingCall() = 0;
    virtual void NewCall(); // added in new release of API
};
```

#### 错误21：没有文档说明是同步还是异步

看如下的头文件中的代码：

```c++
static void ExecuteRequest(CallRequestContainer& reqContainer);
```

当我看到它的时候，我完全不知道这个方法是同步的还是异步的。这非常影响我怎么使用和如何使用这个代码。例如，如果这是一个同步调用，我绝对不会在像游戏场景渲染这样关键代码路径使用它。

**如何解决这个问题？**

这里有一些帮助：

1. 使用C++11的特征，如`future`作为返回值，就显示这是一个异步方法：

```c++
std::future<StatusCode> ExecuteRequest(CallRequestContainer& reqContainer);
```

2. 方法名上增加Sync或者Async标识

```c++
static void ExecuteRequestAsync(CallRequestContainer& reqContainer);
```

3. 在方法上有注释表明它是同步的还是异步的。

#### 错误22：没有使用平台/编译器最低的公共支持

你应该了解你的客户使用哪个版本的编译器或者C++标准。例如，如果你了解到你的许多客户都是使用的C++11，你就不应该使用C++14的特性。

我们最近收到了客户提交的支持请求，他们使用的老版本的visual studio并且c++14的`make_unique`不可用。我们不得不使用条件编译修复这个问题，幸运的是这只有很少的地方使用。

#### 错误23：不考虑开源项目的header only实现

如果你以源码发布你的API，请考虑使用header only。

使用header only发布有如下好处：

1. 你不必担心在不同平台发布.lib和.dll/.so 文件，也不用担心编译器版本问题。这减少了你构建和发布的逻辑。
2. 你的客户可以访问你的所有源码。
3. 你的客户节省了编译你二进制的步骤，并且确保和他的exe相同的设置。
4. 你的客户节省了打包你二进制包的时间。打包二进制会非常的麻烦，例如游戏引擎。
5. 有些情况header only是唯一的选择，例如处理模板(除非你选择显示特化)

这是被许多开源项目使用的非常流行的模式，包括boost和rapidjson。

#### 错误24：参数类型不一致

这是最近我们审核继承的历史代码出现的问题。

头文件有如下定义：

```c++
typedef Stack<int> IntStack;
typedef Stack<double> DoubleStack;
typedef Stack<std::string> StringStack;
```

在代码库中分散了一些没有显示使用`typedef`和`Stack<T>`的代码。一个公共方法，有如下声明：

```c++
void CheckStackFidelity(IntStack testIntStack, Stack<std::string> testStringStack);
```

**如何解决这个问题？**

选择`typedef`或者非`typedef`没有关系。关键是“stay consistent”保持一致。

#### 错误25：没有API review流程

在开发阶段，我常常看到没有及早的进行API review。这是因为没有任何结构化的制定去执行API review。

我发现当没有流程的时候会有多重问题：

1. API不符合Beta客户的使用情景。
2. API和系统或相同产品的其他部分不相似。
3. API有法律/合规/营销问题。我们遇到过这样一种情况：其中一个API的命名不是很合适。

市场需要它，它导致了很多后期重构和延迟。

**如何解决这个问题？**

为了避免上面指出的几种麻烦，你应该建立一个至少执行以下操作的过程：

1. API设计应该先于编码。在C ++上下文中，这通常是带有相关用户文档的头文件。
2. 所有的相关人员都应该review API，包括合作伙伴团队、Beta（私人预览客户）、营销、法律和开发人员（如果贵公司有）。
3. 在私人预览前几个月与＃2中的所有利益相关者进行另一次API审核，以确保他们满意。
4. 明确告知，任何API私有预览更改都是有代价的，人们应该在开发的早期阶段提出他们的建议。



好吧，这些就是我注意到的C++API设计的25个错误。这份清单并不全面，所以你一定要拿一本Martin Reddy的书来深入了解这个主题。