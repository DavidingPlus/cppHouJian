# 侯老师c++课程

# 面向对象高级开发

## 4.10

### 1. Header头文件的防卫式声明

```c++
//complex.h
#ifndef __COMPLEX__
#define __COMPLEX__
//含义:如果程序没有定义过，那么定义出来，走主体内容；如果程序定义过，第二次include时，那么就不走，直接返回，不会有重复include的动作

/*
主体内容
*/

#endif
```

### 2. inline 函数

```c++
class complex{
  public:
    complex();
    
   	double real() const {return re; }//如果函数在类内声明并且定义完成，那么这个函数就是个inline函数
  private:
    double re,im;
};

inline double
real(const complex& x){
    return x.real();
}//inline的执行速度会快一点!
```

为了减少时间开销，如果在类体中定义的成员函数中不包括循环等控制结构，C++系统会自动将它们作为内置(inline)函数来处理。

C++要求对一般的内置函数要用关键字inline声明，但对类内定义的成员函数，可以省略inline，因为这些成员函数已被隐含地指定为内置函数。

如果成员函数不在类体内定义，而在类体外定义，系统并不把它默认为内置(inline)函数，调用这些成员函数的过程和调用一般函数的过程是相同的。如果想将这些成员函数指定为内置函数，应当用inline作显式声明。

如果函数太复杂，编译器没有办法把他看成 inline 函数!! 所以我们在类内实现只是为了建议编译器将其看成 inline 函数来提高效率，但是实际上是不是 inline 函数要看编译器，我们也不知道！

### 3.单例设计模式(构造函数在private部分)

例子：

```c++
//头文件 Stu.h
#ifndef __STU__
#define __STU__

#include <string>
using namespace std;
class Stu
{
public:
    static Stu &getInstance();//单例设计模式 在主程序当中只能使用一份这个类的数据 所有共享 所以静态变量放在堆区
    void setup() {}

private:
    Stu();
    Stu(int id, string name);

    int _ID;
    string _name;
};

Stu &Stu::getInstance()
{
    static Stu stu;
    return stu;
}

#endif
```

```c++
//主程序 main.cpp
#include <iostream>
#include "Stu.h"

int main()
{
    // 单例设计模式
    // 构造函数在private里面 整个类是放在static堆区的 所有用户只用一份这个类的数据
    auto stu = Stu::getInstance(); // 这样就创造出来了一个类 并且是静态变量!!!程序共享这一份

    return 0;
}
```

### 4.const 常量成员函数

```c++
//complex类
class complex
{
public:
  complex(double r = 0, double i = 0) : re(r), im(i) {}
    
  double real() const { return re; }
  double imag() const { return im; }//成员函数 不改变类成员属性的值 建议加上const修饰，换句话说就是拿数据

private:
  double re, im;
};
```

成员函数不改变类成员属性的值建议加上const修饰，换句话说就是拿数据.例如上面的real()和imag()都不改变成员属性的值，所以加上了const修饰。

在上面的例子当中，如果那两个成员函数不加上const，会出现什么情况呢？

```c++
//主函数
const complex c(1,2);//使用者定义这个类是不可以改变的
std::cout<<c.real();//打印real()
```

这是未加const的成员函数

```c++
  double real() { return re; }
  double imag() { return im; }
```

打印real()会出问题，因为使用者不想要改变c里面元素的值，但是这个成员函数在访问的时候不加const，编译器认为有可能会改变成员函数的值，这两者是相互矛盾的，所以会报错，所以需要加上const来保证是不会改变值的。

### 5.引用(指针常量 指针指向不可修改)

引用的本质: 是一个指针常量

为什么选择引用: 传递非常快，并且可以解决形参修改不改变实参的问题

```c++
int a=10;
//自动转换为 int* const ref = &a; 指针常量是指针指向不可改，也说明为什么引用不可更改
int& ref = a; 
ref = 20; //内部发现ref是引用，自动帮我们转换为: *ref = 20;
```

引用是不可修改的，因为引用本质是指针常量，该指针的值是不可修改的，也就是指向的地址区域(a)是不可以变动的，但是解引用修改指向区域的值是完全没有问题的。

所以考虑到这两个问题，在实际操作过程中尽量传入引用。

### 6.friend 友元

```c++
//complex类
class complex
{
public:
  complex(double r = 0, double i = 0) : re(r), im(i) {}
    
  double real() const { return re; }
  double imag() const { return im; }//成员函数 不改变类成员属性的值 建议加上const修饰，换句话说就是拿数据
    
private:
  double re, im;
  
  friend complex &__doapl(complex*, const complex& );//第二个参数传入另一个类对象的引用
  //friend友元表示另一个类对象可以访问本类当中的私有成员属性
};

inline complex &
__doapl(complex *ths, const complex &r)
{
  ths->re += r.re;//这里就可以直接访问本类的私有成员属性
  ths->im += r.im;
  return *ths;
}
```

### 重点：

相同class的各个objects互为friends 友元

还是上面的例子

```c++
//类内
int func(const complex& param){ return param.re + param.im;}
//这里为什么可以直接访问私有成员属性
//主函数
c2.func(c1);
```

两种理解:

1.相同class的各个objects互为友元，所以可以访问私有属性

2.私有成员属性可以类内访问，类外不可以访问，需要访问需要成员函数接口

### 7. return by reference

传送着无需知道接收者是以reference接受

```c++
inline complex&
complex::operator += (const complex &r){
    return __doapl(this,r);
}
```

实现 += 号的重载(复数)

为什么要返回 & ：因为不能创建新对象，我们的目的是修改原对象！！！

这样做可以实现连加操作 c1 += c2 + =c3

关于运算符重载返回reference还有一个例子

```c++
#include <iostream>
using namespace std;
#include <string>
#include <ostream>

class Stu
{
public:
    // 类内的函数会默认为内置inline函数
    Stu();
    Stu(int id, string name) : _ID(id), _name(name) {}

    int getID() { return this->_ID; }
    string getName() { return this->_name; }

private:
    int _ID;
    string _name;
};

//这种特殊的操作符重载只能写在全局，因为写在类里面无法达到 cout<<p 的效果
inline ostream & //返回ostream标准流的引用
//关于返回值 需要考虑连传的话需要返回引用!!!
operator<<(ostream &os, Stu &s)
{
    os << s.getID() << ' ' << s.getName() << endl;
    return cout;
}

int main()
{
    Stu s(1, "张三");
    cout << s;//这样可以实现连 <<

    return 0;
}
```

### return by value:

什么时候只能return by value?我们应该优先考虑return by reference，但是在这个函数返回值的时候需要创建一个新的对象的时候只能return by value

```c++
inline complex
operator+(const complex &x,const complex &y){
    return complex(real(x)+real(y),imag(x)+imag(y));
}

c2=c1+c2;//这行代码的意思是c1+c2创建出来一个新的对象赋值给c2!!!
```

## 4.12

### 1.class with pointer members 带有指针的类

比较经典的类就是string字符串类,必须有拷贝构造 copy ctor和拷贝赋值 copy op=

```c++
class String
{
public:
    String(const char *cstr = 0);
    // 只要类带指针，一定要重写以下两个函数!!!
    // 不重写的话编译器默认的构造函数是浅拷贝 两个指针指向同一块内存对象!!!
    // 当然这里的拷贝指的是深拷贝
    String(const String &str);            // 拷贝构造
    String &operator=(const String &str); // 拷贝赋值

    ~String();

    char *get_c_str() const { return this->_data; }

private:
    char *_data;
};
```

当然这里的拷贝指的是深拷贝!!!

Big Three:拷贝构造，拷贝赋值，析构函数!!!!!!

### 拷贝赋值:检测自我赋值

```c++
inline String &String::operator=(const String &str)
{
    // 深拷贝赋值 先把自身杀掉 然后重新创建
    // 检测自我赋值
    if (this == &str)
        return *this;

    delete[] this->_data;
    // 深拷贝
    this->_data = new char[strlen(str) + 1];
    strcpy(this->_data, str._data);
    return *this;
}
```

这里为什么要检测自我赋值：因为不检测自我赋值的话，如果使用者调用自我赋值的时候，第一步就会把唯一的自身这个_data杀掉，后续就没有办法进行了，会出现安全隐患!!!这也是为了安全和严谨性考虑的

### 2.一些对象的生命期

```c++
class complex{ };

complex c3(1,2);

int main(){
    complex c1(1,2);
    static complex c2(1,2);
    
    complex *c4=new complex(1,2);
    delete c4;
    
    return 0;
}
```

C++程序在执行时，将内存大方向划分为4个区域
代码区：存放函数体的二进制代码，由操作系统进行管理。
全局区：存放全局变量和静态变量以及常量。
栈区：由编译器自动分配释放，存放函数的参数值，局部变量等。
堆区：由程序员分配和释放，若程序员不释放，程序结束时由操作系统回收。

c1 :存放在栈当中，当作用域(这里是main函数)结束的时候就会被自动清理

c2 :静态变量，存放于全局静态区，当整个程序结束之后才会被释放

c3: 全局变量，存放于全局区，当整个程序结束之后才会被释放

c4: new出来的，动态分配内存，存放于堆区，注意new了之后记得在作用域结束之前将其delete掉

​      否则会出现内存泄露的问题，当作用域结束之后c4指针会被释放掉，但是他所指向的内存没有被释放!!!一般写析构函数解决这个问题

### 3. new: 先分配内存空间，再调用构造函数

![](C:\Users\XiaoLuBan\AppData\Roaming\Typora\typora-user-images\image-20230412173723916.png)

### delete: 先调用析构函数，再释放内存空间

![image-20230412174017480](C:\Users\XiaoLuBan\AppData\Roaming\Typora\typora-user-images\image-20230412174017480.png)

### array new 一定要搭配 array delete!!!

![image-20230412185656797](C:\Users\XiaoLuBan\AppData\Roaming\Typora\typora-user-images\image-20230412185656797.png)

写了 [] 的话编译器才会知道你不仅要删除p这个类对象指针,还要把这个p对象指针指向的数组元素给全部删除掉，因为 String* p既可以表示单个的类对象指针也可以表示类对象数组的首元素地址指针!!! 所以必须要要写 [] ,否则只会删除掉p[0]所对应的元素!!!

### 4. static 静态

静态变量和静态函数很特殊，具体看下面代码:

静态函数由于没有this指针，所以只能处理静态变量!!!

```c++
#include <iostream>
using namespace std;

class Account
{
public:
    static double _rate;                      // 静态成员变量一个程序只有一份，类内声明，类外初始化
    static void set_rate(const double &rate); // 静态成员函数，调用时可以声明类对象，可以调用作用域直接访问，类外实现，类外实现的时候不用加关键字static,但是要加作用域
};
double Account::_rate = 8.0;

void Account::set_rate(const double &rate)
{
    Account::_rate = rate;
}

int main()
{
    cout << Account::_rate << endl;
    Account::set_rate(7.0);
    cout << Account::_rate << endl;

    return 0;
}
```

### 进一步补充：把构造函数放在 private 里面，单例设计模式

```c++
class Stu
{
public:
    static Stu &getInstance() { return s; }
    void setup()
    {
        ; // 一系列的接口操作
    }

private:
    Stu();
    Stu(int id, string name);
    int _ID;
    string _name;

    static Stu s;
};
```

这里的就是把构造函数放在 private 当中，对外界的接口就是这个 getInstance()，这个函数返回静态变量 s ，只有一份，通过 setup() 接口进行对这个类内成员的访问和修改!!!

```c++
Stu::getInstance().setup();
```

优化的写法：

由于这里在构建类的时候就引入了静态成员变量，在没有调用的时候可能会导致资源浪费，所以我们将这个静态成员变量在静态函数中创建就好了，在调用的时候创建，然后生命周期一直持续到程序结束

```c++
static Stu& getInstance(){
	static Stu s;
	return s;
}
```

### 5.模板

### 类模板:

用的时候必须要明确指出里面参数的类型

```c++
template <typename T>
class complex{
  public:
    complex();
  private:
    T real;
    T imag;
};

int main()
{
    complex<int>c1();
    complex<double>c2();
}
```

### 函数模板:

用的时候不需要指明函数参数的类型,因为编译器会进行实参的推导

```c++
class stone
{
public:
    stone();
    stone(int w, int h, int weight) : _w(w), _h(h), _weight(weight) {}

    bool operator<=(const stone &sto) { return this->_weight <= sto._weight; }

private:
    int _w, _h, _weight;
};

// 全局函数 比大小
template <typename T>
inline T &
min(T &a, T &b)
{
    return a <= b ? a : b ?
}

int main()
{
    int a = 1, b = 2, c;
    stone a1(1, 2, 3), a2(4, 5, 2), a3;
    c = min(a, b);
    a3 = min(a1, a2);
}
```

### 6.命名空间 namespace

将自己写的东西封装在一个命名空间当中，可以防止与其他人名称一样功能不同的问题

```c++
namespace std{
	;
}
```

### 7. 复合 composition

简单来理解就是 一个类包含另一个类对象,本类可以调用另一个类的底层函数

```c++
//Adapter
template <class T>
class queue
{
protected:
	deque<T> c; //底层容器
public:
    //以下的操作全都是由c的底层函数执行
    bool empty()const{return c.empty();}
};
```

这里其实包含另一种设计模式: Adapter

deque是双端队列，queue是单端队列，显然deque的功能要比queue功能强大，他完全可以适配(Adapter)queue的功能，所以可以采用复合的方式，queue的成员函数调用deque中的部分成员函数来实现自己的功能!!!

