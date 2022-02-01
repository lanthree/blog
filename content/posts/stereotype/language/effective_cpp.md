---
title: "Effective C++"
date: 2022-02-02T21:49:07+08:00
draft: false
categories: [八股文,编程语言]
---

## 条款01：视C++为一个语言联邦

+ C：C++以C为基础，区块（blocks）、语句（statements）、预处理器（preprocessor）、内置数据类型（built-in data types）、数据（arrays）、指针（pointers）等统统来自C。
+ Object-Oriented C++：这部分也就是C with Classes所诉求的，classes（包括构造函数和析构函数），封装（encapsulation）、继承（inheritance）、多态（polymorphism）、virtual函数（动态绑定）……等等。
+ Template C++：这是C++的泛型编程（generic programming）部分，它们带来崭新的编程范型（prpgramming paradigm），也就是所谓的template metaprogramming（TMP，模版元编程）。
+ STL：STL是个template程序库。

请记住：
+ C++高效编程守则视状况而变化，取决于你使用C++的哪一部分。

## 条款02：尽量以const，enum，inline替换#define

以常量替换`#define`：
1. 定义常量指针（constant pointers），如果要在头文件内定义一个常量的（不变的）`char*-bases`字符串，必须写`const`两次：
    + `const char* const authorName = "Scott Meyers";`
    + 或者写成`const std::string authorName("Scott Meyers");`
2. class专属常量
    + ```cpp
      // static class 常量声明位于头文件内
      class CostEstimate {
       private:
        static const double FudgeFactor;
        ...
      };

      // static class常量定义位于实现文件内
      const double CostEstimate::FudgeFactor = 1.35;
      ```

enum hack：一个属于枚举类型（enumerated type）的数值可权充ints被使用

```cpp
class GamePlayer {
 private:
  enum {NumTurns = 5};

  int scores[NumTurns];
  ...
};
```

宏函数可以用template inline函数替代：
```cpp
#define CALL_WITH_MAX(a, b) f((a) > (b) ? (a) : (b))

// 但会有一些不符合预期的事情
int a = 5, b = 0;
CALL_WITH_MAX(++a, b);    // a被累加两次
CALL_WITH_MAX(++a, b+10); // a被累加一次

// 替代函数
template<typename T>
inline void callWithMax(const T& a, const T& b) {
  f(a > b ? a : b);
}
```

请记住：
+ 对于单纯常量，最好以const对象或enums替换`#define`。
+ 对于形似函数的宏（macros），最好改用inline函数替换

## 条款03：尽可能使用const

如果关键字const出现在星号的左边，表示被指物是常量；如果出现在星号右边，表示指针自身是常量；如果出现在星号两边，表示被指物和指针两者都是常量。

const迭代器：
```cpp
std::vector<int> vec;

// iter的作用像个T* const
const std::vector<int>::iterator iter = vec.begin();
*iter = 10;  // 没问题
++iter;      // 不可以

// cIter的作用像个const T*
std::vector<int>::const_iterator cIter = vec.begin();
*cIter = 10; // 不可以
++cIter;     // 没问题
```

在一个函数声明式内，const可以和函数返回值、各参数、函数自身（（如果是成员函数）产生关联。

成员函数如果是const意味着什么？这有两个流行概念：bitwise constness（又称physical constness）和logical constness。

前者是说它不更改对象内的任何一个bit，也是C++对常量性（constness）的定义，即const成员函数不可以更改对象内任何non-static成员变量。但如果该变量是指针，指针指向的内容不在约束之内。

后者主张，一个const成员函数可以修改它所处理的对象内的某些bits，但只有在客户端侦测不出的情况下才得如此，即保证逻辑上的常量性 但实际可以修改某些bits（通过`mutable`释放掉non-static成员变量的bitwise constness约束）：

```cpp
class CTextBlock {
 public:
  ...
  std::size_t length() const;
 private:
  char* pText;
  mutable std::size_t textLength;
  mutable bool lengthIsValid;
};

std::size_t CTextBlock::length() const {
  if (!lengthIsValid) {
    textLength = std::strlen(pText);
    lengthIsValid = true;
  }
  return textLength;
}
```

如果你需要实现const和non-const成员函数，那么可能会有大段的重复代码（只有函数和返回值的const修饰不同），如果代码确实一样，可以考虑用const版本实现non-const版本：

```cpp
class TextBlock {
 public:
  ...
  const char& operator[](std::size_t p) const {
    ...
    return text[p];
  }

  char& operator[](std::size_t p) {
    // 将op[]返回值的const移除
    return const_cast<char&> (
      // 为*this加上const调用const op[]
      static_cast<const TextBlock&>(*this)[p]
    );
  }
  ...
};
```

反过来，令const版本调用non-const版本以避免重复是一种错误行为。

请记住：

+ 将某些东西声明为const可帮助编译器侦测出错误用法。const可被施加于任何作用域内的对象、函数参数、函数返回类型、成员函数本体。
+ 编译器强制实施bitwise constness，但你编写程序时应该使用“概念上的常量性”（conceptual constness）。
+ 当const和non-const成员函数有着实质等价的实现时，令non-const版本调用const版本可避免代码重复。

## 条款04：确定对象被使用前已被初始化

永远在适用对象之前先将它初始化，处于无任何成员的内置类型，你必须手工完成此事。至于内置类型意外的任何其他东西，初始化责任落在构造函数（constructors）身上。

C++有着十分固定的“成员初始化次序”，次序总是相同：base classes更遭遇其dervied class被初始化，而class的成员变量总是以其声明次序被初始化。

不同编译器单元内定义之non-local static对象的初始化次序：

所谓static对象，其寿命从被构造出来知道程序结束为止，包括global对象、定义于namespace作用域内的对象、在classes内、在函数内、以及在file作用域内被声明为static的对象。函数内的static对象称为non-local static对象；其他static对象称为non-local static对象。程序结束时static对象会被自动销毁，也就是他们的析构函数会在`main()`结束时被自动调用。

所谓编译单元（translation unit）是指产出单一目标文件（single object file）的那些源码。

如果某编译单元内的某个non-local staic对象的初始化动作使用了另一个编译单元内的某个non-local static对象，它所用到的这个对象可能尚未被初始化，因为C++对“定义于不同编译单元内的non-local static对象”的初始化次序并无明确定义。

解决方案：将每个non-local static对象搬到自己的专属函数内（该对象在此函数内被声明为static）（refrence-returning）。C++保证，函数内的local static对象会在“该函数被调用期间”“首次遇上该对象之定义式”时被初始化。

```cpp
class FileSystem { ... };
// 这个函数用来替换全局tfs对象
FileSystem &tfs() {
  static FileSystem fs;
  return fs;
}

class Directory { ... };
Directory::Director(params) {
  ...
  std::size_t disks = tfs().numDisks();
  ...
}
// 这个函数用来替换全局tempDir对象
Directory& tempDir() {
  static Directory td;
  return td;
}
```
为了消除与初始化有关的“竞速形势”（race conditions），可在程序的单线程启动阶段（single-threaded startup portion）手工调用所有reference-returning函数。

请记住：
+ 为内置对象进行手工初始化，因为C++不保证初始化它们。
+ 构造函数最好使用成员初值列（member intialization list），而不要在构造函数本体内使用赋值操作（assignment）。初值列列出的成员变量，其排列次序应该和它们在class中的声明次序相同。
+ 为免除“跨编译单元之初始化次序”问题，请以local static对象（reference-returning）替换non-local static对象。

## 条款05：了解C++默默编写并调用哪些函数

如果你自己没声明，编译器就会为class声明一个copy构造函数、一个copy assignment操作符和一个析构函数。此外，如果你没有声明任何构造函数，编译器也会为你声明一个default构造函数。左右这些函数都是public且inline：

```cpp
class Empty {};
// 等价于
class Empty {
 public:
  Empty() {...}  // defualt构造函数
  Empty(const Empty& rhs) {...} // copy构造函数
  ~Empty() {...} // 析构函数

  Empty& operator=(const Empty& rhs) {...} // copy assignment操作符
};
```
> 唯有当这些函数被需要（被调用），它们才会被编译器创建出来。

注意，如果你打算在一个“内含reference成员”/“内含const成员”的class内置支持赋值操作（assignment），你必须自己定义copy assignment操作符。另外，如果某个base classes将copy assignment操作符声明为private，编译器将拒绝为其derived classes生成一个copy assignment操作符。

请记住：
+ 编译器可以暗自为class创建default构造函数、copy构造函数、copy assignment操作符、以及析构函数。

## 条款06：若不想使用编译器自动生成的函数，就该明确拒绝

请记住：
+ 为驳回编译器自动（暗自）提供的机能，可将相应的成员函数声明为private并且不予实现（c++11开始还可以使用delete修饰）。使用像Uncopyable这样的base class也是一种做法。

## 条款07：为多态基类声明virtual析构函数

