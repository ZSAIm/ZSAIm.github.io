---
layout: post
title:  "C++ 临时变量"
categories: C++
tags: 编程 C++ Cpp 临时对象 临时变量
author: ZSAIm
---

* content
{:toc}


### 什么是临时变量？

> 临时变量大多数情况下是算数表达式的结果。

考虑如下代码

```cpp
int main()
{
  int a=1, b=3, c=5;
  // 如下行代码，其中 (a + b) 所生成的临时量就是临时变量。
  c = a + b;
}
```

### 为什么会产生临时变量？

其中整个过程是这样的：







1. 假想存在一个临时变量``tmp``（注意实际上并不存在这样一个变量名）。
2. 执行 ``tmp = a + b`` 表达式，也就是将``a + b``的结果保存到一个临时变量里面。
3. 执行 ``c = tmp``。


看起来或许会觉得步骤二就是多余的，因为直观上觉得这整一个行直接给c赋值a+b不就行了吗！

确实是这样，直观上这是很合情合理的，但是机器并不懂这些东西，他只知道运算、存储结果和转移结果(这里的转移也可以理解成拷贝)。所以他首先是运算结果 ``a + b``，然后存储运算的结果``tmp``，然后转移结果``c = tmp``。

你可能会说：为什么一定要经过``tmp``来转移结果，直接将运算的结果存储到``c``不是来得更直接吗？
没错，这确实是更直接的，但就像我前面所说的，机器并不知道这些东西（或许编译器优化得足够好在这例子里面能够直接给c复制，不过这里我们都以不优化为前提进行分析，因为这样的现象在临时对象中更为普遍），这样编译器会以通用的方式来实现这样的过程，所以他会先将其结果存储到一个临时变量里面，然后再将其存储到目的变量。

### 临时变量生成过程

对于前面的built-in类型我们无法更深入的了解这一过程，因为我们无法重载built-in类型的运算符、构造和析构这些函数，自然我们就无法通过单步调试来了解这一个过程。为了进行单步调试来加深理解，我们可以通过构建一个简单的类和重载必要的函数来展现这一整个过程。


```Cpp
#include <iostream>
using namespace std;

class X {
public:
	int counter;

	//构造函数
	X(int index) {
		counter = index;
		cout << "constructor counter = " << counter << ", addr=" << this << endl;
	}
	//析构函数
	~X() {
		cout << "destructor counter = " << counter << ", addr=" << this << endl;
	}
	//重载 = 赋值运算符
	X& operator=(const X& from) {
		counter = from.counter;
		cout << "assign=： counter: " << counter << " = " << from.counter << endl;
		cout << "assign=： addr: " << "(" << this << ")" << " <= " << "(" << &from << ")" << endl;
		return *this;
	}
	//重载 + 运算符
	X operator+(const X& from) {
		X tmp = *this;
		tmp.counter += from.counter;
		cout << "operator+： counter: " << counter << " <= " << from.counter << endl;
		cout << "operator+： addr: " << "(" << this << ")" << " <= " << "(" << &from << ")" << endl;
		return tmp;
	}
	//重载 = 拷贝运算符
	X(const X& from) {
		counter = from.counter;
		cout << "copy=： counter: " << counter << " <=copy " << from.counter << endl;
		cout << "copy=： addr: " << "(" << this << ")" << " <= " << "(" << &from << ")" << endl;
	}

};

int main()
{
	X a(0), b(1), c(3), d(5);
  // 下一行就是临时对象产生的地方
	a = b + c;
}
```

得到如下输出结果（建议自行单步走一走）

```
constructor counter = 0, addr=0041F730
constructor counter = 1, addr=0041F724
constructor counter = 3, addr=0041F718
constructor counter = 5, addr=0041F70C
copy=： counter: 1 <= 1
copy=： addr: (0041F5E4) <=copy (0041F724)
operator+： counter: 1 + 3
operator+： addr: (0041F724) <= (0041F718)
copy=： counter: 4 <= 4
copy=： addr: (0041F640) <= (0041F5E4)
destructor counter = 4, addr=0041F5E4
assign=： counter: 4 = 4
assign=： addr: (0041F730) <= (0041F640)
destructor counter = 4, addr=0041F640
destructor counter = 5, addr=0041F70C
destructor counter = 3, addr=0041F718
destructor counter = 1, addr=0041F724
destructor counter = 4, addr=0041F730
```


其中临时对象所存放的地址是``0041F640``。

***
### 逐步分析

#### "X a(0), b(1), c(3), d(5)"

```cpp
X a(0), b(1), c(3), d(5);
```
* 构造a,b,c,d，得到结果。

```
constructor counter = 0, addr=0041F730
constructor counter = 1, addr=0041F724
constructor counter = 3, addr=0041F718
constructor counter = 5, addr=0041F70C
```

#### "a = b + c"

```Cpp
a = b + c;
```

按照以上分析临时变量的生成步骤进行分析。

##### 执行 ``b + c``，对象进行加法操作，进入函数operator+，

```cpp
//重载 + 运算符
X operator+(const X& from) {
  X tmp = *this;
  tmp.counter += from.counter;
  cout << "operator+： counter: " << counter << " + " << from.counter << endl;
  cout << "operator+： addr: " << "(" << this << ")" << " <= " << "(" << &from << ")" << endl;
  return tmp;
}
```

* 将``*this``拷贝给局部变量``tmp``，得到结果。

```
copy=： counter: 1 <=copy 1
copy=： addr: (0041F5E4) <= (0041F724)
```

* 输出重载 +运算符函数的cout，得到结果。

```
operator+： counter: 1 + 3
operator+： addr: (0041F724) <= (0041F718)
```

* 我们都知道局部变量会在走出scope之后会销毁，所以语句``return tmp``必然不能直接返回原来的局部变量``tmp``。
* 所以编译器的操作是，先拷贝一份作为临时对象，得到如下结果。

```
copy=： counter: 4 <= 4
copy=： addr: (0041F640) <= (0041F5E4)
```

* 之后如我们所想的，``return tmp``后将销毁局部变量``tmp``，得到如下结果。

```
destructor counter = 4, addr=0041F5E4
```

> 临时变量将在完整表达式结束之后销毁。

* 就像上面的引用所说的，既然会在表达式``a = b + c``结束之后销毁临时变量，自然我们就不能将将结果给``a``用了（这里所说的直接结果意思就是临时变量所存储的内存块）。所以这里进行的是=赋值（事实上这也是必须的，因为这是赋值运算）。

* 所以调用函数``operator=``，得到结果

```
assign=： counter: 4 = 4
assign=： addr: (0041F730) <= (0041F640)
```

* 然后也是如我们所愿，在完整表达式后销毁临时变量。

```
destructor counter = 4, addr=0041F640
```

* 之后将退出``main()``函数，也就是临时变量a,b,c,d的生命周期结束，所以以构造函数相反的顺序进行调用析构函数。得到结果

```
destructor counter = 5, addr=0041F70C
destructor counter = 3, addr=0041F718
destructor counter = 1, addr=0041F724
destructor counter = 4, addr=0041F730
```