用图可以这样表示:

![image-20230412202635207](D:\Typora\Images\image-20230412202635207.png)

### 复合下的构造和析构

构造: 构造由内而外 注意先调用的是内部的**默认**构造，编译器指定的，也符合我们的预期！

析构: 析构由外而内

这些是编译器帮我安排好的，上面是我们希望的设计

![image-20230412203303042](D:\Typora\Images\image-20230412203303042.png)

### 8. 委托 Delefgation.Composition by reference

把复合下面传入的参数类型改为指针!!!

![image-20230412204435460](D:\Typora\Images\image-20230412204435460.png)

### Handle/Body (pimpl)

图示的这种写法很有名，左边是用户看到的类，里面有调用的接口这些，右边是真正字符串的类，用来封装字符串的类型这些。

reference counting: 这种写法在这个特殊例子当中可以实现，用户创建了三个String对象，但是每个对象下面对应的 rep 指针指向的对象其实是一块内存，因为他们的字符串是一样的，这样就可以减少内存的开销。

### 9. 继承 inheritance

构造:由内而外 调用父类的**默认**构造函数，编译器指定的，也符合我们的预期！

析构:由外而内

![image-20230412210437066](D:\Typora\Images\image-20230412210437066.png)

### 虚函数:

override 覆写：注意只能用在虚函数这里

non-virtual 函数: 父类不希望子类重写(override)它

virtual 函数：父类希望子类重写(override)它，父类对它一般已经有默认定义

pure virtual 函数(纯虚函数)：父类希望子类一定要重写(override)它，父类对它没有默认定义，例子：父类是个抽象类

![image-20230412212341135](D:\Typora\Images\image-20230412212341135.png)

注意:子类调用父类的函数，如果子类当中对父类的函数进行了override，那么在调用到这个虚函数的时候就会调用子函数覆写的版本，从而实现我们的需求！！！

下面还有个例子:

```c++
#include <iostream>
using namespace std;

class Animal
{
public:
    virtual void speak() = 0;
};

class Cat : public Animal
{
public:
    virtual void speak() { cout << "喵喵喵" << endl; }
};

class Dog : public Animal
{
public:
    virtual void speak() { cout << "汪汪汪" << endl; }
};

int main()
{
    //用父类对象指针来接受子类对象 来达到子类调用父类对象成员函数的目的
    Animal *cat = new Cat();
    cat->speak(); // 喵喵喵

    Animal *dog = new Dog();
    dog->speak(); // 汪汪汪

    return 0;
}
```

  用父类对象指针来接受子类对象 来达到子类调用父类对象成员函数的目的，这样也可以实现多态.

### 10. 转换函数 conversion function

作用：可以用于类型的转换

重载 () 运算符 在括号的前面加上返回的类型，括号内不传参数，返回值由于在括号前面已经指定了，所以省略不写，编译器指定的

```c++
operator double(){
    return (double)this->_numerator / (double)this->_denominator;
}
```

整体的例子:

```c++
#include <iostream>
using namespace std;

class Fraction
{
public:
    Fraction(int num, int den = 1) : _numerator(num), _denominator(den) {}

    // 转换函数
    // 没有返回值，转化的类型在括号前面已经指定
    operator double() const
    {
        return (double)this->_numerator / (double)this->_denominator;
    }

private:
    int _numerator;
    int _denominator;
};

int main()
{
    Fraction f(3, 5);//这里首先创建了f对象，调用了构造函数
    double d = 4 + f;
    //f是个类对象，他怎么能直接和double类型相加呢？
    //如果写了 + 号运算符重载那么就直接调用即可，但是这里没写啊？
    //所以这里编译器就去找什么东西可以把f里面的参数变为double 就找到了转换函数 这里的f经过编译之后就返回分数的值 也就是0.6
    cout << d << endl;//4.6

    return 0;
}
```

注意main函数里面的细节!!!

### 11.函数对象(仿函数) -> 谓词

谓词:

1.函数指针作谓词

2.函数对象(仿函数)作谓词

```c++
#include <iostream>
using namespace std;
#include <vector>
#include <algorithm>

// 函数指针作谓词
bool Cmp(int val1, int val2)
{
    return val1 <= val2;
}

// 函数对象(仿函数)作谓词
class Fuck
{
public:
    bool operator()(int val1, int val2)
    {
        return val1 <= val2;
    }
};

int main()
{
    vector<int> nums{5, 3, 4, 2, 1};
    // sort(nums.begin(), nums.end(), Cmp);
    sort(nums.begin(), nums.end(), Fuck());
    for_each(nums.begin(), nums.end(), [&](int val)
             { cout << val << ' '; });
    cout << endl;

    return 0;
}
```

仿函数(函数对象): function like classes

在类里面重载 () 运算符

```c++
template <class T>
class identity{
public:
    const T& operator()(const T& x){return x;}
}
```

调用的时候 identity() 这样就是一个函数对象

标准库里面有很多仿函数，这些仿函数都继承了一些标准库里面的父类，这些父类大小为0，没有成员函数。(标准库)

## 4.13

### 1.模板补充: 成员模板 member template

到目前为止，三种模板: 类模板 函数模板 成员模板

![image-20230413163206633](D:\Typora\Images\image-20230413163206633.png)

注意看注释进行理解!!!

```c++
#include <iostream>
using namespace std;

// Base1 鱼类 Derived1 鲫鱼
// Base2 鸟类 Derived2 麻雀
// 相应的有继承关系
class Base1
{
};
class Derived1 : public Base1
{
};

class Base2
{
};
class Derived2 : public Base2
{
};

// 现在定义一个pair类
template <class T1, class T2>
struct Pair
{
    T1 _first;
    T2 _second;
    Pair();
    Pair(const T1 &a, const T2 &b) : _first(a), _second(b) {}

    // 注意这里有一个Pair的拷贝赋值
    template <class U1, class U2>
    Pair(const Pair<U1, U2> &p) : _first(p._first), _second(p._second) {}
    // 这里怎么理解
    // 如果传入的类型是 T1 鱼类 T2 鸟类
    // 然后在调用拷贝赋值的时候传入的类型是 U1 鲫鱼 U2 麻雀
    // 显然 鲫鱼是鱼类 麻雀是鸟类 所以是可以传入的
    // 这个成员模板需要满足的条件就是 p._first这里是可以给自身的成员属性 _first 进行赋值的,在这里满足的是继承的关系
};

int main() { return 0; }
```

这里其实就有考虑指针指向的问题

```c++
class Derived1 : public Base1 { };
Base1 *ptr=new Derived1;//这么写是完全ok的
```

还是考虑 Base1 是鱼类，Derived1 是 鲫鱼，然后我们用鱼类的指针去指向鲫鱼对象，这显然是可以的，因为鲫鱼很明显是鱼，所以这么写是完全ok的。

并且恰好这么写可以使得子类调用父类的虚函数来实现不同的虚函数功能。

### 2.命名空间

```c++
#include <iostream>
using namespace std;

namespace my1
{
    static void test()
    {
        cout << "I am in namespace my1" << endl;
    }
}

namespace my2
{
    static void test()
    {
        cout << "I am in namespace my2" << endl;
    }
}

int main()
{
    my1::test();
    my2::test();

    return 0;
}
```

两个命名空间，即使里面的函数名称一样，传入参数等等方面完全一样，甚至还是静态的，虽然静态的存放于全局静态区只有一份，但是这里用了两个不同的命名空间将他们分割开来，这样就导致两个函数本质上是不同的，从下面的使用就可以看出来了。

### 3. explicit

non-explicit-one-argument ctor(构造函数)

```c++
class Fraction
{
public:
    // non-explicit-one-argument ctor
    Fraction(int num, int den = 1) : _numerator(num), _denominator(den) {}

    //这是上面提到的转换函数
    operator double() const
    {
        return (double)this->_numerator / (double)this->_denominator;
    }

private:
    int _numerator;
    int _denominator;
};
```

我们先将转换函数去掉，重载加号运算符

```c++
class Fraction
{
public:
    // non-explicit-one-argument ctor
    Fraction(int num, int den = 1) : _numerator(num), _denominator(den) {}
	
    //重载加号运算符
    Fraction operator+(const Fraction& f){ ; }
private:
    int _numerator;
    int _denominator;
};

int main(){
    Fraction f(3,5);
    Fraction d=f+4;
    //到这里的时候编译器发现f和4没办法直接相加 即使写了重载 因为需要传入Fraction类型
    //但是编译器看构造函数 默认值den=1 意思是可以传入一个参数，这就和现在的4很贴切了
    //所以编译器会将4转化为Fraction类对象和f进行相加得到对象d
}
```

如果两个同时存在

```c++
class Fraction
{
public:
    // non-explicit-one-argument ctor
    Fraction(int num, int den = 1) : _numerator(num), _denominator(den) {}
    
    //conversion function
    operator double() const
    {
        return (double)this->_numerator / (double)this->_denominator;
    }
	
    //重载加号运算符
    Fraction operator+(const Fraction& f){ ; }
private:
    int _numerator;
    int _denominator;
};

int main(){
    Fraction f(3,5);
    Fraction d=f+4;
    //按照上面的思路是一种走法
    //但是有了转换函数之后编译器发现,f可以先转化为double数字再和4求和，求完和之后再转化为Fraction对象，这就是另一种思路了
    //所以 二义性 报错
}
```

现在如果加上关键字 explicit 呢？

```c++
class Fraction
{
public:
    // explicit-one-argument ctor
    explicit Fraction(int num, int den = 1) : _numerator(num), _denominator(den) {}
    //explict关键字的含义 防止类构造函数的隐式自动转换
    //就是说这里由于只需要传入一个参数，所以编译器很可能会把数字隐式转化为Fraction对象
    //但是加上了explict之后,明确指出不要让编译器这么干，要生成Fraction对象只能显式调用构造函数!!!!
    
    //conversion function
    operator double() const
    {
        return (double)this->_numerator / (double)this->_denominator;
    }
	
    //重载加号运算符
    Fraction operator+(const Fraction& f){ ; }
private:
    int _numerator;
    int _denominator;
};

int main(){
    Fraction f(3,5);
    Fraction d=f+4;//这里仍然会错，因为4不会被转化为Fraction了，也就没有办法直接相加
    double e=f+4;//这里显然就可以了，因为存在转换函数
}
```

这个关键字 explicit 绝大部分都是用在构造函数前面来防止其他类型的隐式转换!!!!

### 4. pointer-like classes 关于智能指针和迭代器

### 智能指针: 用一个类来模拟一般指针的作用

```c++
#include <iostream>
using namespace std;

struct Foo
{
    void method(void);
};

template <class T>
class shared_ptr
{
public:
    shared_ptr(T *p) : _px(p) {}

    //智能指针必然需要重载这两个运算符
    T &operator*() const
    {
        return *(this->_px);
    }

    T *operator->() const
    {
        return this->_px;
    }

private:
    T *_px;
};

int main()
{
    shared_ptr<Foo> sp(new Foo);
    Foo f(*sp);
    sp->method();//这个就相当于 _px->method();

    return 0;
}
```

### 迭代器: iterator 其本质也是一种智能指针

```c++
#include <iostream>
using namespace std;
#include <list>
#include <algorithm>

void test()
{
    list<int> l{5, 2, 4, 3, 1};
    //方法一
    //注意这个迭代器类型怎么写的
    for (list<int>::iterator iter = l.begin(); iter != l.end(); ++iter)
        cout << *iter << ' ';
    cout << endl;

    // 方法2
    for_each(l.begin(), l.end(), [&](int val)
             { cout << val << ' '; });
    cout << endl;
}

int main()
{
    test();
    return 0;
}
```

### 5. specialization 模板特化

对于一个泛型模板，我们调用的时候里面的接口都是一样的。但是如果我们发现有的特殊的类型在某个函数下有更加好的实现方法，这个时候就可以用模板特化来操作了。可以类比子类继承父类(抽象类)的虚函数，特殊化实现，本质是一样的。

![image-20230413211543101](D:\Typora\Images\image-20230413211543101.png)

```c++
#include <iostream>
using namespace std;

template <class Type>
struct Fuck
{
};

// 模板特化
template <>
struct Fuck<int>
{
    int operator()(int val) const { return val; }
};

//注意模板特化的语法
template <>
struct Fuck<string>
{
    string operator()(string ch) const { return ch; }
};

template <>
struct Fuck<double>
{
    double operator()(double val) const { return val; }
};

int main()
{
	// 匿名对象
    cout << Fuck<int>()(1) << endl;
    cout << Fuck<string>()("fuck") << endl;
    cout << Fuck<double>()(3.14) << endl;

    return 0;
}
```

模板特化语法第一行要加上，第二行就是具体类型类的具体操作

```c++
template <>
class Fuck<type>{
  	;  
};
```

### partial specialization 模板偏特化

个数的偏

```c++
template<typename T,typename Alloc= ...>
class Vector{
    ...
};

//模板偏特化 就只特定其中的某个或者某几个元素 其实还是一个模板
template<typename Alloc= ...>
class Vector<bool,Alloc>{
    ...
};
```

范围的偏