请记住：
+ polymorphic（带有多态性质的）base classes应该声明一个virtual析构函数。如果class带有任何virtual函数，他就应该拥有一个virtual析构函数。
+ classes的设计目的如果不作为base classes使用，或不是为了具备多态性（polymorphically），就不该声明virtual析构函数。

## 条款08：别让异常逃离析构函数

C++并不禁止析构函数吐出异常，但它不鼓励你这样做。如果在析构函数中的确有操作可能抛出异常，可以有两种方法避免：

第一种方法，在析构函数内消化掉异常：

```cpp
class DBConn {
 public:
  ...
  ~DBConn() {
    try { db.close(); }
    catch (...) {
      // abort程序：std::abort()
      // 或者 忽略
    }
  }
 private:
  DBConnection db;
};
```

另外一种方法是提供一个新函数，供调用方可以处理异常：

```cpp
class DBConn {
 public:
  ...
  void close() { // 提供的新函数
    db.close();
    closed = true;
  }
  ~DBConn() {
    if (closed) return;

    try { db.close(); }
    catch (...) {
      ...
    }
  }
 private:
  DBConnection db;
  bool closed;
};
```

请记住：
+ 析构函数绝对不要吐出异常。如果一个被析构函数调用的函数可能抛出异常，析构函数应该捕捉任何异常，然后吞下它们（不传播）或结束程序。
+ 如果客户需要对某个操作函数运行期间抛出的异常做出反应，那么class应该提供一个普通函数（而非在析构函数中）执行该操作。

## 条款09：绝不在构造和析构过程中调用virtual函数

请记住：
+ 在构造和析构期间不要调用virtual函数，因为这类调用从不下降至derived class（比起当前执行构造函数和析构函数的那层）。

## 条款10：令operator=返回一个reference to *this

为了实现“连锁赋值”，赋值操作符必须返回一个reference指向操作符的左侧实参：

```cpp
class Widget {
 public:
  Widget& operator+=(const Widget& rhs) {
    ...
    return *this;
  }
  Widget& operator=(const Widget& rhs) {
    ...
    return *this;
  }
  ...
};
```

请记住：
+ 令赋值（assignment）操作符返回一个reference to *this。

## 条款11：在operator=中处理“自我赋值”

证同测试（identify test）：

```cpp
Widget& Widget::operator=(const Widget& rhs) {
  if (this == &rhs) return *this;

  delete pb; // Bitmap* Widget::pb;
  pb = new Bitmap(*rhs.pb);
  return *this;
}
```

为了防止在`new`出现异常导致Widget不可用，可以先不删除：

```cpp
Widget& Widget::operator=(const Widget& rhs) {
  Bitmap* pOrig = pb;
  pb = new Bitmap(*rhs.pb);
  delete pOrig;
  return *this;
}
```

另一个替代方案是`copy-and-swap`：

```cpp
void Widget::swap(Widget& rhs) {
  ...
}
Widget& Widget::operator=(const Widget& rhs) {
  Widget tmp(rhs);
  swap(temp);
  return *this;
}
```

考虑到，如果copy assignment操作符声明的参数如果不是引用，那么以by value方式传递时就会创造一个副本，可以直接拿来swap：

```cpp
void Widget::swap(Widget& rhs) {
  ...
}
Widget& Widget::operator=(const Widget rhs) {
  swap(rhs);
  return *this;
}
```

请记住：
+ 确保当对象自我赋值时 operator= 有良好行为。其中技术包括 比较“来源对象”和“目标对象”的地址、精心周到的语句顺序、以及`copy-and-swap`。
+ 确定任何函数如果操作一个以上的对象，而其中多个对象是同一个对象时，其行为仍然正确。

## 条款12：复制对象时勿忘其中每一个成分

任何时候，只要你承担起“为derived class撰写copying函数”（copy构造函数、copy assignment操作符）的重责大任，必须很小心地也复制其base class成分：

```cpp
Dervied::Dervied(const Dervied& rhs)
  : Base(rhs), dervied_v(rhs.dervied_v)
{}

Dervied& Dervied::operator=(const Dervied& rhs) {
  Base::operator=(rhs);
  dervied_v = rhs.dervied_v;
  return *this;
}
```

请记住：
+ copying函数应该确保复制“对象内的所有成员变量”及“所有base class成分”。
+ 不要尝试以某个copying函数实现另一个copying函数。应该将共同机能放进第三个函数中，并由两个copying函数共同调用。

## 条款13：以对象管理资源

+ 获得资源后立刻放进管理对象（managing object）内。“以对象管理资源”的观念常被称为“资源取得时机便是初始化时机”（Resource Acquisition Is Initialization；RAII）。
+ 管理对象（managing object）运用析构函数确保资源被释放。

请记住：
+ 为防止资源泄漏，请使用RAII对象，它们在构造函数中获得资源并在析构函数中释放资源。

## 条款14：在资源管理类中小心copying行为

假设你自己为mutex实现了一个RAII类：

```cpp
class Lock {
 public:
  explicit Lock(Mutex* pm) : mutexPtr(pm)
  {
    lock(mutexPtr);
  }
  ~Lock() { unlock(mutexPtr); }
};
```

如果Lock对象发生复制，默认的行为很槽糕。大多数时候你会选择一下两种可能：

+ 禁止复制。
+ 对底层资源祭出“引用计数法”（reference-count）。

请记住：
+ 复制RAII对象，必须一并复制它所管理的资源，所以资源的copying行为决定RAII对象的copying行为
+ 普遍而常见的RAII class copying行为是：抑制copying、施行引用计数法（reference counting）。不过其他行为（复制底部资源、转移底部资源的所有权）也都可能被实现。

## 条款15：在资源管理类中提供对原始资源的访问

显示转换：提供`get()`函数，拿到底层资源

隐士转换：提供隐士转换函数

```cpp
class Font {
public:
 ...
 operator FontHandle() const // 隐士转换函数
 {return f;}
 ...
private:
 FontHandle f;
};

Font f1(getFount()); // RAII
FontHandle f2 = f1;  // f1隐士转换为其底层的FontHandle然后才复制它
```

请记住：
+ APIs往往要求访问原始资源（raw resources），所以每一个RAII class应该提供一个“取得其所管理之资源”的办法。
+ 对原始资源的访问可能经由 显式转换 或 隐式转换。一般而言显式转换比较安全，但隐式转换对客户比较方便。

## 条款16：成对使用new和delete时要采取相同的形势

当使用new（也就是通过new动态生成一个对象），有两件事发生。第一，内存被分配出来（通过名为`operator new`的函数）。第二，针对此内存会有一个（或更多）构造函数被调用。

当使用delete，也有两件事发生：针对此内存会有一个（或更多）析构函数被调用，然后内存才被释放（通过名为`operator delete`的函数）。

请记住：
+ 如果你在new表达式中使用`[]`，必须在相应的delete表达式中也使用`[]`；如果不在new表达式中不实用`[]`，一定不要在相应的delete表达式中使用`[]`。

## 条款17：以独立语句将newed对象置入智能指针

假设有如下代码

```cpp
int priority();
void process(std::shared_ptr<Widget> pw, int priority);

process(std::shared_ptr<Widget>(new Widget), priority());
```

上述代码可能以以下顺序执行（编译优化导致顺序不定）：
1. 执行`new Widget`
2. 调用`priority()`
3. 调用`std::shared_ptr`构造函数

若在`priority()`发生异常，那么将会造成资源泄漏。避免这类问题的方法，是使用分离语句：

```cpp
std::shared_ptr<Widget> pw(new Widget);
process(pw, priority());
```

请记住：
+ 以独立语句将newed对象存储于（置入）智能指针内。如果不这样做，一旦异常被抛出，有可能导致难以察觉的资源泄漏。

## 条款18：让接口容易被正确使用，不易被误用

理想上，如果客户企图使用某个接口而却没有获得他所预期的行为，这个代码不应该通过编译；如果代码通过了编译，它的作为就该是客户所想要的。

请记住：
1. 好的接口很容易被正确使用，不容易被误用。你应该在你的所有接口中努力达成这些性质。
2. “促进正确使用”的办法包括接口的一致性，以及与内置类型的行为兼容。
3. “阻止误用”的办法包括建立新类型、限制类型上的操作，束缚对象值，以及消除客户的资源管理责任。
4. `std::shared_ptr`支持定制型删除器（custom deleter）。这可防范DLL问题，可被用来自动解除互斥锁（mutexes）等等。

## 条款19：设计class犹如设计type

你应该带着和“语言设计者当初设计语言内置类型时”一样的谨慎来研讨class的设计。

设计高效的class需要面对的问题：
+ 新type的对象应该如何被创建和销毁？
+ 对象的初始化和对象的赋值该有什么样的差别？
+ 新type的对象如果被`pass-by-value`以值传递），意味着什么？
    + copy构造函数用来定一个type的`pass-by-value`如何实现
