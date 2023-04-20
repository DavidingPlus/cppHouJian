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