```c++
#include <iostream>

using namespace std;

template <typename T>
struct TC // 泛化的TC类模板
{
    void functest()
    {
        cout << "泛化版本" << endl;
    }
};
// 偏特化：模板参数范围上的特化版本
template <typename T>
struct TC<const T> // const的特化版本
{
    // 对特化版本做单独处理
    void functest()
    {
        cout << "偏特化const版本" << endl;
    }
};
template <typename T>
struct TC<T *> // T* 的特化版本
{
    void functest()
    {
        cout << "const T*特化版本" << endl;
    }
};
template <typename T>
struct TC<T &> // T& 的特化版本
{
    void functest()
    {
        cout << "T &左值引用特化版本" << endl;
    }
};

template <typename T>
struct TC<T &&> // T&& 的特化版本
{
    void functest()
    {
        cout << "T &&右值引用特化版本" << endl;
    }
};

void test()
{
    TC<double> td;
    td.functest();

    TC<const double> td2;
    td2.functest();

    TC<double *> tpd;
    tpd.functest();

    TC<const double *> tpd2;
    tpd2.functest();

    TC<int &> tcyi;
    tcyi.functest();

    TC<int &&> tcyi2;
    tcyi2.functest();
}

int main()
{
	test();
    //泛化版本
	//偏特化const版本
	//const T*特化版本
	//const T*特化版本
	//T &左值引用特化版本
	//T &&右值引用特化版本

    return 0;
}
```

# C++标准库 体系结构与内存分析

## 第一讲：STL标准库和泛型编程

## 4.14

### 1.STL 体系结构

六大部件: 容器 分配器 算法 迭代器 适配器 仿函数

容器：各种数据结构

算法：algorithm

迭代器：泛型指针，重载了 * -> ++ --操作的类

仿函数：从实现的角度看是重载了 operator() 的类

适配器：一种修饰容器，仿函数或者迭代器接口的东西

分配器：负责空间的配置和管理

![image-20230414100854356](D:\Typora\Images\image-20230414100854356.png)

 以下有一个程序的例子几乎包含了所有的元素：

<img src="D:\Typora\Images\image-20230414114802698.png" alt="image-20230414114802698" style="zoom: 67%;" />

```c++
#include <iostream>
#include <vector>
#include <algorithm>
#include <functional>
using namespace std;

int main()
{
    int ia[6] = {27, 210, 12, 47, 109, 83};
    vector<int, allocator<int>> vi(ia, ia + 6);

    cout << count_if(vi.begin(), vi.end(), not1(bind2nd(less<int>(), 40))) << endl;
    return 0;
}
```

下面来解释这里面所用到的东西:

1. vector是容器，这里的用法和一般的使用方法不同，这里给出了分配器模板的指定参数

2. count_if第三个参数，本意是想比较迭代器 * iter 和40 的大小，然后使用的仿函数，但是less<int>()这个系统自带的仿函数的实现是这样的

   ```c++
     template<typename _Tp>
       struct less : public binary_function<_Tp, _Tp, bool>
       {
         _GLIBCXX14_CONSTEXPR
         bool
         //从这里可以看出他需要两个参数
         operator()(const _Tp& __x, const _Tp& __y) const
         { return __x < __y; }
       };
   ```

   一般仿函数的用法:

   ```c++
   vector<int>v{5,3,4,6,8};
   sort(v.begin(),v.end(),less<int>());//仿函数作谓词
   ```

   所以这里用 **bind2nd** 将迭代器和40绑定在一起，也叫 function adapter(binder)。

   然后最外面 **not1** 一样的，将条件取反，所以求的就是大于等于40的元素个数了,function adapter(negator)

### 2.基于范围的 for 语句

个人感觉有点像python里面的 for i in range()

```c++
#include <iostream>
using namespace std;
#include <vector>

// 遍历vector容器
template <typename Type>
void print(vector<Type> container)
{
    for (auto elem : container)
        cout << elem << ' ';
    cout << endl;
}

int main()
{
    print(vector<int>{1, 5, 6, 9, 7, 5, 3, 10});

    vector<double> nums{1.1, 2.5, 6.33, 15.66, 1.44, 2.52};
    print(nums);

    // 稍微修改一下
    // 注意传入引用才能修改实参!!!!!
    for (auto &elem : nums)
        elem -= 1; // 减一
    print(nums);

    return 0;
}
```

### 3.容器的结构和分类

总体来讲分为两类:

序列容器 Sequence Containers

关联式容器 Associate Containers 关联式容器采用键值对的方式存储数据，因此这一种容器查找元素的效率最高，最方便

无序容器 Unordered Containers 是关联式容器的一种，c++11新出的

相比于关联式容器的特点：

- 无序容器内部存储的**键值对是无序**的，各键值对的存储位置取决于该键值对中的键
- 和关联式容器相比，无序容器擅长**通过指定键查找对应的值**（平均时间复杂度为 O(1)）；
- 但对于使用迭代器遍历容器中存储的元素，无序容器的执行效率则不如关联式容器。

![image-20230414161555294](D:\Typora\Images\image-20230414161555294.png)

### HashTable Separate Chaining

![image-20230414171828137](D:\Typora\Images\image-20230414171828137.png)

## 4.15

### 1.一些容器的使用

#### 1.1 Sequence Containers 序列容器

#### array(c++11)

array是STL自带的数组类，其本质就是一个固定大小的数组，里面存放的元素类型由用户指定

```c++
void test()
    {
        srand(time(NULL));

        const size_t _size = 100;
        array<int, _size> arr;

        for (int i = 0; i < _size; ++i)
            // 随机数 0-100
            arr[i] = rand() % 101;
        // 打印一些信息
        cout << "arr.size()= " << arr.size() << endl;
        cout << "arr.front()= " << arr.front() << endl;
        cout << "arr.back()= " << arr.back() << endl;
        cout << "arr.data()= " << arr.data() << endl;
    	cout<< " &arr[0]= " << &arr[0] << endl;//第四行和第五行得到的结果是一样的
    }
```

array封装了固定长度数组的一些函数接口，其中data()函数是得到这个数组的首元素地址，也就是第五行，所以四五行结果相同

#### vector

```c++
void test(int length)
    {
        srand(time(NULL));

        vector<int> v;
        for (int i = 0; i < length; ++i)
            v.push_back(rand() % 101);
        // 打印
        cout << "v.size()= " << v.size() << endl;
    	cout << "v.max_size()= " << v.max_size() << endl;//这里的max_size()是指vector容器能装下的最大的大小
        cout << "v.front()= " << v.front() << endl;
        cout << "v.back()= " << v.back() << endl;
        cout << "v.data()= " << v.data() << endl;
        cout << "&v[0]=" << &v[0] << endl;
        cout << "v.capacity()= " << v.capacity() << endl;
    }
```

这里得到我指定size是10000的时候，capacity是16384，恰好是2的14次方，那么为什么是这样呢？

vector当空间不够的时候如何开辟空间：**2倍开辟**!!!

比如现在有2个元素，想要放入第三个，空间不够会新开辟，那么新开辟之后vector的空间大小是4

即:

```c++
// v.size() == 3
// v.capacity() == 4
```

所以当 size==10000的时候，capacity为16384也不奇怪了

并且**内存开辟成长机制**：

当空间不够的时候，vector容器会去内存中找另一块空间是现在2倍的空间，重新开辟内存，并且把现在的内存释放掉，把现在的数据迁移到新的2倍内存当中去!!!!

#### list

![image-20230415143355398](D:\Typora\Images\image-20230415143355398.png)

list是个双端循环链表，注意不仅是双向链表，还是循环的!!!!

```c++
    void test(int length)
    {
        srand(time(NULL));

        list<int> l;
        for (int i = 0; i < length; ++i)
            l.push_back(rand() % 101);//list 提供了back和front两种插入方法,因为有begin()和end()迭代器
        cout << "l.size()= " << l.size() << endl;
        cout << "l.max_size()= " << l.max_size() << endl;
        //注意 front和back的三种得到方式
        cout << "l.front()= " << l.front() << endl;
        cout << "l.front()= " << *l.begin() << endl;
        cout << "l.front()= " << *(++l.end()) << endl;

        cout << "l.back()= " << l.back() << endl;
        cout << "l.back()= " << *(--l.end()) << endl;
        auto iter = --l.begin();
        cout << "l.back()= " << *(--iter) << endl;
    }
```

得到首部和尾部的方式:

1.内置函数 front()和back()

2.使用迭代器 begin() 和 end()

注意这里的迭代器是首闭尾开的形式,就是begin() 指向的第一个元素，end() 指向的最后一个元素的下一个没有值的内存空间,所以上一个区域就是最后一个元素的值 --end()

由于这个双端链表在内存中最后一个元素的末尾还多了一块未分配值的空间，考虑到是循环的，所以++end()就代表第一块元素的空间

但是，由于**大部分的迭代器没有重载 + 和 - 运算符(vector容器有)**，那么在求的时候不能直接用这两个符号，而得使用重载的++ 和 -- 运算符!!!!!!!

#### forward_list(c++11)

![image-20230415143409298](D:\Typora\Images\image-20230415143409298.png)

本质就是一个单向链表，非循环链表

```c++
    void test(int length)
    {
        srand(time(NULL));

        forward_list<int> fl;
        for (int i = 0; i < length; ++i)
            fl.push_front(rand() % 101); // 只提供头插法,因为尾插法太慢了
        cout << "fl.max_size()= " << fl.max_size() << endl;
        cout << "fl.front()= " << fl.front() << endl;
        cout << "fl.front()= " << *(fl.begin()) << endl;
        // cout << "fl.back()= " << *(--fl.end()) << endl; // error 没有重载 -- 运算符 只重载++运算符
        // 不存在 fl.back() 接口
        // 也不存在fl.size()接口
    }
```

注意这个单向链表只有头插法，原因是尾插法每次都要遍历到最后，太慢了，头插法效率更高,所以这个容器也**不存在back()函数接口**

这个单向链表begin()和end()迭代器都存在，但是大部分的时候遍历是使用begin()迭代器，因为**只重载了 ++ 运算符，未重载 -- 运算符!!!**

还有一点与list不同,这个容器**没有 size() 接口**!!!我也不知道为什么，标准库没有提供这个接口

#### slist

这个容器的实现和 forward_list 相同，只不过这个容器是c++之前就有了，而 forward_list 是c++11新提出的

```c++
//头文件
#include <ext\slist>
```

#### deque

双端开口队列

![image-20230415145105415](D:\Typora\Images\image-20230415145105415.png)

但是在实现的时候采用的是**分段连续**的机制：

它真实的结构如下：

<img src="D:\Typora\Images\image-20230415145711652.png" alt="image-20230415145711652" style="zoom:67%;" />

到99的时候，迭代器进行++的操作，需要进行判断走到了这一块内存的末尾，需要移步到下一个buffer的起始位置，也就是0，这就需要对++和--操作符进行重载!!!

```c++
    void test(int length)
    {
        srand(time(NULL));
        deque<int> d;
        for (int i = 0; i < length; ++i)
            d.push_back(rand() % 101);
        cout << "d.size()= " << d.size() << endl;
        cout << "d.max_size()= " << d.max_size() << endl;
        cout << "d.front()= " << d.front() << endl;
        cout << "d.back()= " << d.back() << endl;
    }
```

和前面的使用没有大区别,只是**deque不是循环的，而是两端延升的**

####  stack

deque的功能可以实现stack的所有功能，可以用复合composition的方式来实现stack类

<img src="D:\Typora\Images\image-20230415150557574.png" alt="image-20230415150557574" style="zoom: 67%;" />

#### queue

同stack，略

<img src="D:\Typora\Images\image-20230415150706680.png" alt="image-20230415150706680" style="zoom:67%;" />

#### 1.2 Associate Containers 关联式容器

关联式容器每个元素都存在 key 和 value，这样才能使得查询效率大大提高

#### 红黑树

#### Multiset 和 set

<img src="D:\Typora\Images\image-20230415154938692.png" alt="image-20230415154938692" style="zoom:67%;" />

这种容器的 **key 和 value 值相同!**!!!!

这两个容器的底层都是用**二叉树**(红黑树)实现的，元素在插入的时候都会被**进行自动排序(从小到大)**，唯一的不同点是，Multiset允许插入重复的元素，而set不允许出现重复的元素

```c++
    void test(int length)
    {
        srand(time(NULL));
        set<int> s;
        for (int i = 0; i < length; ++i)
            s.insert(rand());//注意插入的接口是insert()
        cout << "s.size()= " << s.size() << endl;
        cout << "s.max_size()= " << s.max_size() << endl;
        // s.begin() s.end() 存在接口
        // 迭代器存在，因为底层是用二叉树实现的，并且进行了自动排序，所以肯定可以遍历，这个就是学底层的时候该考虑的问题
    }
```

**注意：Multiset 和 set 不能使用 [ ] 来做下标访问!!!!!** 

第一，底层是红黑树，第二，标准库未重载 [ ] 符号!!!



#### Multimap 和 map

这种容器就有分别的 **key 和 value** 了，两者不一定相同，每个元素都是有两个元素!!!底层是用**红黑树**实现的!!!!!

插入元素的时候会**按照key进行排序(从小到大)**

<img src="D:\Typora\Images\image-20230415154949602.png" alt="image-20230415154949602" style="zoom:67%;" />

```c++
    void test(int length)
    {
        srand(time(NULL));
        map<int, int> m;
        for (int i = 0; i < length; ++i)
            m.insert(pair<int, int>(i, rand()));

        cout << "m.size()= " << m.size() << endl;
        cout << "m.max_size()= " << m.max_size() << endl;
        // m.begin() m.end() 存在
    }
```

同set一样，由于底层是用红黑树实现的，所以查询的时候也有迭代器接口，并且效率很高

map不能有重复的，这里的**重复判断是看 key** !!!!!! 两个value相同但是key不同是合法的!!! Multimap 就可以有相同的key!!!