+ 什么是新type的“合法值”？
+ 你的新type需要配合某个继承图系（inheritance graph）吗？
+ 你的新type需要什么样的转换？
+ 什么样的操作符和函数对此新type而言是合理的？
+ 什么样的标准函数应该驳回？（= delete）
+ 谁该取用新type的成员？
+ 什么是新type的“未声明接口”（undeclared interface）？
+ 你的新type有多么一般化？（class template？）
+ 你真的需要一个新type吗？

请记住：
+ class的设计就是type的设计。在定义一个新type之前，请确定你已经考虑过本条款覆盖的所有讨论主题。

## 条款20：宁以pass-by-reference-to-const替换pass-by-value

请记住：
+ 尽量以`pass-by-reference-to-const`替换`pass-by-value`。前者通常比较高效，并可避免切割问题（slicing problem）。
+ 以上规则并不设用于内置类型，以及STL的迭代器和函数对象。对它们而言，`pass-by-value`往往比较何时。

## 条款21：必须返回对象时，别妄想返回其reference

当你必须在“返回一个reference和返回一个object”之间抉择时，你的工作就是挑出行为正确的那个。

请记住：
+ 绝不要返回pointer或reference指向一个local stack对象，或返回reference指向一个heap-allocated对象，或返回pointer或reference指向一个local static对象而有可能同时需要多个这样的对象。

## 条款22：将成员变量声明为private

请记住：
+ 切记将成员变量声明为private。这可赋予客户访问数据的一致性、可细微划分访问控制、允许约束条件获得保证，并提供class作者以充分的实现弹性。
+ protected并不比public更具封装行。

## 条款23：宁以non-member、non-friend替换member函数

愈多东西被封装，我们改变那些东西的能力也就愈大。这就是我们首先推崇封装的原因：它使我们能够改变事物而只影响有限的客户。

考虑对象内的数据，愈少代码可以看到数据（也就是访问它），愈多的数据可被封装，而我们也就愈能自由地改变对象数据。

请记住：
+ 宁可拿non-member non-friend函数替换member函数。这样做可以增加封装性、包裹弹性（packaging flexibility）和机能扩充性。

## 条款24：若所有参数接续类型转换，请为此采用non-member函数

假设有类`Rational`：

```cpp
class Rational {
 public:
  // 可以不为explicit：允许int-to-Rational隐式转换
  Rational(int numerator = 0,
           int demoninator = 1);
  int numerator() const;
  int demoninator() const;
 private:
  ...
};
```

此时想支持乘法操作，加入该操作定义放到函数内：

```cpp
class Rational {
 public:
  ...
  const Rational operator* (const Rational& rhs) const;
};
```

现在可以做Rational与Rational的乘法，假设还想做int与Rational的乘法：

```cpp
Rational oneHalf(1, 2);
Rational result_1 = oneHalf * 2; // 可以，2隐式转换
Rational result_2 = 2 * oneHalf; // 不可以
```
`2 * oneHalf`尝试找`2.operator*(oneHalf)`与全局的`operator*(2, oneHalf)`但是都没有。那么，一个支持混合运算的方法，就是声明为全局操作函数：

```cpp
class Rational {
 public:
  ...
};

const Rational operator* (
    const Rational& lhs, const Rational& rhs) {
  return Rational(lhs.numerator() * rhs.numerator(),
                  lhs.demoninator() * rhs.demoninator());
}
```

> 当从Object-Oriented C++转进入Template C++并让Rational成为一个class template而非class，又有一些需要考虑的新争议、新接发。

请记住：
+ 如果你需要为某个函数的所有参数（包括被this指针所指的那个隐于参数）进行类型转换，那么这函数必须是个non-member。

## 条款25：考虑写出一个不抛异常的swap函数

所谓swap（置换）两个对象值，意思是将两对象的值彼此赋予多方。缺省情况下swap动作可由标准程序提供的swap算法完成。其典型实现完全如你所预期：

```cpp
namespace std {
template<typename T>
void swap(T& a, T& b) {
  T temp(a);
  a = b;
  b = temp;
}
}
```

只要类型T支持copying，缺省的swap实现代码就会帮你置换类行为T的对象，你不需要为此另外再做任何工作。

假设如果需要swap一个pimpl（pointer to implementation）的类对象：

```cpp
class WidgetImpl {
 public:
  ...
 private:
  int a, b, c;           // 可能有许多数据
  std::vector<double> v; // 意味复制时间很长
};

class Widget { // 这个class使用pimpl手法
 public:
  Widget(const Widget& rhs);
  // 复制Widget时，令它复制其WidgetImpl对象
  Widget& operator=(const Widget& rhs) {
    ...
    *pImpl = *(rhs.pImpl);
    ...
  }
  ...
 private:
  WidgetImpl* pImpl;
};
```

理论上，如果要swap这两个，只需要置换其中的指针即可，但上面的的swap函数会连`WidgetImpl`也做置换（因为Widget的实现）。此时，我们可以特化`std::swap`：

```cpp
class Widget {
 public:
  ...
  void swap(Widget& other) {
    std::swap(pImpl, other.pImpl);
  }
  ...
};

namespace std {
template<> // 表示全特化total template specialization
// 特化std::swap
void swap<Widget>(Widget& a, Widget& b) {
  a.swap(b);
}
}
```

如果Widget、WidgetImpl是class template的话，因为C++不允许偏特化（partially specialize）一个function template（std::swap）。重载可以解决这种问题，但是重载std函数，不可行。

可以在私有的命名空间内定义一个swap函数：

```
namespace WidgetStuff {
template<typename T>
class Widget {...};
...

// 这里不属于std命名空间
template<typename T>
void swap(Widget<T>& a, Widget<T>& b) {
  s.swap(b);
}
}
```

C++的名称查找法则（name lookup rules）确保将找到global作用域或`Widget`所在之命名空间内的任何Widget专属的swap，并使用“实参取决之查找规则”（argument-dependent lookup）找出`WidgetStuff`内的swap。

总结之，如果std::swap的缺省实现，效率可接受，你不需要额外做任何事。如果缺省实现的版本效率不足，试着做以下事情：

1. 提供一个public swap成员函数，让它高效地置换你的类型的两个对象值。这个函数绝不该抛出异常。
2. 在你的class或template所在的命名空间内提供一个non-member swap，并令它调用上述swap成员函数。
3. 如果你正编写一个class（而非class template），为你的class特化std::swap。并令它调用你的swap成员函数

请记住：
+ 当`std::swap`对你的类型效率不高时，提供一个swap成员函数，并确定这个函数不抛出异常。
+ 如果你提供一个member swap，也该提供一个non-member swap用来调用前者。对于classes（而非templates），也请特化`std::swap`。
+ 调用swap时应针对`std::swap`使用using声明式，然后调用swap并且不带任何“命名空间资格修饰”。
+ 为“用户定义类型”进行std templates全特化是好的，但千万不要尝试在std内加入某些对std而言全新的东西。

## 条款26：尽可能延后变量定义式的出现时间

只要你定一个变量而其类型带有一个构造函数或析构函数，那么当程序的控制流（control flow）到达这个变量时，你便得承受构造成本；当这个变量离开其作用域时，你便得承受析构成本。即使这个变量最终并未被使用。

请记住：
+ 尽可能延后变量定义式的出现。这样做可增加程序的清晰度并改善程序效率。

## 条款27：尽量少做转型动作

C++规则的设计目标之一是，保证“类型错误”绝不可能发生。理论上如果你的程序很“干净的”通过编译，就表示它并不企图在任何对象身上执行任何不安全、无意义、愚蠢荒谬的操作。这是一个极具价值的保证。可别草率地放弃它。

两种无差别的“旧式转型”形式：

```cpp
(T)expression // 将expression转型为T
T(expression) // 将expression转型为T
```

C++还提供了四种新式转型（常常被称为new-style或C++ style casts）：

```cpp
const_cast<T>( expression )
dynamic_cast<T>( expression )
reinterpret_cast<T>( expression )
static_cast<T>( expression )
```

+ `const_cast`通常被用来将对象的常量性转除（cast away the constness）。它也是唯一有此能力的C++ style转型操作符。
+ `dynamic_cast`主要用来执行“安全向下转型”（saft downcasting），也就是用来决定某对象是否归属继承体系中的某个类型。它是唯一无法由旧式语法执行的动作，也是唯一可能耗费重大运行成本的转型动作。
+ `reinterpret_cast`意图执行低级转型，例如将 pinter to int转型为一个int。
+ `static_cast`用来强迫隐式转换（implicit conversions），例如将non-const转型为const（反过来只能用`const_cast`），将`void*`指针转换为`typed`指针，将pointer-to-base转换为pointer-to-derivied。

除了对一般转型保持机敏与猜疑，更应该在注重效率的代码中对`dynamic_cast`保持机敏与猜疑。绝对必须避免的一件事是所谓的“连串（cascading）`dynamic_cast`”。

请记住：
+ 如果可以，尽量避免转型，特别是在注重效率的代码中避免`dynamic_cast`。如果有个设计需要转型动作，试着发展无需转型的替代设计。
+ 如果转型是必要的，试着将它隐藏于某个函数背后。客户随后可以调用该函数，而不需要将转型放进他们自己的代码内。
+ 宁可使用C++ style（新式）转型，不要使用旧式转型。前者很容易辨识出来，而且也比较有着分门别类的职掌。