**注意：Multimap 和 map 不能使用 [ ] 来做下标访问!!!!!**

原因类似见上



#### 哈希表(其实原本名字前缀是hash,现在改名叫unordered)

#### unordered_multiset 和 unorder_set

**key和value值相同**，与上面不同的是这个本质是用**哈希表**实现的!

<img src="D:\Typora\Images\image-20230415160759007.png" alt="image-20230415160759007" style="zoom:67%;" />

```c++
void test(int length)
    {
        srand(time(NULL));
        unordered_set<int> us;
        for (int i = 0; i < length; ++i)
            us.insert(rand());

        cout << "us.size()= " << us.size() << endl;
        cout << "us.max_size()= " << us.max_size() << endl;
        cout << "us.bucket_count()= " << us.bucket_count() << endl;//篮子的大小
        cout << "us.load_factor()= " << us.load_factor() << endl;//装载因子=元素个数/篮子个数
        cout << "us.max_load_factor()= " << us.max_load_factor() << endl;
        cout << "us.max_bucket_count()= " << us.max_bucket_count() << endl;//最大的篮子个数,和最大元素个数相同
    }
```

unordered_set 通过一个**哈希函数**，将对象的值映射到一个数组下标，这个数组下标对应的是unordered_set中的一个“桶”，表示所有可以映射到这个下标的元素的集合，通常用链表表示。

这个vector数组我一般形象的称其为**篮子**。

篮子扩充机制：**当元素个数size()不断增加，达到篮子个数bucket_count()的时候，vector容器进行近似2倍的扩充**，具体略

所以，**篮子个数一定大于元素个数**!!!!



#### unordered_multimap 和 unordered_map

与上面的大概相同，不同的是，传入的数据类型是一个**键值对 pair<keyType,valueType>**

其他略，具体实现后面再谈

### 2.使用分配器 allocator

这部分先了解怎么使用分配器，后面会有专题来讲解分配器的原理

![image-20230415165550803](D:\Typora\Images\image-20230415165550803.png)

**虽然分配器有申请内存空间并且归还内存空间的接口，但是不建议直接使用分配器，因为这样分配器的负担太重了。而应该去使用容器，让分配器给容器分配空间，这样的效率会高很多!!!**

## 第二讲：分配器 迭代器

### 3. OOP(面向对象编程)和GP(泛型编程)

OOP将 data 和 methods 结合在一起,GP却将他们两个分开来

采用GP:

1.容器Containers和算法Algorithms可以各自闭门造车，通过迭代器Iterator连接起来即可

2.算法ALgorithms通过迭代器Iterator确定操作范围，并通过Iterator取用Container元素

### 4.随机访问迭代器

随机访问迭代器 RandomAccessIterator：能够随机访问容器中的任一元素，例如vector单端数组

这样的迭代器可以进行+ -号的运算，例如:

```c++
auto mid=(v.begin()+v.end())/2 //随访访问迭代器才可以这么操作
```

提到这里，就不得不提一下算法库里的全局函数 sort() 了

**sort()函数内部实现的机制调用了随机访问迭代器，进行了+-的运算，所以能调用的前提只能是随机访问迭代器，比如vector,deque**

**所以由于list不满足这个迭代器，所以他不能调用全局sort函数，只能用自己类实现的sort函数，即 l.sort()**

```c++
#include <iostream>
using namespace std;
#include <list>
#include <algorithm>

template <typename Type>
void print(list<Type> &l)
{
    for_each(l.begin(), l.end(), [&](auto val)
             { cout << val << ' '; });
    cout << endl;
}

int main()
{
    list<int> l;
    for (int i = 0; i < 10; ++i)
        l.push_back(9 - i);
    print(l);
    // sort(l.begin(), l.end(), less_equal<int>());//用不了 因为他不是RandomAccessIterator Error!!!
    l.sort(less_equal<int>());
    print(l);

    return 0;
}
```

### 5.GP 泛型编程举一个例子

```c++
#include <iostream>
using namespace std;

namespace fuck
{
    template <typename Type>
    inline const Type &max(const Type &a, const Type &b)
    {
        return a < b ? b : a;
    }

    template <typename Type, class functor>
    inline const Type &max(const Type &a, const Type &b, functor &cmp)
    {
        return cmp(a, b) ? b : a;
    }
}

bool strCmp(const string &s1, const string &s2)
{
    return s1.size() < s2.size();
}

void test()
{
    cout << "max of zoo and hello: " << fuck::max(string("zoo"), string("hello")) << endl;         // zoo
    cout << "max of zoo and hello: " << fuck::max(string("zoo"), string("hello"), strCmp) << endl; // hello
}

int main()
{
    test();
    return 0;
}
```

这个例子很简单，就不多做解释了

## 4.16

### 1.重载new运算符 operator new

![image-20230416094923459](D:\Typora\Images\image-20230416094923459.png)

可以看到，在c++当中，new关键字在调用之后都会走到c语言的malloc函数来分配内存，然后malloc函数分配内存的机制就是上面那个内存块所示

<img src="D:\Typora\Images\image-20230416095052581.png" alt="image-20230416095052581" style="zoom: 67%;" />

size所包含的内容才是我想要的存放数据的内容部分，但是malloc会给我们开辟比size更大的空间，这些在另一门课里面会具体谈到。

### 2.分配器 allocators

### VC6 allocator

VC6里面的分配器具体实现如下图：

![image-20230416095559995](D:\Typora\Images\image-20230416095559995.png)

分配器当中最重要的就是 **allocate 函数 和 deallocate 函数**

从上图中可以看出，VC提供的分配器在分配的时候，allocate函数在调用的时候会调用 new 关键字，也就是会调用 malloc 函数

在释放内存的时候调用deallocate 函数，也就是调用delete关键字，最终就是调用free 函数

结论：

![image-20230416095953933](D:\Typora\Images\image-20230416095953933.png)

对于这个allocator，如果硬要用的话可以这么使用

```c++
    // 建立分配器
    int *p = allocator<int>().allocate(512, (int *)0);
    // 归还
    allocator<int>().deallocate(p, 512);//在归还的时候还需要之前的大小，所以非常不好用!!!
```

### BC++ allocator

BC5 STL中对分配器的设计和VC6一样，没有特殊设计

![image-20230416101021861](D:\Typora\Images\image-20230416101021861.png)

操作略

### GCC2.9 allocator

和前面两个一样，也没有特殊设计，就是简单的调用malloc 和 free分配和释放内存

![image-20230416101451151](D:\Typora\Images\image-20230416101451151.png)

右边这一段注释的意思就是虽然这里实现了符合标准的allocator，但是他自己的容器从来不去用这些分配器，这些分配器都有一个致命的缺点，就是因为本质是在调用mallloc和free函数，根据前面的内存分配机制很容易看出会产生很多的其他空间，从而被浪费，所以开销相对比较大，一般不用

### GCC2.9 自己使用的分配器：alloc(不是allocator!!!)

这个分配器想必比allocator要好用的多

![image-20230416102010067](D:\Typora\Images\image-20230416102010067.png)

其具体实现如下：

![image-20230416102442772](D:\Typora\Images\image-20230416102442772.png)

**怎么实现的呢？设计了16个链表，每个链表管理特定大小的区块，#0管理8个字节，#1管理16，以此类推，最后#15管理168个字节。所有使用这个分配器的元素的大小会被调整到8的倍数，比如50的大小会被调整到56。如果该链表下面没有挂内存块，那么会向操作系统用malloc函数申请一大块内存块，然后做切割，之后分出一块给该容器，用单项链表存储。这样的好处是避免了cookie的额外开销，减少了内存浪费。**

这个东西的缺陷到内存管理里面去讲。

### GCC4.9 使用的分配器：allocator(不是alloc!!!)

![image-20230416105043344](D:\Typora\Images\image-20230416105043344.png)

发现 allocator 是继承的父类 new_allocator

![image-20230416105348361](D:\Typora\Images\image-20230416105348361.png)

**发现GCC4.9使用的分配器和之前的分配器没什么区别，没有特殊设计，就是调用的malloc函数和free函数，不知道为什么(这个团队没解释)**

**但是但是！GCC4.9里面的__pool_alloc就是GCC2.9里面的alloc,非常好用的那个**

![image-20230416110401962](D:\Typora\Images\image-20230416110401962.png)

### 3.容器之间的关系

容器与容器之间的关系基本上都是复合的关系，比如set/multiset和map/multimap底层都是由rbtree红黑树实现的等等，具体见下图

![image-20230416154815162](D:\Typora\Images\image-20230416154815162.png)

### 3.区别size()和sizeof()

以容器list为例，list.size()和sizeof(list)是没有直接的大小联系的（单项链表forward_list不存在size()方法）

```c++
//在vscode+linux g++编译器中
    list<char> l;
    for (int i = 0; i < 26; ++i)
        l.push_back('a' + i);

    cout << l.size() << endl;  // 26
    cout << sizeof(l) << endl; // 24
```

l.size()指的是容器中存放的元素个数；sizeof(l)指的是需要形成list这个容器需要这个类所占的内存有多大，list类里面不仅存放了链表的指针，还有其他的成员属性来配合控制这个容器的运行.所以sizeof(l)和这个元素的个数一般没有关系。

### 4.深入探索 list

GCC2.9是这样写的

![image-20230416160403390](D:\Typora\Images\image-20230416160403390.png)

可以看出，list里面非常重要的一点设计就是**委托**设计，**即list本身的类并不是实际的双向链表，用户所能操作的这个类其实可以看作双向链表的管理类，里面有成员函数，迭代器，还有一根指向双向链表的指针，实际的双向链表结构就如上面所示,__list_node，这个才是真正的存储结构**，这也是为什么sizeof()和size()是不一样的，因为设计者很好的把二者分开了，使得用户和写代码的人都能很好的管理自己的部分。

由于list的存储是不连续的，所以相应的他的迭代器也需要是智能指针，需要重载++和--运算符，(注意list的迭代器不是随机访问迭代器，所以不能使用+ -号运算符，也不能使用算法库的函数sort()，而需要使用自带的函数sort() )那么就应该是一个类了。**进而推得所有的容器(除了vector和array)的迭代器，最好都写成一个类来实现。**

### list的迭代器

这个迭代器最重要的就是重载 ++ 运算符，也就是前置++和后置++

![image-20230416163304200](D:\Typora\Images\image-20230416163304200.png)

前置++和后置++的区别就是后置++的参数列表里面会有一个占位符int来表示他是后置++

```c++
//前置++
self& operator++(){
    node=(link_type)((*node).next);
    return *this;
}
//这里的self是指迭代器这个类，是个别名。这个实现还是比较容器理解的
//注意返回的是迭代器新的位置所以可以返回引用类型
```

```c++
//拷贝构造
__list_iterator(const iterator& x):node(x.node){}
```

```c++
//重载*
reference operator*()const{return (*node).data;}
```

```c++
//后置++
self operator++(int){
	self tmp=*this;
    //这一步仔细看看，可能会调用拷贝构造(=)，可能也会调用*重载，但是*重载明显不符合要求
    //再加上 = 在前面，所以调用的是拷贝构造，*this已经被看作拷贝构造的参数，拷贝出来了一个新的原位置迭代器!!!!
	++*this;
    //前置++
	return tmp;
}
//注意由于需要返回原位置的迭代器，而现在的迭代器已经改变了，所以最好新创建一个，return by value
```

### 关于为什么后置++不能返回引用，比较有说服力的还有如下的原因：

```c++
int i=2,j=2;
cout<< ++++i << j++++ << endl;
```

**我们尝试将i和j分别进行前置和后置++分别加两次，c++的编译器允许前置++连续加，但是不允许后置++连续加，我们知道想要连续加的条件就是要返回引用继续修改原本的值，所以既然不允许连续后置++，那么就return by value，直接创建一个新对象**

然后类在重载这两个运算符的时候也会向编译器自带的规则看起，也不允许后置++连续加，所以就只能return by value

### 关于 * 和 & 运算符的重载

```c++
#include <iostream>
using namespace std;
#include <list>

class fuck
{
public:
    void print() { cout << "hello" << endl; }
    fuck(int data = 0) : _data(data) {}//构造函数，可以将int类型转化为类对象

    int getData() { return this->_data; }

private:
    int _data;
};

int main()
{
    list<fuck> l{1, 2, 3, 6, 5, 8, 8, 9};
    auto iter = l.begin();//取得首个元素迭代器
    //*iter取得的是 fuck 对象,iter->取得的是 fuck对象指针
    //对于简单的类型 iter->没什么作用，比如int，这时候*iter就代表了value，但是对于类对象那就不一样了
    cout << (*iter).getData() << endl;
    iter->print();

    return 0;
}
```

*和->的具体实现,Type是类的类型

```c++
typedef Type& reference;
typedef Type* pointer;
//* 返回的是类对象
reference operator*(){
    return *(this->node);//node是迭代器当中存放的链表指针对象
}
//-> 返回的是类对象指针
pointer operator->(){
    return &(operator*());
}
```

### G4.9 和 G2.9的区别

具体区别就如下图所示：

![image-20230416171855901](D:\Typora\Images\image-20230416171855901.png)

在4.9版本当中

![image-20230416174305071](D:\Typora\Images\image-20230416174305071.png)

**刻意在list尾端加上一段空白的区域来复合STL迭代器前闭后开的特征!!!但是相应的这个设计的复杂度又大大增加了。**

## 4.19

### 1.迭代器的设计原则

Iterator需要遵循的原则：在调用算法的时候，iterator作为中间桥梁连接容器和算法，所以算法需要知道Iterator的很多东西