## 条款28：避免返回handles指向对象内部成分

请记住：
+ 避免返回handles（包括references、指针、迭代器）指向对象内部。准守这个条款可增加封装性，帮助const成员函数的行为更像个const，并将发生“虚吊号码牌”（dangling handles）的可能性降至最低。

## 条款29：为“异常安全”而努力是值得的

“异常安全”有两个条件，当异常被抛出时，带有异常安全性的函数会：
+ 不泄漏任何资源。
+ 不允许数据败坏。

有个一般性规则是这么说的：较少的代码就是较好的代码，因为出错机会比较少，而且一旦有所改变，被误解的机会也比较少。

异常安全函数（Exception-safe functions）提供以下三个保证之一：
+ 基本承诺：如果异常被抛出，程序内的任何事物仍然保持在有效状态下。没有任何对象或数据结构会因此而败坏，所有对象都处于一种内部前后一致的状态（例如所有的class约束条件都继续获得满足）。
+ 强烈保证：如果异常被抛出，程序状态不改变。调用这样的函数需有这样的认知：如果函数成功，就是完全成功，如果函数失败，程序会回复到“调用函数之前”的状态。
+ 不抛掷（nothrow）保证：承诺绝不抛出异常，因为它们总是能够完成它们原先承诺的功能。

一个一般化的设计策略很典型地会导致强烈保证，copy and swap：为你打算修改的对象（原件）做出一份副本，然后在那副本上做一切必要修改，如果有任何修改动作抛出异常，原对象仍保持未改变状态，待所有改变都成功后，再将修改过的那个副本和原对象在一个不抛出异常的操作中置换（swap）。

当你撰写新代码或修改旧代码时，请仔细想想如何让它具备异常安全性。首先是“以对象管理资源”，那可阻止资源泄漏；然后是挑选三个“异常安全保证”中的某一个实施于你所写的每一个函数身上。

请记住：
+ 异常安全函数（Exception-safe functions）即使发生异常也不会泄露资源或允许任何数据结构败坏。这样的函数区分为三种可能的保证：基本型、强烈型、不抛异常型。
+ “强烈保证”往往能够以copy-and-swap实现出来，但“强烈保证”并非对所有函数都可实现或具备现实意义。
+ 函数提供的“异常安全保证”通常最高只等于其所调用之各个函数的“异常安全保证”中的最弱者。

## 条款30：透彻了解inlining的里里外外

inline函数背后的整体观念是，将“对此函数的每一个调用”都一函数本体替换之。

inline只是对编译器的一个申请，不是强制命令。这项申请可以隐喻提出，隐喻方式是将函数定义于class定义式内：

```cpp
class Person {
 public:
  ...
  // 一个隐喻的inline申请
  int age() const {return theAge;}
  ...
};
```

请记住：
+ 将大多数inline限制在小型、被频繁调用的函数身上。这可使日后的调试过程和二进制升级（binary upgradability）更容易，也可使潜在的代码膨胀问题最小化，使程序的快速提升机会最大化。
+ 不要只因为function templates出现在头文件，就将它们声明为inline。

## 条款31：将文件间的编译依存关系将至最低

+ 如果使用object references或object pointers可以完成任务，就不要使用object。
    + 你可以只靠一个类型声明式就定义出指向该类型的references和pointers；但如果定义某类型的objects，就需要用到该类型的定义式。
+ 如果能够，尽量以class声明式替换class定义式。
    + 当你声明一个函数而它用到某个class时，你并需要该class的定义；纵使函数以by value方式传递该类型的参数（或返回值）亦然。
+ 为声明式和定义式提供不同的头文件。

通过Handle class封装实现细节：

```cpp
// Person.h

class PersonImpl; // Person实现类的前置声明

class Person {
 public:
  Person(const std::string& name, ...);
  std::string name() const;
  ...
 private:
  std::shared_ptr<PersonImpl> pImpl;
};

// Person.cpp
#include "Person.h"
#include "PersonImpl.h"

Person::Person(const std::string& name, ...)
  : pImpl(new PersonImpl(name, ...)) {}

std::string Person::name() const {
  return pImpl->name();
}
```

通过Interface class封装实现细节：

```cpp
// Person.h

class Person {
 public:
  virtual ~Person();
  virtual std::string name() const = 0;
  ...

  // 工厂函数
  static std::shared_ptr<Person>
    create(const std::string name, ...);
}

// Person.cpp
#include "Person.h"
#include "RealPerson.h"
// class RealPerson : public Person

Person::~Person(){}

std::shared_ptr<Person> Person::create(const std::string& name, ...) {
  return std::shared_ptr<Person>(new RealPerson(name, ...));
}
```

Handle class和Interface class解除了接口和实现之间的耦合关系，从而降低文件间的编译依存关系（compilation dependencies）。

请记住：
+ 支持“编译依存性最小化”的一般构想是：相依于声明式，不要相依于定义式。基于此构想的两个手段是Handle class和Interface class。
+ 程序库头文件应该以“完全且仅有声明式”（full and declaration-only forms）的形式存在。这种做法不论是否涉及templates都适用。

## 条款32：确定你的public继承塑模出is-a关系

> + public继承 意味 is-a
> + virtual函数 意味 接口必须被继承
> + non-virual函数意味 接口和实现都必须被继承

如果你令class D（Derived）以public形式继承class B（Base），你便是告诉C++编译器说，每一个类型为D的对象 同时也是一个类型为B的对象，反之不成立。你的意思是 B比D表现出更一般化的概念，而D比B表现出更特殊化的概念。你主张“B对象可派上用场的任何地方，D对象一样可以派上用场”（此即所谓 Liskov Substitution Principle），因为每一个D对象都是一种（是一个）B对象。反之如果你需要一个D对象，B对象无法效劳，因为虽然每个D对象都是一个B对象，反之并不成立。

> 另外两个class之间的关系是has-a（有一个）和is-implemented-in-terms-of（根据某物实现出）。

请记住：
+ “public继承”意味is-a，适用于base class身上的每一件事情一定也适用于devived class身上，因为每一个devied class对象也都是一个base class对象。

## 条款33：避免遮掩继承而来的名称

当位于一个derived class成员函数内指涉（refer to）base class的某物（也许是个成员函数、typedef、或成员变量）时，编译器可以找出我们所指涉的东西，因为derived class继承了声明于base class内的所有东西。实际运作方式是，derived class作用域被嵌套在base class作用域内。

请记住：
+ derived class内的名称会遮掩base class内的名称。在public继承下从来没有人希望如此。
    + public继承表示is-a，不应该屏蔽函数base class的名字
+ 为了让被遮掩的名字重见天日，可使用using声明式（using Base类名::名字）

## 条款34：区分接口继承和实现继承

public继承概念，由两部分组成：函数接口（function interface）继承和函数实现（function implementations）继承。

+ 成员函数的接口总是会被继承。
+ 声明一个pure virtual函数的目的是为了让derived class只继承函数接口。
+ 声明简朴的（非纯）impure virtual函数的目的，是让derived class继承该函数的接口和缺省实现。
+ 声明non-virtual函数的目的是为了令derived class继承函数的接口及一份强制性实现。

请记住：
+ 接口继承和实现继承不同。在public继承之下，derived class总是继承base class的接口。
+ pure virutal函数只具体指定接口继承。
+ 简朴的（非纯）impure virtual函数具体指定接口继承及缺省实现继承。
+ non-virtual函数具指定定接口继承以及强制性实现继承。

## 条款35：考虑virtual函数意外的其他选择

假设一个游戏任务类：

```cpp
class GameCharacter {
 public:
  virtual int healthValue() const;
  ...
};
```

**藉由Non-Virtual Interface手法实现Template Method模式**：该流量建议，较好的设计是保留healthValue为public成员函数，但是让它称为non-virtual，并调用一个private virtual函数进行实际工作：

```cpp
class GameCharacter {
 public:
  int healthValue() const {
    ... // 可以用于处理各种事前工作
    int retVal = doHealthValue();
    ... // 可以用于处理各种事后工作
    return retVal;
  }
  ...
 private:
  // derived class可重新定义它
  // 缺省算法，计算健康指数
  virtual int do doHealthValue() const {
    ...
  }
};
```

这一基本设计，也就是“令客户通过public non-virtual成员函数间接调用private virtual函数”，称为non-virtual interface（NVI）手法。它是所谓**Template Method**设计模式（与C++ template并无关联）的一个独特表现形式。这个non-virtual函数称为virtual函数的wrapper。

**藉由Function Pointers实现Strategy模式**：主张“人物健康指数的计算与人物类型无关”。例如我们可能会要求每个人物的构造函数接受一个指针，指向一个健康计算函数，而我们可以调用该函数进行实际计算：