**算法需要知道迭代器的必要信息，进而决定采取最优化的动作**

![](D:\Typora\Images\image-20230419170456164.png)

在C++标准库当中设计出五种标准类型:

```c++
iterator_category;
//迭代器的分类：只读，只写，允许写入型算法在迭代器区间上进行读写操作(Forward Iterator)，可双向移动，Random Access Iterator
difference_type;//用来表示迭代器之间的距离，也可以用来表示一个容器的最大容量
value_type;//迭代器所指对象的类型

reference;
pointer;//最后两个基本没用到
```

这五种类型被称作 **associated type 相关类型**；迭代器本身必须定义出来，以便回答算法

![image-20230419170815213](D:\Typora\Images\image-20230419170815213.png)

比如我自己写一下链表list的五个相关类型

```c++
//GCC4.9版本
template <class _Type>
struct List_Iterator
{
    typedef std::bidirectional_iterator_tag _iterator_category;
    typedef ptrdiff_t _difference_type;
    typedef _Type _value_type;
    typedef _Type& _reference;
    typedef _Type* _pointer;
};
```

**引出问题：算法调用的时候传进去的可能不是个迭代器，可能是个指针，这个时候该怎么办呢？**

当然，指针是个**退化**的迭代器!!!

### 2. Traits 萃取机

![image-20230419190233258](D:\Typora\Images\image-20230419190233258.png)

**Iterator Traits用于区分是 class Iterators (也就是一般的迭代器)还是 non-class Iterators(即 native pointer)；两种情况对应不同的的处理!!**

在算法和迭代器之间加一层中间层Iterator traits来进行判断，好针对性的进行设计!!!

具体怎么做呢？

这时候算法里面就不能直接问迭代器的五个类型了，因为 native pointer 里面没有这五个参数,所以需要间接通过Traits去问!!!

下面的例子先回答了一个问题 value_type

```c++
template <typename I,...>
void algorithm(...){
    typename iterator_traits<I>::value_type v1;//通过traits去问
}

//如果是class Iterator在这里
template <class I>
struct iterator_traits{
    typedef typename I::value_type value_type;
}

//两个模板偏特化 pointer to T
template <class T>
struct iterator_traits<T*>{
    typedef T value_type;
}

//pointer to const T
template <class T>
struct iterator_traits<const T*>{
    typedef T value_type; //注意是T而不是const T
    //为什么是 T 而不是 const T？因为 value_type主要目的是去声明变量，const T没办法声明变量!!!
}
```

下面把指针的全回答完毕

```c++
//pointer to T
template <class T>
struct iterator_traits<T*>{
    typedef random_access_iterator_tag iterator_category;
    typedef T value_type;
    typedef T* pointer;
    typedef T& reference;
    typedef ptrdiff_t difference_type;
}
//pointer to const T
template <class T>
struct iterator_traits<const T*>{
    typedef random_access_iterator_tag iterator_category;
    typedef T value_type;
    typedef const T* pointer;
    typedef const T& reference;
    typedef ptrdiff_t difference_type;
}
```

## 第三讲：容器

### 3.深入探索vector

其实自己都可以封装一个vector，当然所有的功能是不现实的，但是基本的功能还是可以

### GCC2.9的设计

直观感受就是简洁明了

![image-20230419192921703](D:\Typora\Images\image-20230419192921703.png)

这里面有三根泛型指针，start，finish和end_of_storage

**下面就是比较重要的push_back()查数据，引出后面的二倍扩展空间!!!**

![image-20230419195537174](D:\Typora\Images\image-20230419195537174.png)

**else这里，调用insert_aux之后为什么还要进行一次判断呢？**

**这是因为由于insert_aux是一个辅助函数，那么在实际操作过程中可能会被其他类函数调用，比如insert，在这些函数的实现逻辑当中是需要进行判断的。**

![image-20230419195610102](D:\Typora\Images\image-20230419195610102.png)

**这段源码的大致意思就是：capacity不够了就在内存中找另一块2倍大小的内存用于存放新的vector，把原来的元素拷贝过去然后把原来的vector杀掉，各种值迁移到新内存这边去!!!**

**至于拷贝安插点后的原内容是因为这里可能会被insert函数给调用，这个部分是insert函数的逻辑!!!**

**Vector容器的迭代器**

既然vector是一个连续空间，那么iterator就不必要设置成为class了，只需要设置为pointer就可以

```c++
//所以如上所示，vector的迭代器是一根指针
typedef value_type* iterator; // T*
```

### GCC4.9的设计

复杂，依托答辩

![image-20230419201320469](D:\Typora\Images\image-20230419201320469.png)

## 4.20

### 1.深入探索 deque 和 queue , stack

### deque

deque 双端队列的实现结构：

![image-20230420164335846](D:\Typora\Images\image-20230420164335846.png)

**具体实现：将deque的存储划分为若干个等大的区域，每个区域的首元素用一个指针存放在一个vector容器中(就是图中的map数组)，当缓冲区buffer的左端或者右端不够的时候，就新开一个缓冲区放在左边和右边，存入元素，并且把该buffer的首指针存入map数组的左边或者右边。因此deque的迭代器的就分为：**

**first 该缓冲区的首地址；last 该缓冲区的末尾(首闭尾开)；cur 元素的位置；node 存入map数组的指针!!!**

下面看一下deque的具体源代码设计：

**GCC2.9:**

![image-20230420170120084](D:\Typora\Images\image-20230420170120084.png)

其中第三个参数 Bufsiz 是可以人为指定缓冲区的大小

![image-20230420170739434](D:\Typora\Images\image-20230420170739434.png)

从这里可以看出，上面调用了一个函数，如果 BufSiz 不为0，那么就设置大小为人为指定；如果为0，则表示设置为预设的值，需要查看要存放的类型的大小，大于512就指定一个缓冲区只放这一个元素，个数设置为1；小于的话就计算出个数，计算出个数之后就可以知道缓冲区的大小了

![image-20230420170150250](D:\Typora\Images\image-20230420170150250.png)

这个迭代器里面也包含了五个必要类型，写的非常严谨，也有了四个需要的参数!!!

**Insert函数:**

![image-20230420172205138](D:\Typora\Images\image-20230420172205138.png)

如果是在头部和尾部插入，那么和push_front()和push_back()没有区别

如果在中间插入就调用赋值函数 insert_aux()

![image-20230420172617688](D:\Typora\Images\image-20230420172617688.png)

由于在中间插入必然会导致元素的拷贝过程，为了减少开销，提高效率，我们需要判断元素在deque靠近start和finish哪一端的位置，这样可以更好的去选择操作的方向

**deque如何模拟连续空间操作？(操作符重载)**

![image-20230420173341877](D:\Typora\Images\image-20230420173341877.png)

**deque 的 - 号操作符重载**

由于 deque 存储的缓冲区buffer机制，我们必须判断两个迭代器之间有多少个缓冲区buffer，然后再根据计算公式来进行计算得出两个迭代器之间元素的个数

具体就是这样!

**根据两个迭代器的node指针找到map数组里面两个指针距离的位置就可以知道两个中间差了多少个缓冲区buffer了，再加上本缓冲区内的距离就是两个迭代器之间的元素个数!!!**

![image-20230420173832550](D:\Typora\Images\image-20230420173832550.png)

**++ 和 -- 操作符重载**

注意需要判断迭代器移动过程中是否超越了本缓冲区的界限移入另一个缓冲区!!!

一个比较好的编码习惯就是后++调用前++；后--调用前--

![image-20230420174307662](D:\Typora\Images\image-20230420174307662.png)

**+= + 号操作符重载**

在 += 运算符重载中，需要注意判断迭代器位置移动之后有没有超越边界，如果超越了边界，需要进行相应的边界修改

![image-20230420193008988](D:\Typora\Images\image-20230420193008988.png)

+= 如果没有正确的缓冲区需要切换到正确的缓冲区；如果是正确的缓冲区那就很简单了

**-= - 号运算符重载**

用的是+=和+号的重载

![image-20230420193316669](D:\Typora\Images\image-20230420193316669.png)

**GCC 4.9:** 依托答辩

![image-20230420193754702](D:\Typora\Images\image-20230420193754702.png)

## 4.21

### 1. queue

**内部存了一个 deque 容器，二者形成了复合 composition 关系**

queue内部的函数，deque能完全满足它，所以调用 deque 的成员函数就可以了!!!

![image-20230421103130501](D:\Typora\Images\image-20230421103130501.png)

**queue和stack，关于其iterator和底层容器**

**stack和queue不允许遍历，也不提供iterator!!!!**

因为他们的模式是先进后出和先进先出，这样的模式不允许能访问到任意位置的元素

![image-20230421104600566](D:\Typora\Images\image-20230421104600566.png)

**关于stack和queue的内部支撑容器，上面讲的是deque，其实也可以用list**

默认提供的是deque，这个效率比较快一点

![image-20230421104609426](D:\Typora\Images\image-20230421104609426.png)

**stack可以用vector做底部支撑；queue不可以用vector!!!**

因为vector没有 pop_front() 函数!!!

![image-20230421104723249](D:\Typora\Images\image-20230421104723249.png)

**关于其他的底部容器支撑，stack和queue都不可以选用set或者map做底部容器支撑，因为他们两个也没有相应的函数提供!!!**

![image-20230421105352734](D:\Typora\Images\image-20230421105352734.png)

关于这些底部容器支撑，如果你没有调用它不存在的函数，那其实调用还是可以的，但是总体来看是不行的!!!

### 2.自己手写了一个简单的二叉树(创建二叉树函数不会)

```c++
//TreeNode.h
#ifndef __TREENODE__
#define __TREENODE__

enum Left_Right
{
    Left,
    Right
};

// 定义结点类
#include <iostream>
template <class Type>
struct TreeNode
{
    typedef Type __ValueType;
    typedef TreeNode<__ValueType> __NodeType;
    typedef __NodeType *__pointer;

    __ValueType val;
    __pointer left;
    __pointer right;

    void __init__();
    void insert(__ValueType val, bool is_left = Left); // 1为左 0为右 默认为做左

    void PreOrder();
    void InOrder();
    void PostOrder();
    void visit();
};

// 虽然不给节点写构造函数但是写一个初始化没问题的
template <class Type>
inline void TreeNode<Type>::__init__()
{
    this->left = nullptr;
    this->right = nullptr;
}

// 插入子树
template <class Type>
inline void TreeNode<Type>::insert(__ValueType val, bool is_left)
{
    if (is_left == Left)
    {
        if (this->left)
        {
            cout << "leftnode has already be used." << endl;
            return;
        }
        __pointer newNode = new TreeNode<__ValueType>;
        newNode->__init__();
        newNode->val = val;
        this->left = newNode;
    }
    else
    {
        if (this->right)
        {
            cout << "rightnode has already be used." << endl;
            return;
        }
        __pointer newNode = new TreeNode<__ValueType>;
        newNode->__init__();
        newNode->val = val;
        this->right = newNode;
    }
}

// visit
template <class Type>
inline void TreeNode<Type>::visit()
{
    cout << this->val << endl;
}

// 遍历
template <class Type>
inline void TreeNode<Type>::PreOrder()
{
    if (!this)
        return;
    visit();
    left->PreOrder();
    right->PreOrder();
}

template <class Type>
inline void TreeNode<Type>::InOrder()
{
    if (!this)
        return;
    left->InOrder();
    visit();
    right->InOrder();
}

template <class Type>
inline void TreeNode<Type>::PostOrder()
{
    if (!this)
        return;
    left->PostOrder();
    right->PostOrder();
    visit();
}

#endif
```

```c++
//BinaryTree.h
#ifndef __BINARTTREE__
#define __BINARTTREE__
#include "TreeNode.h"

enum Order
{
    pre,
    in,
    post
};

// 写一个全局函数来删除二叉树
template <typename Type>
inline void deleteNodes(TreeNode<Type> *node)
{
    if (node->left)
        deleteNodes(node->left);
    if (node->right)
        deleteNodes(node->right);
    delete node;
}

// 定义整颗二叉树类
template <class Type>
class BinaryTree
{
    typedef Type __ValueType;
    typedef TreeNode<__ValueType> __NodeType;
    typedef __NodeType &__reference;
    typedef __NodeType *__pointer;

public:
    // BinaryTree();
    explicit BinaryTree(__ValueType val = NULL); // 不给默认值就是NULL
    ~BinaryTree() { deleteTree(); }
    void deleteTree();
    void printTree(Order ord);
    __reference getroot() const { return *root; }

private:
    __pointer root;
};

// 构造函数
template <class Type>
inline BinaryTree<Type>::BinaryTree(__ValueType val)
{
    root = new TreeNode<Type>;
    root->val = val;
    root->left = nullptr;
    root->right = nullptr;
}

// 整棵树的析构函数
template <class Type>
inline void BinaryTree<Type>::deleteTree()
{
    deleteNodes(root);
}

// 前序遍历
template <class Type>
inline void BinaryTree<Type>::printTree(Order ord)
{
    switch (ord)
    {
    case pre:
        root->PreOrder();
        break;
    case in:
        root->InOrder();
        break;
    case post:
        root->PostOrder();
        break;
    }
}

#endif
```

```c++
//main.cpp
#include <iostream>
using namespace std;
#include "BinaryTree.h"

/*
       1
   3         2
4    6    8
    7        0
*/
namespace test
{
    BinaryTree<int> sample()
    {
        BinaryTree<int> tree(1);
        tree.getroot().insert(3, Left);
        // left
        auto leftNode = tree.getroot().left;
        leftNode->insert(4, Left);
        leftNode->insert(6, Right);
        auto leftNode2 = leftNode->right;
        leftNode2->insert(7, Left);
        // right
        tree.getroot().insert(2, Right);
        auto rightNode = tree.getroot().right;
        rightNode->insert(8, Left);
        auto rightNode2 = rightNode->left;
        rightNode2->insert(0, Right);

        return tree;
    }
}

int main()
{
    auto tree = test::sample();

    tree.printTree(pre); // 1 3 4 6 7 2 8 0
    cout << endl;
    tree.printTree(in); // 4 3 7 6 1 8 0 2
    cout << endl;
    tree.printTree(post); // 4 7 6 3 0 8 2 1

    return 0;
}
```

### 3 rb_Tree 红黑树

红黑树是一种高度平衡的二叉搜寻树；由于它保持尽量的平衡，非常有利于search和insert的操作，并且在改变了元素的操作之后会继续保持树状态的平衡

### 红黑树 rb_Tree 与二叉平衡树 AVL 的对比：

![image-20230421175541816](D:\Typora\Images\image-20230421175541816.png)

**为什么要有红黑树？**