```cpp
class GameCharacter; // 前置声明
int defaultHealthCalc(const GameCharacter& gc);

class GameCharacter {
 public:
  typedef int (*HealthCalcFunc)(const GameCharacter&);
  explicit GameCharacter(HealthCalcFunc hcf = defaultHealthCalc)
    : healthFunc(hcf) {}
  int healthValue() const {
    return healthFunc(*this);
  }
  ...
 private:
  HealthCalcFunc hcf;
};
```

这种做法提供了某些弹性：

+ 同一人物类型之不同实体可以有不同的健康计算函数。
+ 某已知人物之健康指数计算函数可在运行期间变更。

**藉由std::function完成Strategy模式**：

```cpp
class GameCharacter; // 前置声明
int defaultHealthCalc(const GameCharacter& gc);

class GameCharacter {
 public:
  typedef std::function<int (const GameCharacter&)> HealthCalcFunc;
  explicit GameCharacter(HealthCalcFunc hcf = defaultHealthCalc)
    : healthFunc(hcf) {}
  int healthValue() const {
    return healthFunc(*this);
  }
  ...
 private:
  HealthCalcFunc hcf;
};
```

和前一个设计比较，这个设计几乎相同。唯一不同的是如今GameCharacter持有一个`std::function`对象，相当于一个指向函数的范化指针。能有更惊人的弹性：

```cpp
short calcHealth(const GameCharacter&); // 注意返回类型为non-int

struct HealthCalculator { // 函数对象
  int operator()(const GameCharacter&) const {
    ...
  }
}

class GameLevel {
 public：
  // 成员函数；注意返回类型为non-int
  float health()(const GameCharacter&) const;
  ...
};

class EviBadGuy: public GameCharacter {
 ... // 人物模型
};
class EyeCandyCharacter: public GameCharacter {
 ... // 人物模型
};

// 使用某个 函数 计算健康指数
EviBadGuy ebg1(calcHealth);
// 使用某个 函数对象 计算健康指数
EyeCandyCharacter ecc1(HealthCalculator());

GameLevel currentLevel;
...
// 使用某个 成员函数 计算健康指数
EviBadGuy ebg2(std::bind(
    &GameLevel::health, currentLevel, _1
));
```

`GameLevel::health`在被调时需要两个参数，隐式的GameLevel和一个GameCharacter，上述`std::bind`可以把一个GameLevel对象绑定在`GameLevel::health`的第一个参数上，以后再调用时就只需要一个GameCharacter，如此就满足了GameCharacter初始化参数的要求。

**古典的Strategy模式**：

```cpp
class GameCharacter; // 前置声明
class HealthCalcFunc {
 // 可以有很多Derived class
 public:
  ...
  virtual int calc(const GameCharacter& gc) const {
    ...
  }
  ...
};

HealthCalcFunc defaultHealthCalc;
class GameCharacter {
 // 可以有很多Derived class
 public:
  explicit GameCharacter(HealthCalcFunc* phcf = &defaultHealthCalc)
    : pHealthCalc(phcf) {}
  int healthValue() const {
    return pHealthCalc->calc(*this);
  }
 private:
  HealthCalcFunc* pHealthCalc;
};
```

摘要：
+ 使用non-virtual interface（NVI）手法，那是**Template Method**设计模式的一种特殊形式。它以public non-virutal成员函数包裹降低访问性（private或protected）的virtual函数。
+ 将virtual函数替换为“函数指针成员变量”，这是**Strategy设计模式**的一种分解表现形式。
+ 以`std::function`成员变量替换virtual函数，因而允许使用任何可调用物（callable entity）搭配一个兼容于需求的签名式。这也是Strategy设计模式的某种形式。
+ 将继承体系内的virtual函数替换为另一个继承体系内的virtual函数。这是Strategy设计模式的传统实现手法。

请记住：
+ virtual函数的替代方案包括NVI手法及Strategy设计模式的多种形式。NVI手法自身是一个特殊形式的Template Method设计模式。
+ 将机能从成员函数移到class外部函数，带来的一个缺点是：非成员函数无法访问class的non-public成员。
+ `std::function`对象的行为就像一般函数指针。这样的对象可接纳“与给定之目标签名式（target signature）兼容”的所有可调用物（callable entity）。

## 条款36：绝不重新定义继承而来的non-virtual函数

请记住：
+ 绝对不要重新定义继承而来的non-virtual函数。

## 条款37：绝不重新定义继承而来的缺省参数值

virtual函数系动态绑定（dynamically bound），而缺省参数值却是静态绑定（statically bound）。动态绑定又名后期绑定，late binding；静态绑定又名前期绑定，early binding。

考虑以下的class继承体系：

```cpp
class Shape {
 public:
  enum ShapeColor {Red, Green, Blue};
  virtual void draw(ShapeColor color = Red) const = 0;
  ...
};

class Rectangle : public Shape {
 public:
  // 注意，赋予不同的缺省参数值
  virtual void draw(ShapeColor color = Green) const;
  ...
};
class Circle : public Shape {
 public:
  // 去掉了缺省值，寓意需要一个参数
  virtual void draw(ShapeColor color) const;
};
```

考虑如下使用代码：

```cpp
Shape* pc = new Circle();    // pc的静态类型是Shape*，动态类型是Circle*
Shape* pr = new Rectangle(); // pr的静态类型是Shape*，动态类型是Circle*

pc->draw(); // 因为缺省参数是静态绑定，所以效果等同于 pc->draw(Shape::Red);
pr->draw(); // 因为缺省参数是静态绑定，所以可以通过，虽然Circle::draw定义需要明确参数
```

上面调用时，函数体来自Derived类，缺省参数又来自Base类，很糟糕！

但如果试着遵守这条规则，并且同时提供缺省参数值，那么会有代码重复 以及 相依性：

```cpp
class Shape {
 public:
  enum ShapeColor {Red, Green, Blue};
  virtual void draw(ShapeColor color = Red) const = 0;
  ...
};

class Rectangle : public Shape {
 public:
  virtual void draw(ShapeColor color = Red) const;
  ...
};
```

当你想令virtual函数表现出你所想要的行为但却遭遇麻烦，聪明的做法是考虑替代设计，例如NVI手法：

```cpp
class Shape {
 public:
  enum ShapeColor {Red, Green, Blue};
  virtual void draw(ShapeColor color = Red) const {
    doDraw(color);
  }
  ...
 private:
  virtual void doDraw(ShapeColor color) const = 0;
};

class Rectangle : public Shape {
 public:
  ...
 private:
  virtual void doDraw(ShapeColor color) const = 0;
};
```

请记住：
+ 绝对不要重新定义一个继承而来的缺省参数值，因为缺省参数值都是静态绑定，而virtual函数——你唯一应该覆写的东西——却是动态绑定。

## 条款38：通过复合塑模出has-a或“根据某物实现出”

复合（composition，同义词，layering（分层），containment（内含），aggregation（聚合）和embedding（内嵌））是类型之间的一种关系，当某种类型的对象内含它种类型的对象，便是这种关系。例如：

```cpp
class Address {...};
class PhoneNumber {...};

class Person {
 public:
  ...
 private:
  std::string name;        // 合成成分物（composed object）
  Address address;         // 同上
  PhoneNumber voiceNumber; // 同上
  PhoneNumber faxNumber;   // 同上
};
```

复合意味has-a（有一个）或is-implemented-in-terms-of（根据某物实现出）。

请记住：
+ 复合（composition）的意义和public继承完全不同。
+ 在应用域（application domain），复合意味has-a（有一个）。在实现域（implementation domain），复合意味is-implemented-in-terms-of（根据某物实现出）。

## 条款39：明智而审慎地使用private继承

和public继承不同：如果class之间的继承关系是private，编译器不会自动将一个derived class对象转换为一个base class对象。由private base class继承而来的所有成员，在derived class中都会变成private 属性。

private继承意味 implemented-in-terms-of（根据某物实现出）。private继承纯粹只是一种实现技术。private继承纯的含义与复合一样，在实现上，尽可能使用复合，必要时（当protected成员 或 virtual函数牵扯进来时）才使用private继承。

```cpp
class Timer {
 public:
  explicit Timer(int tickFrequency);
  // 定时器每滴答一次，此函数就被自动调用一次
  virtual void onTick() const;
  ...
};

class Widget : private Timer {
 private:
  // 利用定时器做一些事情
  virtual void onTick() const;
};
```

但是可以以复合（composition）替代：

```cpp
class Widget {
 private:
  class WidgetTimer : public Timer {
   public:
    virtual void onTick() const;
    ...
  };
  WidgetTimer timer;
  ...
};
```

此方案同时还可以防止derived class重新定义onTick。

当你面对“并不存在is-a关系”的两个class，其中一个需要访问另一个的protected成员，或需要重新定义其一或多个virtual函数，private继承极有可能成为正统设计策略。一个混合了public继承和复合的设计，往往能够释出你要的行为，尽管这样的设计有较大的复杂度。“明智而审慎地使用private继承”意味，在考虑过所有其他方案之后，如果仍然认为private继承是“表现程序内两个class之间的关系”的最佳办法，这才用它。

请记住：
+ private继承意味is-implemented-in-terms-of（根据某物实现出）。它通常比复合（composition）的级别低。但是当derived class需要访问protected base class的成员，或需要重新定义继承而来的virtual函数时，这么设计是合理的。
+ 和复合（composition）不同，private继承可以造成empty base最优化。这对致力于“对象尺寸最小化”的程序库开发者而言，可能很重要。

## 条款40：明智而审慎地使用多重继承

一旦涉及多重继承（multiple inheritance；MI），C++社群便分为两个基本阵营。其一认为如果单一继承（single inheritance；SI）是好的，多重继承一定更好；另一派阵营则主张，单一继承是好的，但多重继承不值得拥有（或使用）。

当MI进入设计景框，程序有可能从一个以上的base class继承相同的名称（如函数、typedef等等）。那会导致较多的歧义（ambiguity）机会。

多重继承的意思是继承一个以上的base class，但这些base class并不常在继承体系中又有更高级的base class，因为那些会导致要命的“钻石型多重继承”：

```cpp
class File {...};
class InputFile : public File {...};
class OutputFile : public File {...};
class IOFile : public InputFile, public OutputFile {
 ...
};
```

如果File终有一个fileName成员变量，默认情况下，IOFile经`File->InputFile->IOFile`、`File->OutputFile->IOFile`两条继承链路会分别继承一个fileName；如果那不是你要的，你必须令那个带有此数据的class成为一个virtual base class：

```cpp
class File {...};
class InputFile : virtual public File {...};
class OutputFile : virtual public File {...};
class IOFile : public InputFile, public OutputFile {
 ...
};
```

对virtual base class的忠告：第一，非必要不要使用virtual class，平常使用non-virtual继承；第二，如果你必须使用virtual base class，尽可能避免在其中放置数据。

请记住：
+ 多重继承比单一继承复杂。他可能导致新的歧义性，以及对virtual继承的需要。
+ virtual继承会增加大小、速度、初始化（及赋值）复杂度等等成本。如果virutal base class不带任何数据，将是最具实用价值的情况。
+ 多重继承的确有正当用途。其中一个情结涉及“public继承某个Interface class”和“private继承某个协助实现的class”的两相组合。

## 条款41：了解隐式接口和编译期多态

面向对象变成世界总是以显式接口（explicit interface）和运行期多态（runtime polymorphism）决绝问题。

Template及泛型编程的世界，与面向对象有根本上的不同。在此世界中显式接口和运行期多态仍然存在，但重要性降低。反倒是隐式接口（implicit interface）和编译期多态（compile-time polymorphism）移到前头了。

通常显式接口由函数的签名式（也就是函数名称、参数类型、返回类型）构成。例如：

```cpp
class Widget {
 public:
  Widget();
  virtual ~Widget();
  virtual std::size_t size() const;
  virtual void normalize();
  void swap(Widget& other);
};
```

隐式接口就完全不同了，它并不基于函数签名式，而是由有效表达式（valid expression）组成：

```cpp
template<typename T>
void doProcessing(T& w) {
  if (w.size() > 10 && w != someNastyWidget) {
    ...
  }
  ...
}
```

T的隐式接口看起来好像有这些约束：
+ 它必须提供一个名为size的成员函数，该函数返回一个整数。
    + 或者size函数返回类型支持 `operator>(int)`
+ 它必须支持一个`operator!=`函数，用来比较两个T对象。
    + 或者T的隐式转换类型支持`operator!=(someNastyWidget的类型)`

加诸于template参数身上的隐式接口，就像加诸于class对象身上的显式接口一样真实，而且两者都在编译期完成检查。就像你无法以一种“与class提供之显式接口矛盾”的方式使用对象（代码将编译不过），你也无法在template中使用“不支持template所要求之隐式接口”的对象（代码一样编译不过）。

请记住：
+ class和template都支持接口（interface）和多态（polymorphism）。
+ 对class而言，接口是显式的（explicit），以函数签名为中心。多态则是通过virtual函数发生于运行期。
+ 对template参数而言，接口是隐式的（implicit），奠基于有效表达式。多态则是通过template具现化和函数重载解析（function overloading resolution）发生于编译期。

## 条款42：了解typename的双重意义

声明template参数时，不论是用关键字class或typename意义完全相同。另外，任何时候当你想要在template中指涉一个嵌套从属类型名称，就必须在紧邻它的前一个位置放上关键字typename：

```cpp
template<typename C>       // typename也可是class
void f(const C& container, // 不允许使用typename
       typename C::iterator iter) {    // 嵌套从属类型，一定要会用typename
  if (container.size() >= 2) {
    // 一定要typename
    typename C::const_iterator *pIter; // 嵌套从属类型，一定要会用typename
    int val = *(container.begin());
    ...
  }
}
```

> template内出现的名称如果想依于template参数，称之为从属名称（dependent names）。如果从属名称在class内呈嵌套状，我们称它为嵌套从属名称（nested dependent name）。`C::const_iterator`就是这样一个名称。实际上它还是嵌套从属类型名称（nested dependent type name）。
> template内的不依赖于template参数的名称，称为非从属名称（non-dependent name）。

但有例外，typename不可以出现在base classes list内的嵌套从属类型名称前，也不可在member intialization list中作为base clas修饰符：

```cpp
template<typename T>
class Derived : public Base<T>::Nested { //base class list
 public:
  explicit Derived(int x)
   : Base<T>::Nested(x) { // mem init list
    typename Base<T>::Nested temp;
    ...
  }
};
```

请记住：
+ 声明template参数时，前缀关键字class和typename可互换。
+ 请使用关键字typename标识嵌套从属类型名称；但不得在base class list或member intialization list内以它作为base class修饰符。

## 条款43：学习处理模版化基类内的名称

请记住：
+ 可在derived class template内通过“this->”指涉base class templatete内的成员名称，或藉由一个明白写出的“base class资格修饰符”完成。

## 条款44：将与参数无关的代码抽离template

请记住：
+ template生成多个class和多个函数，所以任何template代码都不该与某个造成膨胀的template参数产生相依关系。
+ 因非类型模板参数（non-type template parameter）而造成的代码膨胀，往往可消除，做法是以函数参数或class成员变量替换template参数。
+ 因类型参数（type parameter）而造成的代码膨胀，往往可降低，做法是让带有完全相同二进制表述（binary representation）的具现类型（instantiation type）共享实现代码。

## 条款45：运用成员函数模板接受所有兼容类型

假设有如下继承关系：

```cpp
class Top {...};
class Middle : public Top {...};
class Bottom : public Middle {...};
```

使用原始指针可以很方便的做隐式转换：

```cpp
Top* pt1 = new Middle();
Top* pt2 = new Bottom();
```

如果你自己实现一个只能指针类，那么不太能直接完成隐式转换：

```cpp
template<typename T>
class SmartPtr {
 public:
  explicit SmartPtr(T* realPTr);
  ...
};

SmartPtr<Top> pt1 =                //编译不过
  SmartPtr<Middle>(new Middle()); 
```

**Template和泛型编程（Generic Programming）**

就上述情况而言，我们需要为SmartPtr写一个构造模版，这样的模版是所谓member function template（常简称为member template），其作用是为class生成函数：

```cpp
template<tyname T>
class SmartPtr {
 public:
  template<typename U>  // 泛化（generalized）copy构造函数
  SmartPtr(const SmartPtr<U>& other); // 没有explicit为了实现隐式转换
};
```

这个泛化copy构造函数，还能提供根据`SmartPtr<Top>`创建`SmartPtr<Middle>`的转换，但这并不是我们想要的。我们可以如下实现转换限制：

```cpp
template<tyname T>
class SmartPtr {
 public:
  template<typename U>
  SmartPtr(const SmartPtr<U>& other)
   : heldPtr(other.get()) {...} // 以other内的指针初始化heldPtr
                                // C++可以自动完成转换检查
  T* get() const {return heldPtr;}
  ...
 private:
  T* heldPtr;
};
```

请记住：
+ 请使用member function template（成员函数模版）生成“可接受所有兼容类型”的函数。
+ 如果你声明member template用于“泛化copy构造”或“泛化assignment操作”，你还是需要声明正常的copy构造函数和copy assignment操作符。

## 条款46：需要类型转换时请为模版定义非成员函数

扩充条款24的例子，将Rational和`operator*`模版化：

```cpp
template<typename T>
class Rational {
 public:
  Rational(const T& numerator = 0,
           const T& demoninator = 1);
  const T numerator() const;
  const T demoninator() const;
  ...
};

template<T> // 为了实现混合运算，以non-member实现
const Rational<T> operator* (const Rational<T>& lhs,
                             const Rational<T>& rhs) {
  ...
}
```

但是，因为template实参推导过程中并不考虑采纳“通过构造函数而发生的”隐式类型转换，如下代码无法通过编译：

```cpp
Rational<int> oneHalf(1,2);
Rational<int> result = oneHalf * 2; // 编译不过
```

可以通过在class template内声明该function template为友元来完成该函数的推导：