大多数二叉排序树 BST 的操作（查找、最大值、最小值、插入、删除等等）都是 O(h)O(h)O(h) 的时间复杂度，h 为树的高度。但是对于斜树而言（BST极端情况下出现），BST的这些操作的时间复杂度将达到  O(n) 。为了保证BST的所有操作的时间复杂度的上限为  O(logn)，就要想办法把一颗[BST树](https://www.zhihu.com/search?q=BST树&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"answer"%2C"sourceId"%3A1246106121})的高度一直维持在logn，而红黑树就做到了这一点，红黑树的高度始终都维持在logn，n 为树中的顶点数目。

rb_Tree和AVL相比，虽然AVL更加平衡，但是条件更加苛刻，**红黑树追求的是大致平衡。** AVL 树比红黑树更加平衡，但AVL树在插入和删除的时候也会存在大量的旋转操作。**所以涉及到频繁的插入和删除操作，切记放弃AVL树，选择性能更好的红黑树；当然，如果你的应用中涉及的插入和删除操作并不频繁，而是查找操作相对更频繁，那么就优先选择 AVL 树进行实现**

![](D:\Typora\Images\image-20230421174737185.png)

红黑树**提供遍历操作以及迭代器iterator**，但是这个迭代器是**只读迭代器**，因为**不能修改节点上元素的值**，如果修改了元素的值，那么会导致大小关系发生变化，整个红黑树的平衡性就发生变化了

图中第三段，按理来说红黑树的元素是不能通过迭代器修改元素值的，但是这个红黑树后面是用于set和map容器的，set的key和value相等不能修改；但是map的key和value没有必然联系，排序和查找都是基于key来进行的，value可以任意修改，所以可以通过迭代器修改value，这么做做是正确的

红黑树的设计当中存在两种设计 **insert_unique() 和 insert_equal()** ，表示key可以重复或者不重复，这样就可以引申出 set和 multiset，mal和 multimap

### 标准库对红黑树的实现:

![image-20230421182526116](D:\Typora\Images\image-20230421182526116.png)

**这里标准库里面的value不是指我们理解的value值，这里的value是key和data合起来叫做value，也就是一整个节点的类型；第三个参数是说我们怎么样从这个节点value当中把重要的key拿出来!!!第四个参数是说如何根据key来进行比较，因为后续要进行排序操作!!!**

在rb_tree类当中，它的设计和我的设计差不多，都是把树和节点分开的来设计，所以在树rb_tree当中只用了三个参数来向外表现

```c++
size_type node_count;//节点的数量
link_type header;//头节点(类型是指针)
Compare key_compare;//比较key的函数指针或者仿函数
```

**在红黑树的结构里有一个 header 节点，他的元素值为空，跟list的设计一样，前闭后开的区间!!!**

这样的设计会使后面的实现方便很多

![image-20230421182650170](D:\Typora\Images\image-20230421182650170.png)

虽然红黑树不推荐直接使用，因为更好的做法是使用上层容器；但是可以简单的使用一下来测试我们对其的理解

```c++
rb_tree<int,int,identity<int>,less<int>,alloc>;
//key和value类型相同，说明key和data是一个东西(否则返回的value不可 能是int类型而应该是个类)，这样的话从value中取出key就可以使用写好的identity函数对象，即你传什么给我我就给你返回什么
```

使用红黑树的测试程序:(新版本的名称有些变化)

```c++
#include <iostream>
using namespace std;
#include <bits/stl_tree.h>

int main()
{
    _Rb_tree<int, int, _Identity<int>, less<int>, allocator<int>> rbtree;
    cout << rbtree.size() << endl;  // 0
    cout << sizeof(rbtree) << endl; // 48 跟插不插元素没关系，因为里面存的是节点指针d
    cout << rbtree.empty() << endl; // 1

    rbtree._M_insert_unique(3);
    rbtree._M_insert_unique(8);
    rbtree._M_insert_unique(5);
    rbtree._M_insert_unique(9);
    rbtree._M_insert_unique(13);
    rbtree._M_insert_unique(3);      // unique 所以3插不进去
    cout << rbtree.size() << endl;   // 5
    cout << rbtree.empty() << endl;  // 0
    cout << rbtree.count(3) << endl; // 1

    rbtree._M_insert_equal(3); // equal 所以3能插进去
    rbtree._M_insert_equal(3);
    cout << rbtree.size() << endl;   // 7
    cout << rbtree.empty() << endl;  // 0
    cout << rbtree.count(3) << endl; // 3

    return 0;
}
```

### 4.基于红黑树的set和map

### set/multiset

由于set/multiset的key和value相同，所以没有办法通过迭代器修改元素的值，也就是修改key，error

set插入元素使用 insert_unique()；multiset可以重复，使用 insert_equal()

![image-20230421192116968](D:\Typora\Images\image-20230421192116968.png)

注意不能修改迭代器的值，const_iterator，以下是示例代码：

```c++
#include <iostream>
using namespace std;
#include <set>
#include <vector>

int main()
{
    vector<int> v{1, 3, 5, 4, 6, 8, 8, 9};
    for (int &val : v)
        ++val;
    for (int &val : v)
        cout << val << ' ';
    cout << endl;

    //会自动排序
    set<int, less<int>> s{3, 1, 5, 4, 6, 8, 8, 9};
    // for (int &val : s) // 这里就会报错,因为这个的迭代器是不可以更改值的
    //     ++val;
    for (int val : s)
        cout << val << ' ';
    cout << endl;

    return 0;
}
```

### map/multimap

**map没有办法通过迭代器修改key的值，但是可以用过迭代器修改value的值!!!!!**

map插入元素使用 insert_unique()；multimap 可以重复，使用 insert_equal()

![image-20230421194843419](D:\Typora\Images\image-20230421194843419.png)

**key_type和data_type被包装成为一个pair<const Key,T>；注意这里const修饰代表key无法修改，然后value_type是真正的存放类型，然后select1st代表拿取pair里面的第一个元素!!!!**

使用红黑树测试map：

```c++
#include <iostream>
using namespace std;
#include <bits/stl_tree.h>

// 源代码这么写的，我没看懂
template <class T>
struct SelectFirst
{
    template <class Pair>
    typename Pair::first_type &
    operator()(Pair &x) const
    {
        return x.first;
    }

    // typename T::first_type &
    // operator()(T &x) const
    // {
    //     return x.first;
    // }
};

int main()
{
    typedef int Key_Type;
    typedef pair<const int, char> Value_Type;

    // _Rb_tree<Key_Type, Value_Type, _Select1st<Value_Type>, less<int>> rbtree;
    _Rb_tree<Key_Type, Value_Type, SelectFirst<Value_Type>, less<int>> rbtree; // error
    // select1st怎么写不知道
    cout << rbtree.size() << endl;  // 0
    cout << sizeof(rbtree) << endl; // 48 跟插不插元素没关系，因为里面存的是节点指针d
    cout << rbtree.empty() << endl; // 1

    rbtree._M_insert_unique(make_pair(3, 'a'));
    rbtree._M_insert_unique(make_pair(8, 'b'));
    rbtree._M_insert_unique(make_pair(5, 'c'));
    rbtree._M_insert_unique(make_pair(9, 'd'));
    rbtree._M_insert_unique(make_pair(13, 'e'));
    rbtree._M_insert_unique(make_pair(3, 'f')); // unique 所以3插不进去
    cout << rbtree.size() << endl;              // 5
    cout << rbtree.empty() << endl;             // 0
    cout << rbtree.count(3) << endl;            // 1

    rbtree._M_insert_equal(make_pair(3, 'a')); // equal 所以3能插进去
    rbtree._M_insert_equal(make_pair(3, 'a'));
    cout << rbtree.size() << endl;   // 7
    cout << rbtree.empty() << endl;  // 0
    cout << rbtree.count(3) << endl; // 3

    return 0;
}
```

其他都没什么，其中第三个模板参数SelectFirst<>(我自己写的)不是很理解为什么这么写

```c++
template <class T>//不明白这里为什么要用两次模板并且第一次的模板参数没啥用
struct SelectFirst
{
    template <class Pair>
    typename Pair::first_type &
    operator()(Pair &x) const
    {
        return x.first;
    }
};
```

使用map的示例代码:

```c++
#include <iostream>
using namespace std;
#include <map>

int main()
{
    // 第一个参数是key(不可修改,所以进去后红黑树会自动转为const类型),第二个参数是data
    //元素会按照key自动排序!!!
    map<int, char, less<int>> m{make_pair(9, 'a'),
                                make_pair(5, 'b'),
                                make_pair(6, 'c'),
                                make_pair(4, 'd'),
                                make_pair(8, 'c'),
                                make_pair(9, 'b'),
                                make_pair(6, 'd'),
                                make_pair(1, 'a')};
    m[0] = 'f';
    for (auto &val : m)
    {
        //key不可修改，但是data可以修改
        cout << val.first << ' ' << m[val.first] << endl;//类似于py的字典
        val.second++;
        cout << val.first << ' ' << val.second << endl;
    }

    return 0;
}
```

## 4.22

### 1.map独特的 operator [ ]!!!

**作用：根据key传回data。注意只有map有，因为key不为data并且key是独一无二的!!!**

**如果key不存在的话，就会创建这个key并且data使用默认值!!!**(和py的字典差不多)

![image-20230422140640649](D:\Typora\Images\image-20230422140640649.png)

使用二分查找在有序的key当中查找目标key，如果找不到的话就进行insert操作创建一个新的key！！！

### 2.hashtable 散列表

哈希表的设计

![image-20230422143136762](D:\Typora\Images\image-20230422143136762.png)

**在有限的空间之下根据哈希函数将元素(分为key和data)的key映射成为hashcode放到对应的位置下面，key下面用一个链表将key和data串起来!!!!**

**由于bucket数组存的是链表指针，这个链表如果串的元素太多了之后那么搜索效率会大大降低，这个状态就是非安全状态。程序员的经验告诉我们当所有的链表下面串的元素个数大于buckets数组的大小的时候就比较危险了。**

**这个时候需要打散hashtable，增大buckets数组的size，一般是两倍左右，并且数组的size最好是质数，并且将元素按照新的hash规则重新插入链表中!!!!**

总结就是：不能让hashtable下面串的链表太长，太长了需要增加buckets的size来打散哈希表重新回到安全状态。

GCC下面的buckets数组的size是这么确定的：**大致都是2倍附近的质数**

![image-20230422144338234](D:\Typora\Images\image-20230422144338234.png)

来看看hashtable的实现：

![image-20230422145128498](D:\Typora\Images\image-20230422145128498.png)

Value代表key和data集合，key就是键值，HashFcn代表哈希函数，就是如何把key映射为编号hashcode，ExtractKey代表如何从value里面取出key；EqualKey代表如何判断两个key相同!!!

至于是单向链表还是双向链表，这个就看不同的版本了。

**参数模板里面最难的一点就是决定hashtable的hash函数，怎么样将hash的key值映射为hashcode!!!**

参考系统提供的hash模板函数

**注意：hash函数(一般是个仿函数)返回的值应该是一个编号，也就是 size_t**

![image-20230422154019414](D:\Typora\Images\image-20230422154019414.png)

定义了hash函数，然后什么也不做，后面进行一些特化的处理!!!

![image-20230422154054412](D:\Typora\Images\image-20230422154054412.png)

注意这里的hash函数设计，我们在将key转化为hashcode的过程中，可以任意设计hash函数使得转化成为的hashcode**尽量不重复，尽量够乱!!!**

**在算出hashcode之后还要放入篮子，这个时候就很简单了，就把hashcode求篮子的size的余数就可以知道放在哪里了!!!!**(现在基本都是这么做的)

![image-20230422155807393](D:\Typora\Images\image-20230422155807393.png)

使用hashtable的例子：

```c++
#include <iostream>
using namespace std;
#include <hashtable.h>
#include <cstring>

struct eqstr
{
    bool operator()(const char *str1, const char *str2) const
    {
        return strcmp(str1, str2) == 0;
    }
};

// 如果是自己设计就可以这么设计
inline size_t _hash_string(const char *s)
{
    size_t ret = 0;
    for (; *s != '\n'; ++s)
        ret = 10 * ret + *s;
    return ret;
}
struct fuck
{
    size_t operator()(const char *s) const { return _hash_string(s); }
};


int main()
{
    __gnu_cxx::hashtable<const char *, const char *,
                         hash<const char *>, // 标准库没有提供 hash<std::string>!!!!
                         _Identity<const char *>,
                         eqstr> 
                 // 不能直接放入strcmp，因为我们需要判断是否相同，返回的是true和false;而strcmp返回的是1 0 -1，接口不太一致
        ht(50, hash<const char *>(), eqstr());// 这个东西没有默认空的构造函数，需要提供一些东西
    // 从这里可以看出直接使用hashtable非常难用

    ht.insert_unique("kiwi");
    ht.insert_unique("plum");
    ht.insert_unique("apple");
    for_each(ht.begin(), ht.end(), [&](auto data)
             { cout << data << endl; });

    // cout << hash<int>()(32) << endl;
    return 0;
}
```

##  第四讲：算法

### 3.算法概述

![image-20230422162235208](D:\Typora\Images\image-20230422162235208.png)

算法没有办法直接面对容器，他需要借助中间商迭代器才可以，换句话说，算法不关系容器是怎么样的，只关心容器提供给我的迭代器是怎么样的，而迭代器的设计的符号重载是普适的，这样就可以适用于大多数容器了。

### 4.迭代器的五种分类：注意这五种都是类!!!

![image-20230422162614061](D:\Typora\Images\image-20230422162614061.png)

random_access_iterator_tag 随机访问迭代器：可以跳着访问，任意一个都可以访问(重载了+ - += -= ++ -- 运算符)

bidirectional_iterator_tag 双向访问迭代器：可以往前走或者往后走，但是一次只能走一格(重载了 ++ -- 运算符)

farward_iterator_tag 单向访问迭代器：只能向一个方向走，inin一次只能走一格

打印一下各种容器的iterator_category

![image-20230422163446874](D:\Typora\Images\image-20230422163446874.png)

```c++
#include <iostream>
using namespace std;
#include <array>
#include <vector>
#include <map>
#include <list>
#include <forward_list>
#include <deque>
#include <set>
#include <unordered_set>
#include <unordered_map>
#include <bits/stream_iterator.h>

// 可以只指定值不给参数
void __display_category(random_access_iterator_tag)
{
    cout << "random_access_iterator" << endl;
}
void __display_category(bidirectional_iterator_tag)
{
    cout << "bidirectional_iterator" << endl;
}
void __display_category(forward_iterator_tag)
{
    cout << "forward_iterator" << endl;
}
void __display_category(output_iterator_tag)
{
    cout << "output_iterator" << endl;
}
void __display_category(input_iterator_tag)
{
    cout << "input_iterator" << endl;
}

template <typename I>
void display_category(I iter)
{
    // 加上typename是为了是 I 就是迭代器类型(目前这么理解)
    typename iterator_traits<I>::iterator_category cagy; // 去问萃取剂这个迭代器是什么类型
    __display_category(cagy);
}

int main()
{
    display_category(array<int, 10>::iterator());
    display_category(vector<int>::iterator());
    display_category(list<int>::iterator());
    display_category(forward_list<int>::iterator());
    display_category(deque<int>::iterator());

    display_category(set<int>::iterator());
    display_category(map<int, int>::iterator());
    display_category(multiset<int>::iterator());
    display_category(multimap<int, int>::iterator());
    display_category(unordered_set<int>::iterator());
    display_category(unordered_map<int, int>::iterator());
    display_category(unordered_multiset<int>::iterator());
    display_category(unordered_multimap<int, int>::iterator());

    // 这两个不太一样，是从适配器adapter产生的
    display_category(istream_iterator<int>());
    display_category(ostream_iterator<int>(cout, ""));

    return 0;
}
```

### 5.iterator_category对算法的效率影响

不同的迭代器类型会导致在访问的过程中效率有区别

![image-20230422171334117](D:\Typora\Images\image-20230422171334117.png)

**注意对右边代码的解读，这个distance函数是找两个迭代器之间的距离(ptrdiff_t 类型)，然后就问萃取机迭代器的类型是什么？然后针对函数调不同的重载函数就可以了!!!**

```c++
#include <iostream>
using namespace std;
#include <vector>
#include <list>

template <typename Iterator, typename Distance>
inline Iterator &_advance(Iterator &iter, Distance n, std::random_access_iterator_tag)
    {
        iter += n;
        return iter;
    }

template <typename Iterator, typename Distance>
inline Iterator &_advance(Iterator &iter, Distance n, std::bidirectional_iterator_tag)
    {
        if (n >= 0)
            while (n--)
                ++iter;
        else
            while (n++)
                --iter;
        return iter;
    }

template <typename Iterator, typename Distance>
inline Iterator &_advance(Iterator &iter, Distance n, std::input_iterator_tag)
    {
        while (n--)
            ++iter;
        return iter;
    }

template <typename Iterator, typename Distance>
inline Iterator Advance(Iterator iter, Distance n)
    // 这里最好不传入引用类型，因为第一下面没有更改iter的值，不用担心实参形参的问题；
    // 第二，外部可能传入的是begin()和end()这类没有办法直接修改的迭代器
    // 我们在使用的时候都是声明了一个运动迭代器，他的初值是begin(),这样来操作的
    // 所以传入引用会出问题，最好传值，但是后面就可以传入引用了，因为我们是创建了一个新的迭代器对象
    {
        typedef typename std::iterator_traits<Iterator>::iterator_category Iterator_Category;
        return _advance(iter, n, Iterator_Category());
    }

int main()
{
    vector<int> v{3, 5, 6, 7};
    cout << *myadvance().Advance(v.begin() + 2, -1) << endl; // 5
    list<int> l{3, 5, 6, 7, 12};
    cout << *myadvance().Advance(l.begin(), 4) << endl; // 12

    return 0;
}
```

注意注释的内容，为什么这里该传引用，这里不该传引用!!!

**从这里我们可以看出，迭代器类型的不同会导致算法效率的不同，但是我们不是通过模板特化来实现的，是通过函数重载来实现的!!!**

**算法源码对 iterator_category 的暗示:**

![image-20230422183037284](D:\Typora\Images\image-20230422183037284.png)

因为上面的迭代器都是模板，但是有些算法在实现的过程中只对某种类型的迭代器有效，所以设计者会暗示迭代器的类型来方便阅读和修改!!!

## 4.23

### 1.算法源代码剖析

C++ STL 库里面的标准算法格式

```c++
template <typename Iterator>
std::Algorithm(Iterator iter1,Iterator iter2){
    ...
}

//带比较的参数 一般是仿函数
template <typename Iterator,typename Cmp>
std::Algorithm(Iterator iter1,Iterator iter2,Cmp cmp){
    ...
}
```

### accumulate

遍历整个容器对每个元素进行操作(可以是累加)然后返回值

![image-20230423113446373](D:\Typora\Images\image-20230423113446373.png)

测试accumulate：

```c++
int myfunc(int x, int y)
{
    return x + 2 * y;
}

struct myclass
{
    int operator()(int x, int y) const { return x + 3 * y; }
} myobj;

void test_accumulate()
{
    cout << "test_accumulate().........." << endl;
    int init = 100;
    int nums[] = {10, 20, 30};

    cout << "using default accumulate: ";
    cout << accumulate(nums, nums + 3, init); // 160
    cout << '\n';

    cout << "using functional's minus: ";
    // minus 减法 仿函数
    cout << accumulate(nums, nums + 3, init, minus<int>()); // 40
    cout << '\n';

    cout << "using custom function: ";
    cout << accumulate(nums, nums + 3, init, myfunc); // 220
    cout << '\n';

    cout << "using custom object: ";
    cout << accumulate(nums, nums + 3, init, myobj); // 280
    cout << '\n';
}
```

自己实现以下accumulate:

```c++
#include <iostream>
using namespace std;
#include <vector>
#include <list>
#include <bits/stl_numeric.h> //accumulate
#include <string>

struct Sum_Square
{
    template <class Value_Int>
    inline Value_Int &operator()(Value_Int &val, Value_Int val_iter)
    {
        val += val_iter * val_iter;
        return val;
    }
};

struct String_Append
{
    template <class Value_String>
    inline Value_String &operator()(Value_String &val, Value_String val_iter)
    {
        val_iter.append(" ");
        val.append(val_iter);
        return val;
    }
};

struct Algorithm
{
    template <class Iterator, class Value_Type>
    inline static Value_Type Accumulate(Iterator begin, Iterator end, Value_Type val)
    {
        for (; begin != end; ++begin)
            val += *begin;
        return val;
    }

    template <class Iterator, class Value_Type, class Binary_Operation>
    inline static Value_Type Accumulate(Iterator begin, Iterator end, Value_Type val, Binary_Operation binary_op)
    {
        for (; begin != end; ++begin)
            val = binary_op(val, *begin);
        return val;
    }
};

int main()
{
    vector<int> v{5, 3, 6, 9, 10};
    cout << Algorithm::Accumulate(v.begin(), v.end(), 0 << endl; // 33
    cout << Algorithm::Accumulate(v.begin(), v.end(), 0, Sum_Square()) << endl; // 251
    list<string> l{"hello", "I", "want", "to", "fuck", "you", "my", "friend."};
    cout << Algorithm::Accumulate(l.begin(), l.end(), string(), String_Append()) << endl;

    return 0;
}
```

### for_each

容器的遍历算法

![image-20230423171745364](D:\Typora\Images\image-20230423171745364.png)

自己实现一下for_each

```c++
#include <iostream>
using namespace std;
#include <vector>
#include <algorithm>

struct Algorithm
{
    template <class Iterator, class Function>
    //为什么要返回 Function 仿函数呢?(或者函数指针)
    inline static Function For_each(Iterator first, Iterator last, Function f)
    {
        for (; first != last; ++first)
            f(*first); // 注意是直接把数据传递给函数 f
        return f;
    }
};

int main()
{
    vector<int> v1{2, 5, 3, 6, 9};
    vector<int> v2;

    // 不修改v1的值
    Algorithm::For_each(v1.begin(), v1.end(), [&](int val)
                        { val*=2;v2.push_back(val); });

    for_each(v1.begin(), v1.end(), [&](auto val)
             { cout << val << ' '; });
    cout << endl;

    for (auto val : v2)
        cout << val << ' ';
    cout << endl;

    return 0;
}
```

看到for_each的返回值，我不得不思考为什么要返回Function仿函数呢？(很少情况下函数指针)

**原因是：可以监视仿函数对象在经过这个for_each操作之后的状态**

```c++
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

/*
    for_each()它可以返回其仿函数(返回所传入的函数对象的最终状态).
    这样我们就可以通过for_each()的返回值来获取仿函数的状态.
*/

/* 仿函数 */
class CSum
{
public:
    CSum() { m_sum = 0; }

    void operator()(int n) { m_sum += n; }

    int GetSum() const { return m_sum; }

private:
    int m_sum;
} cs;

int main()
{
    vector<int> vi;
    for (int i = 1; i <= 100; i++)
        vi.push_back(i);
    // 通过for_each返回值访问其最终状态(返回所传入的函数对象的最终状态).
    cs = for_each(vi.begin(), vi.end(), cs); // 返回的是一个新创建的对象，未返回引用，不会修改实参
    cout << cs.GetSum() << endl;

    return 0;
}
```

### replace,replace_if,replace_copy,replace_copy_if

```c++
#include <iostream>
using namespace std;
#include <vector>
#include <algorithm>

template <typename Container>
void print(Container con)
{
    for (auto val : con)
        cout << val << ' ';
    cout << endl;
}

struct Algorithm
{
    template <class Iterator, class Value_Type>
    inline static void Replace(Iterator first, Iterator last, const Value_Type oldval, const Value_Type newval)
    {
        for (; first != last; ++first)
            if (*first == oldval)
                *first = newval;
    }

    template <class Iterator, class Value_Type, class Predicate>
    // 给一个谓词来判断条件是否更改
    inline static void Replace_if(Iterator first, Iterator last, Predicate pred, const Value_Type newval)
    {
        for (; first != last; ++first)
            if (pred(*first))//这里谓词传递的参数只有一个值
                *first = newval;
    }

    // 上面的算法当中传入的参数只有一个值，没传入如果需要比较的基准值
    template <class Value_Type>
    bool operator()(const Value_Type &val)
    {
        return val > 5;//我们肯定不想在内部手动更改这个5，而是想在外面写代码的时候把5写进去
    }
    // 为了解决这个问题需要引用仿函数适配器 functor adapter
    // 标准库提供的 bind2nd() 用法 bind2nd(greater<int>,val)
};

int main()
{
    vector<int> v{1, 1, 1, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
    print(v);

    Algorithm::Replace(v.begin(), v.end(), 1, 66);
    print(v);

    vector<int> v2{1, 1, 1, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
    Algorithm::Replace_if(v2.begin(), v2.end(), bind2nd(greater<int>(), 5), 666);
    print(v2);

    vector<int> v3;
    v3.resize(v.size()); // 注意这里要给v3预分配空间，不然会段错误
    replace_copy(v.begin(), v.end(), v3.begin(), 10, 50);
    print(v3);

    return 0;
}
```

### count,count_if

这个差不多就不写了

![image-20230423175336384](D:\Typora\Images\image-20230423175336384.png)

为什么要返回difference_type呢？

算法通过萃取机询问迭代器，迭代器之间的间距类型怎么表示，这个类型就是difference_type

标准库的定义是 ptrdiff_t，也就是 long long，这下就可以理解了

有些容器自带的成员函数，比如图中的，这些函数的执行效率肯定比全局的执行效率更高!!!

### find,find_if

循序式查找，效率并不是很高，找不到返回last迭代器

![image-20230423202430305](D:\Typora\Images\image-20230423202430305.png)

### sort

```c++
#include <iostream>
using namespace std;
#include <vector>
#include <array>
#include <algorithm>

template <typename Container>
void print(Container con)
{
    for (auto val : con)
        cout << val << ' ';
    cout << endl;
}

bool myfunc(int i, int j)
{
    return i < j;
}

struct myclass
{
    bool operator()(int i, int j) { return i < j; }
} myobj;

int main()
{
    array<int, 8> arr = {32, 71, 12, 45, 26, 80, 53, 33};
    vector<int> v(arr.begin(), arr.end());
    print(v);

    // using default comparison (operator <)
    sort(v.begin(), v.begin() + 4); // 排序前四个 12 32 45 71 26 80 53 33
    print(v);

    // using function as comp
    sort(v.begin() + 4, v.end(), myfunc); // 12 32 45 71 26 33 53 80
    print(v);

    // using object as comp
    sort(v.begin(), v.end(), myobj); // 12 26 32 33 45 53 71 80
    print(v);

    // reverse iterators
    sort(v.rbegin(), v.rend(), less<int>()); // 80 71 53 45 33 32 26 12
    print(v);

    return 0;
}
```

![image-20230423203719337](D:\Typora\Images\image-20230423203719337.png)

**注意一点的是stl标准库里面的 sort 函数要求的是 random_access_iterator_tag!!!!!**

**所以list和forward_list没办法调用，只能调用他们自己的类函数sort!!!**

### binary_search(通过二分查找确定元素在不在容器当中)

**二分查找一定只能适用于一个有序序列!!!!并且在库函数当中只能用于升序序列!!!!**

![image-20230423210411376](D:\Typora\Images\image-20230423210411376.png)

使用例子：

```c++
#include <iostream>
using namespace std;
#include <algorithm>
#include <vector>

template <typename Container>
void Sort(Container &con) // 传引用，不然不改变实参
{
    sort(con.begin(), con.end());
}

template <typename Container>
void rSort(Container &con) // 传引用，不然不改变实参
{
    sort(con.rbegin(), con.rend());
}

template <typename Container>
void print(Container &con) // 传引用，不然不改变实参
{
    for (auto val : con)
        cout << val << ' ';
    cout << endl;
}

namespace Fuck
{
    template <typename Random_Iterator, typename Value_Type>
    bool __Binary_Search(Random_Iterator first, Random_Iterator last, const Value_Type &val,
                         random_access_iterator_tag)
    {
        // 先做一个检查 val比 *first大 那么找不到
        if (val < *first)
            return false;

        while (first != last)
        {
            Random_Iterator mid = first + (last - first) / 2; // 没有两个迭代器相加的重载版本!!!!
            if (*mid > val)
                last = mid; // 注意last要满足前闭后开
            else if (*mid < val)
                first = ++mid;
            else
                return true;
        }
        return false;
    }

    template <typename Random_Iterator, typename Value_Type>
    bool __Binary_Search(Random_Iterator first, Random_Iterator last, const Value_Type &val,
                         random_access_iterator_tag, int) // 多一个int代表降序
    {
        // 先做一个检查 val比 *first大 那么找不到
        if (val > *first)
            return false;

        while (first != last)
        {
            Random_Iterator mid = first + (last - first) / 2; // 没有两个迭代器相加的重载版本!!!!
            if (*mid > val)
                first = ++mid;
            else if (*mid < val)
                last = mid; // 注意last要满足前闭后开
            else
                return true;
        }
        return false;
    }

    template <typename Iterator, typename Value_Type>
    // 写了一个random_access_iterator的重载
    bool Binary_Search(Iterator first, Iterator last, const Value_Type &val)
    {
        // 想办法让其可以适用于降序序列
        typedef typename iterator_traits<Iterator>::iterator_category Iterator_Category;
        if (*first < *(last - 1)) // 升序 保持前闭后开的规则!!!
            return __Binary_Search(first, last, val, Iterator_Category());
        else
            return __Binary_Search(first, last, val, Iterator_Category(), true);
    }
}

int main()
{
    vector<int> v{1, 3, 6, 8, 7, 9, 2, 0};

    Sort(v); // 0 1 2 3 6 7 8 9
    print(v);
    cout << Fuck::Binary_Search(v.begin(), v.end(), 2) << endl;
    // cout << binary_search(v.begin(), v.end(), 5) << endl;
    rSort(v);
    print(v);
    cout << Fuck::Binary_Search(v.begin(), v.end(), 2) << endl;

    return 0;
}
```

## 4.24

## 第五讲：仿函数 适配器

### 1.仿函数 functors(注意要继承)

标准库提供的三大类型的仿函数：算术类 逻辑运算类 相对运算类

![image-20230424161751257](D:\Typora\Images\image-20230424161751257.png)

还有之前提到过的几个仿函数：

![image-20230424162149742](D:\Typora\Images\image-20230424162149742.png)

标准库的示范：

![image-20230424162843434](D:\Typora\Images\image-20230424162843434.png)

**注意到一点：标准库提供的functors都存在继承关系!!!!只有这样才算是真正融入了STL体系，这样才能更好的运作。**

**仿函数的 adaptable可适配 条件**

![image-20230424163242826](D:\Typora\Images\image-20230424163242826.png)

**STL规定，每个Functor都应该挑选适当的类来继承，因为适配器adapter会提出一些问题!!!!**

**什么叫adaptable?如果你希望自己的仿函数是可以被适配器调整的话，那么就继承一个适当的类，这样可以完成更多的操作!!!为什么要继承呢？因为可能在被adapter改造的时候可能会问这些问题。这也和算法问迭代器的五个问题一样，那里是通过迭代器的萃取机 Iterator Traits (也叫迭代器适配器 Iterator Adapters )去问的，这里同理通过继承的关系去回答adapter的问题!!!**

### 2.适配器 Adapter

存在多种 Adapters ，还是那张图，注意关系

<img src="D:\Typora\Images\image-20230424170726596.png" alt="image-20230424170726596" style="zoom:67%;" />

Adapter的关键是：

**这个Adapter要去改造某个东西(比如图中的container，functor，iterator)，这里就有两种解决方式，第一种是继承的方式，就是Adapter继承这个东西，拥有这个东西的属性来进行改造；第二种是内含的方式，Adapter内部有这个东西来进行改造!!!!**

**在标准库里面的实现绝大多数都是内含的方式!!!**

一下就是一些适配器的例子：

### 容器适配器：stack,queue

![image-20230424172134280](D:\Typora\Images\image-20230424172134280.png)

这个之前用过很多次了，就是把默认的容器拿进来进行改造，比如这里给的默认值是 deque ，改造之后能够以一种全新的面貌展现给用户，能够更加准确的针对用户的需要来进行相应的操作。

## 4.25

### 函数适配器：binder2nd

![image-20230425185823723](D:\Typora\Images\image-20230425185823723.png)

```c++
#include <iostream>
using namespace std;
#include <vector>
#include <algorithm>

namespace fuck
{
    // 自己写一个bind2nd和binder2nd
    // 仔细敲打一下这段代码
    // 这里暗示了需要传入的是一个二元运算符 然后下面的类型名称是继承里面写好的类型名称
    template <class Binary_Op>
    class _BinderSecond
    // 不继承这一行也可以运作，但是没办法进行后续的改造
    // 这里就不继承了!!!
    // : public unary_function<typename Binary_Op::first_argument_type, typename Binary_Op::second_argument_type>
    {
    protected:
        Binary_Op op;
        typename Binary_Op::second_argument_type value; // 第二参数 需要设定的固定值
    public:
        // ctor
        _BinderSecond(const Binary_Op &x, const typename Binary_Op::second_argument_type &y)
            : op(x), value(y) {}

        typename Binary_Op::result_type
        operator()(const typename Binary_Op::first_argument_type &x)
        {
            return op(x, value);
        }
    };

    template <class Binary_Op, class Value_Type>
    inline _BinderSecond<Binary_Op> _BindSecond(const Binary_Op &op, const Value_Type &val)
    {
        typedef typename Binary_Op::second_argument_type second_type;//这句话就是adapter在问问题
        return _BinderSecond(op, second_type(val));
    };
}

int main()
{

    vector<int> v{1, 3, 2, 5, 9, 8, 7, 6, 4, 10};
    cout << count_if(v.begin(), v.end(),
                     fuck::_BindSecond(less<int>(), 5)) // 绑定第二参数
         << endl;

    return 0;
}
```

注意其中的一些代码：

```c++
typename Binary_Op::second_argument_type value;
```

**为什么要加上 typename ，是为了通过编译，因为这个时候我们不知道Binary_Op是什么类型，然后如果他是我们想要的，也就是其中含有这个类型定义，那么就能通过编译，否则在这里就会报错!!!!**

**仿函数functors的可适配(adaptable)条件**

继承(因为adapter会问问题，提问类型)，是一个functor

![image-20230425191652925](D:\Typora\Images\image-20230425191652925.png)

### 函数适配器：not1

![image-20230425194548756](D:\Typora\Images\image-20230425194548756.png)

```c++
#include <iostream>
using namespace std;
#include <vector>
#include <algorithm>

namespace fuck
{
    // 对谓词做否定
    template <class Predicate>
    class _unary_negate
        // 继承为了后续的改造
        : public unary_function<typename Predicate::argument_type, bool>
    {
    protected:
        Predicate pred;

    public:
        // ctor
        _unary_negate(const Predicate &x) : pred(x) {}
        
        bool operator()(const typename Predicate::argument_type &x) const
        {
            return !pred(x);
        }
    };

    template <class Predicate>
    inline _unary_negate<Predicate> _Not1(const Predicate &pred)
    {
        return _unary_negate<Predicate>(pred);
    }
}

int main()
{
    vector<int> v{1, 3, 2, 5, 9, 8, 7, 6, 4, 10};
    cout << count_if(v.begin(), v.end(),
                     fuck::_Not1(bind2nd(less<int>(), 5))) // 绑定第二参数
         << endl;

    return 0;
}
```

**观察发现这些adapter的实现方法基本都是一个模板辅助函数，调用一个模板类，这个类里面有构造函数和小括号重载!!!!**

### 新型适配器：bind(since c++11)

右边是老版本，左边是新版本!!!

![image-20230425194051698](D:\Typora\Images\image-20230425194051698.png)

可见bind的实现是非常复杂的!!!!

下面是对bind的一些测试：

![image-20230425200558918](D:\Typora\Images\image-20230425200558918.png)

**bind 可以绑定：**

**functions函数；function objects 函数对象(仿函数)；**

**member functions 成员函数；data members 成员属性**

前两个比较好理解，其中第三个和第四个的绑定规则是：

**注意第一个参数传入的是传的是地址!!!!**

**member functions, _1;**

**data members,_1**

**必须有第二个参数，第二个参数必须是必须是某个object的地址，可以是一个占位符，在调用的时候被外界指定!!!**

**第一个参数可以理解为调用类里面的什么接口，第二个参数可以理解为谁来调用!!!!**

使用例子：

```c++
#include <iostream>
using namespace std;
#include <functional>
using namespace std::placeholders; // 使用占位符 _1 _2 _3这些
#include <vector>
#include <algorithm>

double my_divide(double x, double y)
{
    return x / y;
}

struct MyPair
{
    double a, b;
    double multiply() { return a * b; }
};

void Bind_Functions()
{
    // binding functions
    auto fn_five = bind(my_divide, 10, 2); // return 10.0/2.0
    cout << fn_five() << endl;             // 5

    auto fn_half = bind(my_divide, _1, 2); // return x/2.0
    cout << fn_half(10) << endl;           // 5

    auto fn_rounding = bind(my_divide, _2, _1); // 第一参数为除数，第二参数为被除数 return y/x
    cout << fn_rounding(10, 2) << endl;         // 0.2

    auto fn_invert = bind<int>(my_divide, _1, _2); // int 代表希望返回的类型 return int(x/y)
    cout << fn_invert(10, 3) << endl;              // 3
}

void Bind_Members()
{
    MyPair ten_two{10, 2};

    auto bound_memfn = bind(&MyPair::multiply, _1); // return x.multiply()
    cout << bound_memfn(ten_two) << endl;           // 20

    auto bound_memdata = bind(&MyPair::a, ten_two); // return tentwo.a
    cout << bound_memdata() << endl;                // 10

    auto bound_memdata2 = bind(&MyPair::b, _1); // return x.b
    cout << bound_memdata2(ten_two) << endl;    // 2

    vector<int> v{1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
    auto _fn = bind(less<int>(), _1, 5);
    cout << count_if(v.cbegin(), v.cend(), _fn) << endl;
    cout << count_if(v.begin(), v.end(), bind(less<int>(), _1, 5)) << endl;
}

int main()
{
    Bind_Functions();
    Bind_Members();

    return 0;
}
```

### 迭代器适配器：rbegin，rend

![image-20230425203641882](D:\Typora\Images\image-20230425203641882.png)

这个迭代器就是在正向迭代器的基础之上进行改造的迭代器!!!

### 迭代器适配器：inserter(没弄懂)

![image-20230425205345712](D:\Typora\Images\image-20230425205345712.png)

注意copy是已经写死的函数，那么如何才能改变他的行为呢？

**答案是借助操作符重载，本例子就是重载了 = 号运算符就是实现了由赋值操作变为插入操作了!!!!**

## 第六讲：STL周围的细碎知识点

### 1.一个万用的 hash function