```cpp
template<typename T>
class Rational {
 public:
  ...
  // 声明operator*函数
  // 在class Rational被具现化时，就会具现化出operator*的声明
  friend
  const Rational operator* (const Rational& lhs,
                            const Rational& rhs);
};

template<T> // 定义operator*函数
const Rational<T> operator* (const Rational<T>& lhs,
                             const Rational<T>& rhs) {
  ...
}
```

但此时，无法连接，因为“定义operator*函数”的地方，还是无法正确完成推导。此时，我们可以把其定义与声明放在一起：

```cpp
template<typename T>
class Rational {
 public:
  ...
  friend
  const Rational operator* (const Rational& lhs,
                            const Rational& rhs) {
    return Rational(lhs.numerator()* rhs.numerator(), ...);
  }
};
```

为了让类型转换可能发生于所有实参身上，我们需要一个non-member函数；为了令这个函数自动具现换，我们需要将它声明在class内部；而在class内部声明non-member函数的唯一办法就是：令它成为一个friend。

请记住：
+ 当我们编写一个class template，而它所提供之“于此template相关的”函数支持“所有参数之隐式类型转换”时，请将那些函数定义为“class template内部的friend函数”。

## 条款47：请使用traits class表现类型信息

STL共有5种迭代器分类，对应于它们支持的操作：
+ Input迭代器：只能向前移动，一次一步，只可读取。例如，`istream_iterators`。
+ Output迭代器：只能向前移动，一次一步，只可涂写。例如，`ostream_iterators`。
+ Forward迭代器：可以做前述两种分类所能做的每一件事，而且可以读或者写其所指物一次以上。例如，单向linked list的迭代器。
+ Bidirectional迭代器：可以双向移动。例如，STL的set、map、list等迭代器。
+ Random Access迭代器：可执行“迭代器算数”，在常量时间内向前或向后跳跃任意距离。例如，STL中vector、deque、string的迭代器。

对于这5中分类，C++标准程序库分别提供专属的卷标结构（tag struct）加以确认：

```cpp
struct input_iterator_tag {};
struct output_iterator_tag {};
struct forward_iterator_tag : public input_iterator_tag {};
struct bidirectional_iterator_tag : public forward_iterator_tag {};
struct random_access_iterator_tag : public bidirectional_iterator_tag {};
```

假设要实现一个名为advance的函数，用来将某个迭代器移动某个给定距离：

```cpp
template<typename IterT, typename DistT>
void advance(IterT& iter, DistT d); // 如果d<0 则向后移动
```

因为不同迭代器分类有不同的移动能力，advance的实现最好根据分类加以区分，例如：

```cpp
template<typename IterT, typename DistT>
void advance(IterT& iter, DistT d) {
  if (iter is random access iterator) {
    iter += d;
  } else {
    ... // 一步一步移
  }
}
```

为了取得类型的某些信息，可以使用traits技术：它们是一种技术，也是一个C++程序员共同遵守的协议。例如迭代器traits：

```cpp
template<typename IterT>
struct iterator_traits;
```
`iterator_traits`的运作方式是，针对每一个类型IterT，在`struct iterator_traits<IterT>`内一定声明某个typedef为`iterator_category`，用来确认IterT的迭代器分类。

首先，它要求“用户自定义的迭代器类型”必须嵌套一个typedef：

```cpp
template<...>
class deque {
 public:
  class iterator {
   public:
    typedef random_access_iterator_tag iterator_category;
    ...
  };
  ...
};

template<...>
class list {
 public:
  class iterator {
   public:
    typedef bidirectional_iterator_tag iterator_category;
    ...
  }
  ...
};
```

而`iterator_traits`只需：

```cpp
template<typename IterT>
struct iterator_traits {
 typedef typename IterT::iterator_category iterator_category;
};
```

然后对于内置类型——指针，特别提供一个偏特化版本（partial template specialization）：

```cpp
template<typename IterT>
struct iterator_traits<IterT*> {
 typedef random_access_iterator_tag iterator_category;
};
```

现在，就可以对迭代器获取分类信息，然后操作：

```cpp
template<typename IterT, typename DistT>
void advance(IterT& iter, DistT d) {
  if (typeid(typename std::iterator_traits<IterT>::iterator_category)
    == typeid(std::random_access_iterator_tag)) {
    ... 
  } else {
    ...
  }
}
```

最后，可以通过重载，把运行期的if判断，提前至编译期：

```cpp
template<typename IterT, typename DistT>
void doAdvance(IterT& iter, DistT d,
             std::random_access_iterator_tag) {
  ...
}

template<typename IterT, typename DistT>
void doAdvance(IterT& iter, DistT d,
             std::bidirectional_iterator_tag) {
  ...
}
...

template<typename IterT, typename DistT>
void doAdvance(IterT& iter, DistT d) {
  doAdvance(iter, d,
    typename std::iterator_traits<IterT>::iterator_category);
}
```

traits广泛用于标准程序库，其中上述讨论的`iterator_traits`除了供应`iterator_category`还供应另外四分迭代器相关信息（其中最常见的是`value_type`）。此外，还有`char_traits`用来保存字符类型的相关信息，以及`numeric_limits`用来保存数值类型的相关信息（例如最值），等等。

请记住：
+ traits class使得“类型相关信息”在编译期可用。它们以template和“template特化”完成实现。
+ 整合重载技术（overloading）后，traits class有可能在编译期对类型执行`if...else`测试。

## 条款48：认识template元编程

template metaprogramming（TMP，模版元编程）是编写template-based C++程序并执行与编译期的过程。

TMP有两个伟大的效力。第一，它让某些事情更容易；第二，由于template metaprogram执行于C++编译期，因此可将工作从运行期转移到编译期。

TMP的起手程序是在编译期计算阶乘，TMP的阶乘运算示范如果通过“递归模版具现化”（recursive template instantiation）实现循环，以及如何在TMP中创建和使用变量：

```cpp
template<unsigned n>
struct Factorial {
  enum {value = n * Factorial<n-1>::value};
};
template<>
struct Factorial<0> {
  enum {value = 1};
}
```

如此你可以通过`Factorial<n>::value`得到n阶乘值。

为了领悟TMP之所以值得学习，很重要的一点实现对它能够达成什么目标有一个比较好的理解，下面举三个例子：

+ 确保量度单位正确。
+ 优化矩阵运算。
+ 可以生成客户定制之设计模式（custom design pattern）实现品。

请记住：
+ template metaprogramming（TMP，模版元编程）可将工作由运行期移往编译期，因而得以实现早起错误侦测和更高的执行效率。
+ TMP可被用来生成“基于政策选择组合”（based on combinations of policy choice）的客户定制代码，也可用来避免生成对某些特殊类型并不适合的代码。

## 条款49：了解new-handler的行为

当operator new抛出异常以反应一个未获满足的内存需求之前，它会先调用一个客户指定的错误处理函数，一个所谓的`new-handler`。为了指定这个“用以处理内存不足”的函数，客户必须调用`set_new_handler`：

```cpp
namespace std {
typedef void (&new_handler)();
new_handler set_new_handler(new_handler p) throw();
}
```

可以这样使用`set_new_handler`：

```cpp
void outOfMem() {
  std::cerr << "Unable to satisfy request for memory\n";
  std::abort();
}

int main() {
  std::set_new_handler(outOfMem);
  int* pBig = new int[100000000L];
}
```

当operator new无法满足内存申请时，它会不断调用new-handler函数，知道找到足够内存。一个设计良好的new-handler函数必须做一下事情：
+ 让更多内存可被使用。
+ 安装另一个new-handler。转给它认为能释放更多内存的handler处理。
+ 卸除new-handler。即将null给`set_new_handler`，operator new会在内存分配不成功时抛异常。
+ 抛出（或派生自）`bad_alloc`的异常。
+ 不返回。`abort`或`exit`。

我们可以通过如下方式给单独一个class指定自己的new-handler：

```cpp
// -- 声明文件 --
class Widget {
 public:
  // 供客户设置Widget私有的new-handler
  static std::new_handler set_new_handler(std::new_handler p) throw();
  // 声明该函数后，new对象时使用该函数替代 ::operator new
  static void* operator new(std::size_t size) throw(std::bad_alloc);
 private:
  static std::new_handler currentHandler;
};

// -- 实现文件 --
std::new_handler Widget::currentHandler = 0;

// 设置Widget私有的new-handler，仅记录下来
std::new_handler set_new_handler(std::new_handler p) throw() {
 std::new_handler oldHandler = currentHandler;
 currentHandler = p;
 return oldHandler;
}

// RAII对象，释放时恢复global new-handler为传入的new-handler
class NewHandlerHolder {
 public:
  explicit NewHandlerHolder(std::new_handler nh)
   : handler(nh) {}
  ~NewHandlerHolder() {
    std::set_new_handler(handler);
  }

  NewHandlerHolder(const NewHandlerHolder&) = delete;
  NewHandlerHolder& operator=(const NewHandlerHolder&) = delete;
 private:
  std::new_handler handler;
};

void* Widget::operator new(std::size_t size) throw(std::bad_alloc) {
  // 设置global new-handler为私有new-handler 
  // 并把原new-handler初始化给RAII，达到最后还原的目的 
  NewHandlerHolder h(std::set_new_handler(currentHandler));
  // 调用全局的operator new做实际的new操作
  return ::operator new(size);
}
```

不同class都可以实现这一套机制，那么可以将此设计为一个基类：

```cpp
// 之所以是template，是为了给每一个类型维护自己的static std::new_handler
template<typename T>
class MewHandlerSupport {
 public:
  static std::new_handler set_new_handler(std::new_handler p) throw();
  static void* operator new(std::size_t size) throw(std::bad_alloc);
 private:
  static std::new_handler currentHandler;
};

template<typename T>
std::new_handler MewHandlerSupport<T>::currentHandler = 0;
template<typename T>
std::new_handler 
MewHandlerSupport<T>::set_new_handler(std::new_handler p) throw() {
 std::new_handler oldHandler = currentHandler;
 currentHandler = p;
 return oldHandler;
}
template<typename T>
void* MewHandlerSupport<T>::operator new(std::size_t size) throw(std::bad_alloc) {
  NewHandlerHolder h(std::set_new_handler(currentHandler));
  return ::operator new(size);
}
```

请记住：
+ `set_new_handler`允许客户指定一个函数，在内存分配无法获得满足时被调用。
+ nothrow new（`new (std::nothrow) Widget()`）是一个颇为局限的工具，因为它只适用于内存分配，后继的构造函数调用还是可能抛出异常。

## 条款50：了解new和delete的合理替换时机

替换编译器提供的operator new或operator delete的最常见的三个理由：
+ 用来检测运用上的错误。
+ 为了强化效能。
+ 为了收集使用上的统计数据。

例如一个定制的global operator new的简单例子，促进并协助检测“overruns”（写入点在分配区块尾端之后）或“underruns”（写入点在分配区块起点之前）：

```cpp
static const int signature = 0xDEADBEEF;
typedef unsigned char Byte;

// 先忽略条款51中讨论的new需固守的常规
void* operator new(std::size_t size) throw(std::bad_alloc) {
  using namespace std;
  size_t realSize = size + 2*sizeof(int);
  void* pMem = malloc(realSize);
  if (!pMem) throw bad_alloc();

  // 将signature写入内存的最前端和最后端
  *(static_cast<int*>(pMem)) = signature;
  *(reinterpret_cast<int*>(
      static_cast<Byte*>(pMem) + realSize - sizeof(int)
  )) = signature;
  return static_cast<Byte*>(pMem) + sizeof(int);
}
```

合理替换缺省的new和delete的一些目的：
+ 为了检测运用错误。
+ 为了收集动态分配内存之使用统计信息。
+ 为了增加分配和归还的速度。
+ 为了降低缺省内存管理器带来的空间额外开销。
+ 为了弥补缺省分配其中的非最佳齐位。
+ 为了将相关对象成簇集中。
+ 为了获得非传统的行为。

请记住：
+ 有许多理由需要写一个自定的new和delete，包括改善效能、对heap运用错误进行调试、收集heap使用信息。

## 条款51：编写new和delete时需固守常规

实现一致性operator new必得返回正确的值，内存不足时必得调用new-handler函数，必须有对零内存需求的准备，还需避免不慎掩盖正常形式的new。

一个non-member operator new伪代码：

```cpp
// 你的operator new可能接受额外参数
void* operator new(std::size_t size) throw(std::bad_alloc) {
  using namespace std;
  if (size == 0) {
    size = 1;
  }
  while (true) {
    尝试分配size bytes;
    if (分配成功)
      return (一个指针，指向分配得来的内存);
    
    // 分配失败；找出目前的new-handler函数
    new_handler globalHandler = set_new_handler(0);
    set_new_handler(globalHandler);

    if (globalHandler) (*globalHandler)();
    else throw std::bad_alloc();
  }
}
```

考虑class定制的operator new，因是成员函数，所以可以被继承，但是一般不想（因为大小可能不一样）：

```cpp
class Base {
 public:
  static void* operator new(std::size_t size) throw(std::bad_alloc);
};
class Dervied : public Base {
 ...
}；

void* operator new(std::size_t size) throw(std::bad_alloc) {
  // 如果不是Base类一样的大小，就转交标准operator new处理
  if (size != sizeof(Base))
    return ::operator new(size);
  ...
}
```

如果打算实现class专属的“array new”：`operator new[]`。唯一需要做的一件事就是分配一块未加工内存（raw memory）。

`operator delete`情况更简单，你需要记住的唯一事情就是C++保证“删除null指针永远安全”：

```cpp
void operator delete(void* rawMemory) throw() {
  if (rawMemory == 0) return;
  ...
}
```

member版本，只需要多加一个动作检查删除大小：

```cpp
class Base {
 public:
  static void* operator new(std::size_t size) throw(std::bad_alloc);
  static void operator delete(void* rawMemory, std::size_t size) throw();
  ...
};

void operator delete(void* rawMemory, std::size_t size) throw() {
  if (rawMemory == 0) return;
  if (size != sizeof(Base)) {
    ::operator delete(rawMemory);
    return;
  }
  ...
  return;
}
```

请记住：
+ operator new应该内含一个无穷循环，并在其中尝试分配内存，如果它无法满足内存需求，就应该调用new-handler。它也应该有能力处理0 bytes申请。class专属版本则还应该处理“比正确大小更大的（错误）申请”。
+ operator delete应该在收到null指针时不做任何事。class专属版本还应该处理“比正确大小更大的（错误）申请”。

## 条款52：写了placement new也要写placement delete

如果制定的operator new接受的参数除了一定会有的那个`size_t`之外还有其他，这便是所谓的placement new：

```cpp
class Widget {
 public:
  static void* operator new(std::size_t size, std::ostream& logStream)
    throw(std::bad_alloc);
  ...
};
```

在众多placement new版本中，特别有用的一个是“接受一个指针指向对象该被构造之处”：

```cpp
class Widget {
 public:
  // 说到placement new时，也大多指的这个版本
  static void* operator new(std::size_t size, void* pMemory)
    throw(std::bad_alloc);
  ...
};
```

如果一个带额外参数的operator new没有“带相同额外参数”的对应版operator delete，那么当new的内存分配动作需要取消并恢复旧观时就没有任何operator delete会被调用。

```cpp
class Widget {
 public:
  static void* operator new(std::size_t size, std::ostream& logStream)
    throw(std::bad_alloc);
  static void operator delete(void* pMemory) throw();
  static void operator delete(void* pMemory, std::ostream& logStream) throw();
  ...
};

// 先调用Widget::new - placement版本
// 然后做Widget构造，如果有异常会调用 Widget::delete - placement版本
Widget* pw = new (std::cerr) Widget; 

// 调用 Widget::delete - 非placement版本
// placement delete只在“伴随placement new调用而触发的构造函数”出现异常才会被调用
delete pw;
```

缺省情况下，C++在global作用域内提供一下形式的operator new

```cpp
// normal new
void* operator new(std::size_t size) throw(std::bad_alloc);
// placement new
void* operator new(std::size_t size, void*) throw();
// nothrow new
void* operator new(std::size_t size, const std::nothrow_t&) throw();
```

为了防止自制的operator new/delete，可以在base class内含所有形式的new和delete：

```cpp
class StandardNewDeleteForms {
 public:
  // normal new
  static void* operator new(std::size_t size) throw(std::bad_alloc)
  { return ::operator new(size); }
  static void operator delete(void* pMemory) throw()
  { ::operator delete(pMemory); }
  // placement new
  static void* operator new(std::size_t size, void* ptr) throw()
  { return ::operator new(size, ptr); }
  static void operator delete(void* pMemory, void* ptr) throw()
  { ::operator delete(pMemory, ptr); }
  // nothrow new
  static void* operator new(std::size_t size, const std::nothrow_t& nt) throw()
  { return ::operator new(size, nt); }
  static void operator delete(void* pMemory, const std::nothrow_t&) throw()
  { ::operator delete(pMemory); }
};
```

请记住：
+ 当你写一个placement operator new，请确定也写出了对应的placement operator delete，如果没有这样做，你的程序可能会发生隐晦微妙而时断时续的内存泄漏。
+ 当你声明placement new和placement delete，请确定不要无意识（非故意）低遮掩了其他的正常版本。

## 条款53：不要轻忽编译器的告警

请记住：
+ 严肃对待编译器发出的告警信息。努力在你的编译器的最高（最严苛）告警级别下争取“无任何告警”的荣誉。
+ 不要过度依赖编译器的报警能力，因为不同的编译器对待事情的态度并不相同。一旦一直到另一个编译器上，你原本依赖的告警信息有可能消失。

## 条款54：让自己数据包括TR1在内的标准程序库

## 条款55：让自己熟悉Boost

[boost](https://www.boost.org/)