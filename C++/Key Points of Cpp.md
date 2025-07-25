## Key Points of Cpp

### 1.为什么会出现面向对象

- C 语言：面向过程，定义的数据所有函数都可以访问
- 面向对象：可以将 **数据** 及 **使用这些数据的函数** 放在一起（类） - 每个函数只可以访问它需要访问的数据，不会混杂在一起

![image-20250515112557763](https://raw.githubusercontent.com/mumushu1/Pictures/main/11122e55a971b4d6748066a57f6409f0.png)	

### 2.头文件防卫式声明

- 作用：若一个头文件被包含多次，防止 **重复定义**
- 比如 **`include "a.h"`** 和 **`include "b.h"`** ，如果 b.h 里面又引用 a.h，在编译时 a.h 就会被编译两次

```cpp
#ifndef CPP_STL_COMPLEX_H
#define CPP_STL_COMPLEX_H

class complex {

};

#endif //CPP_STL_COMPLEX_H
```

- 另一种方式： `#param once`，告诉编译器“这个文件只包含一次”

```cpp
#param once
class Person{};
```

### 3.内联函数 - inline

- 函数的调用过程
- 存在即合理：一些函数体较小，比如只进行 **简单** 的加法并返回，执行这种函数时，在 **参数传递、保存上下文** 等函数调用的操作上花费的时间甚至大于执行函数本身的时间
- 内联做了什么：编译器会在调用处 **直接展开函数代码**，而不是走一次“调用+返回”的流程。
- 🎯 **在类内定义的函数，默认就是 `inline` 的，** 类外定义的函数，如果想内联，需要加关键字 inline

```cpp
inline int add(int a,int b) {
    return a + b;}
```

- inline 只是给编译器建议，最后是否为内联编译器自有定夺
- ① 不适合内联：函数复杂、有递归，虚函数 ② 适合内联：短小，简单

### 4.访问级别

- public：可以被类外的函数访问，如果想让类中的函数被外界使用，就将这些函数放在 public 中
- private
    - 只有类内的函数才可访问，**一般存放类中定义的数据**，这也是面向对象本来的目的，将数据及访问数据的函数放在一个类中
    - 当然如果想定义一个只能在类中才能使用的函数，也可以将这个函数放在 private 中
- protect：暂未涉及
- 将成员变量设为 `private` 是为了封装和安全，将访问接口 `real()` 和 `imag()` 设为 `public` 是为了受控地提供读取功能。二者不矛盾，而是配合良好。

```cpp
class complex {
public:
    complex(double r = 0,double i = 0) : re(r),im(i){}
    
    double real() const {
        return re;
    }
    double imag() const {
        return im;
    }

private:
    double re,im;//实部和虚部
};

int main() {
    complex obj(1,2);
    obj.re = 2;
    //使用类内函数访问类中私有变量
    double re = obj.real();
    double im = obj.imag();
    cout << "re = " << re << " im = " << im << endl;

    //使用类外函数访问类中私有变量（错误）
    cout << obj.re;

    return 0;
}
```

### 5.构造函数 & 重载，单例设计模式

- 构造函数的 **初始列 `complex(double r,double i) : re(r),im(i) {}`**
- 一个类可以有多个构造函数，因为会存在 **多种不同的定义实例的方式** ，虽然每个构造函数的名称都相同，但 **编译器对于这些函数的标记是不同的**，编译器可以根据名称、返回值、参数等把一个函数赋予一个唯一的标识
- 将 **构造函数放在 private 中，并在 public 提供静态实例，可形成单例设计模式**  - 一个类不允许外界创建实例，只对外提供唯一实例

```cpp
class A {
public:
    static A& getInstance();

private:
    A();
    A(const A& rhs);
};

A& A :: getInstance() {
    static A a;
    return a;
}
```

### 6.常量函数

- 即在函数参数列表之后，函数体之前，加入 const 关键字
- **只适用于类的成员函数，加上const表示此函数不会通过this指针改变类成员变量的值，而对于全局函数，因为没有this指针和成员变量，所以自然也没有加从const这个说法**

```cpp
  double real() const {return re;}
  double imag() const {return im;}
  
  const complex obj(1,2);
```

- 作用： **若函数会改变变量的值，则不加 const，否则加 const**
- 如果这个函数没有改变变量的值，以上面的 real()和 imag()为例，如果它们没有加 const，在创建对象时，就无法创建 const 类型的对象
    - 直观理解就是：你在创建对象时希望这个对象是 const，也就是 **不可修改的，** 但你的 **函数** 没加 const，即函数认为是可修改的，这就会矛盾；
    - 所以如果你不创建 const 类型的对象，对于不会修改值的函数不加 const 也不会报错，但不推荐！
- **写代码的习惯：对于不会修改变量值的函数，一律写成常量函数**

### 7.参数传递：传值、传引用、传 const 引用

- 值传递：就是把需要传递的变量一整个搬过去，压到栈中，不论这个参数有多大，比如值传递一个大小为 100 的整型数组，那么 400B 的数据就会被压栈
- 引用传递：传引用就理解为传指针，也就是传这个参数的 **地址**
- 写代码的大方向：所有的参数传递都传引用，省空间，效率更高
- 都传引用的问题：我们知道，值传递在函数内部是不能修改参数值的，而引用传递可以，那 **如何在传引用的前提下，还要求函数不改变参数的值？答案就是传 const 引用**，比如之前写过的按 pair 的 key 排序的函数

```cpp
auto cmp = [](const pair<int,int> &pair1 , const pair<int,int> &pair2) {
    return pair1.first < pair2.first;
};
```

- cosnt 表示即使传的是引用，但是在函数内部也不能修改这个参数的值
- 函数返回类型也有返回值以及返回引用，写代码时，如果 **可以返回引用**，那就返回引用，即尽量返回引用
    - 什么时候不能返回引用：返回局部对象的引用 ❌，返回临时对象的引用 ❌
    - 什么时候可以返回引用：当函数返回的是一个 **在函数外仍然有效的对象，** 则可以返回引用

### 8.友元

- ==友元函数可以使用类中 private 内的变量（protect 和 public 自然也可以），友元函数的权限比子类要更强一点，子类只能访问 public 和 protect==
- 这其实是打开了 C++封装的一个大门
- 如果不使用友元，则只能使用类中 public 中提供的方法去拿 private 中的变量，本例中就是 **`real()`** 和 **`imag()`** 两个函数
- **同一个 class 的不同对象之间互为友元**

```cpp
class complex {
public:
    complex() : re(0),im(0){}
    complex(double r,double i) : re(r),im(i) {}

    int add(const complex& obj) {
        return obj.re + obj.im;
    }

    double real() const {return re;}
    double imag() const {return im;}

private:
    double re,im;//实部和虚部
};
int main() {
    complex c1(1,2);
    complex c2 (2,3);
    int res = c1.add(c2); //c1 可以访问 c2 的 private 变量
    cout << "res = " << res;

    return 0;
}
```

### 9.操作符重载 1 - 成员函数

- 任何成员函数，都有一个隐藏的参数 - this 指针（除了静态函数），谁调用这个函数，this 指针就指向谁的地址
- this 是一个指针类型，所以如果函数的参数有 this，则参数类型需要为指针类型而不是引用类型
- **操作符重载：实际上就是一个函数，有返回值，有参数，函数名就是需要重载的操作符，除此之外，在操作符前加上关键字 *operator***
- **`*ths`** 本身在表达式语义上是一个左值（lvalue），可以安全地返回引用。关键在于：**这个对象还在外部“活着”**，生命周期没结束！

```cpp
class complex {
public:
    complex() : re(0),im(0){}
    complex(double r,double i) : re(r),im(i) {}
    double real() const {return re;}
    double imag() const {return im;}

private:
    double re,im;//实部和虚部

public:
		//ths 是一个指针，*ths 是一个 complex 类的对象，返回* ths 就相当于返回了一个对象，所以返回值是 complex&
    complex& __doapl(complex* ths,const complex& r) {
        ths->re += r.re;
        ths->im += r.im;
        return *ths;
    }
    //this 是一个指针，所有上面必须传一个指针，不能传引用
    complex& operator += (const complex& r) {
        return __doapl(this,r);
    }
};
int main() {
    complex c1(1,2);
    complex c2 (2,3);

    c1 += c2;
    cout << c1.real() << "+" << c1.imag();//3 + 5
    return 0;
}

```

- 调用函数的对象，不需要知道函数的返回值是什么 ，比如在执行 c1 += c2 时，c1 无需知道返回值是 complex&类型的，如果仅执行 c1+= c2 的操作，操作符重载对应函数的返回值也可以是 void，但需要考虑使用操作符的多种情况，比如实现 c1 += c2 += c3，就需要要求返回值是 complex&类型

### 10.操作符重载 2 - 非成员函数

- 和成员函数操作符重载的不同：成员函数可以使用 this 指针

```cpp
//class 之外 ①：全局函数
inline double img(const complex& x) {
    return x.imag();
}
inline double real(const complex& x) {
    return x.real();
}
//class 之外 ②：操作符重载
inline complex operator + (const complex& c1,const complex& c2) {//复数 + 虚数
    return complex(real(c1) + real(c2),img(c1) + img(c2)); 
}
inline complex operator + (double x,const complex& c1) {//复数 + 实部
    return complex(real(c1) + x, img(c1));
}
inline complex operator + (const complex& c1,double y) {
    return complex (real(c1),img(c1) + y);
}

```

- 共轭复数 & 重载 << 输出
    - 现代 C++能够 cout 各种类型的数据，实际上就是对 << 做了很多操作符重载 string
    
    ```cpp
    //共轭复数
    inline complex conj (complex& x) {
        return complex(real(x),-img(x));
    }
    //重载 <<，输出复数
    inline ostream & operator << (ostream& os,const complex& x) {
        return os << '(' << real(x) << ',' << img(x) << 'i' << ')';
    }
    ```
    

### 11.临时对象(temp object)

- 用法： **`typename + ()` , typename 可以是任何类型，当然也可以是类**
- 比如上面的三个函数返回 **`complex(real(c1)_+x,img(c2))` ，** 就是创建了一个临时对象，这个对象的 **生命周期在本行代码执行完就结束**
- 所以上面三个函数一定不能返回引用

### 12.复数类总结 - 在写一个类时需要注意的地方

- 构造函数：学会使用 **初始列**
- 函数的参数传递：引用传递，是否需要加 const
- 函数的返回值：是否可以 **return by reference**
- 函数定义：是否需要标记为 **const** ⭐
- 操作符重载：定义在类内还是类外

### 13.拷贝构造、拷贝赋值、析构（big three）

- 字符串类的设计：使用一个指针指向字符串存储的地址，而非直接使用数组存字符串 - 因为字符串长度有大有小，如果在类中设定一个数组成员存字符串，不知道需要设定多大。
- 所以字符串类是一个带指针的类 ，对于带指针的类，一定要自己写出 **拷贝构造&拷贝** 赋值

- 如果不自己写，使用编译器提供的拷贝构造和拷贝复制会怎样：以下面为例，如果使用编译器的复制，会使 b 的指针指向 a 指向的地方，导致 b 原来指向的东西没有指针指向了 **（内存泄漏）**，b 和 a 共同指向一个位置，其中一个析构也会出现问题，这种拷贝称为 **浅拷贝（只拷贝指针），** 我们后面自己实现的拷贝复制，可以实现 **深拷贝**

<img src="https://raw.githubusercontent.com/mumushu1/Pictures/main/4e53f407045d57fe54e97c245dc6d5f7.png" alt="image-20250515112654042" style="zoom: 67%;" />	

```cpp
//构造函数
inline String ::String(const char* str) {
    if (str) {
        m_data = new char[strlen(str) + 1];
        strcpy(m_data,str);
    }
    else {
        m_data = new char[1];
        *m_data = '\0';
    }
}
//拷贝构造函数：直接取目标 str 的内容作为自己的内容（深拷贝）
inline String ::String(const String &str) {
    m_data = new char[strlen(str.m_data) + 1];
    strcpy(m_data,str.m_data);//strcpy 将指针指向的内容复制
}
//拷贝赋值
inline String &String::operator=(const String &str) {
    //检测自我赋值：这一步的必要的，如果自己拷贝自己，没有这一步直接执行 delete [] m_data，会将自己删掉，无法进行后面的拷贝工作
    if (this == &str) {
        return *this;
    }
    delete[] m_data;
    m_data = new char[strlen(str.m_data) + 1];
    strcpy(m_data,str.m_data);
    return *this;
}
//析构函数
inline String :: ~String() {
    delete[] m_data;
}
```

### 14.static

- 静态数据&静态函数：一个类只有一个
    - 静态函数没有 this 指针作为参数（其他成员函数默认有 this 指针作为参数）
    - **静态函数只能处理静态数据**
- 调用静态函数&静态变量的两种方式
    - 通过对象调用（和普通成员变量&成员函数调用方法相同）
    - 通过类名直接调用 **`Account::m_rate`**
- 需要使用静态的场景：比如账户类，每个成员可以有自己的存款、利息，但每个人的利率是相同的，也就是一个类中只需要一个利率，那么就把这个利率设为静态

```cpp
class Account {
public:
    static double m_rate;//利率

    static void set_rate(const double& x) {
        m_rate = x;
    }
};
double Account::m_rate = 9.0;//如何赋值

int main() {
    Account obj;
    cout << obj.m_rate << endl;

    //直接使用类名访问静态成员函数和静态成员变量
    Account::set_rate(5.0);
    cout << Account::m_rate;
}
```

### 15.模板

- 类模板：类的成员变量可以有不同的类型

```cpp
template<typename T>

class lr_template {
public:
    lr_template(const T& r,const T& i) : re(r),im(i){}
    T get_re() const {return re;}
    T get_im() const {return im;}

private:
    T re;
    T im;
};

int main() {
    lr_template<int> c1(12,3);
    lr_template<float> c2(2.1,32.4);
}
```

- 函数模板

```cpp
class Stone {
public:
    Stone(int w,int h,int weight) : w(w),h(h),weight(weight){}

    bool operator < (const Stone& sto) const {
        return this->weight < sto.weight;
    }

    int get_weight() const {
        return this->weight;
    }

private:
    int w,h,weight;
};

template<typename T>
inline const T& min_cl(const T& a,const T& b) {
    return a < b ? a : b;
}

int main() {
    Stone s1(1,1,2);
    Stone s2(1,1,3);
    //调用时自动推导参数类型
    auto res = min_cl(s1,s2);
    cout << "较小的weight为:" << res.get_weight() << endl;

    return 0;
}
```

- 类模板和函数模板总结
    - 类模板用于写 **通用的数据结构类，** 函数模板用于写 **通用算法函数**
    - 类模板在创建对象时必须 **指定** 类型，函数模板在调用时 **自动推导类型**

### 16.命名空间

- 作用：避免命名冲突，在同一命名空间中，不能出现名字相同功能不同的函数（除了重载），不同命名空间中则可以
- C++的标准库定义在 std 命名空间中

```cpp
#include <iostream>
namespace MySpace{
    void printf() {
        std::cout << "This is my namespace" << std::endl;
    }
}

int main() {
	using namespace MySpace;
  printf();//This is my namespace
}
```

- 三种指定命名空间的方法

```cpp
//法一
using namespce std;

//法二：仅使用命名空间中的某几个函数
using std::cout;// from xx import xx

//法三：直接在使用前声明
std::cout << "hello" << std::endl;
```

以上内容是关于 **基于对象的设计**，即设计单一的类，无论这个类是否带指针，下面的内容是关于 **面向对象的设计**，即类与类之间具有 **关系** 

### 17.类与类之间的关系

- composition（复合）
- inheritance（继承）
- delegation（委托）

### 18.composition 复合（has-a 的关系）

- 复合是指 **一个类中有另一个类的对象作为成员变量**
- 比如队列 queue 和双端队列 deque，一个类的成员函数直接使用另一个类的成员函数
- 形成的设计模式：***adapter 适配器***，使用复合来实现“行为转发“，即下面 queue 类中 deque 类的对象调用自己的函数
  
    <img src="https://raw.githubusercontent.com/mumushu1/Pictures/main/b90145630b1d47c4f27e408c59f96b90.png" alt="image-20250515112731756" style="zoom:80%;" />		
    
- 下面的例子就类似于 DS 中的邻接表数据结构，在一个 struct ALGraph 中存在 struct VNode 类型的对象

<img src="https://raw.githubusercontent.com/mumushu1/Pictures/main/9a6802ef0b67068f34c39becba764933.png" alt="image-20250515112754358" style="zoom: 80%;" />	

```cpp
typedef struct ArcNode{
	int data;
	struct ArcNode* next;
}ArcNode;

typedef struct VNode{
	int data;
	struct ArcNode* first;
}VNode;

typedef struct{
	struct VNode* List[maxsize];
	int numv,nume;
}ALGraph;
```

- 复合关系下的构造和析构（编译器执行的顺序），以 queue 和 deque 为例
    - 构造由内而外：先调用 deque 的 **默认构造函数**，再调用 queue 的构造函数（这个应该不难理解）
    - 析构由外而内：先调用 queue 的析构函数，再调用 deque 的构造函数（想象成剥洋葱，先拿掉外面的瓣再拿里面的瓣）
- **类之间的复合关系，就是一个类作为另一个类的“成员变量”，构成“整体–部分”的结构，复合对象随宿主类一起创建和销毁，生命周期绑定在一起。**

### 19.delegation 委托

- 概念：一个对象 **将某个操作的实现委托给另一个对象来完成**，即“功能代理”。
- 如何理解委托：即 **composition by reference，** 也是一个类中含有另一个类的东西，只不过含有的不是另一个类的对象，而是另一个类的指针
- 这样做的意义：主类给外界提供接口，至于主类的函数，都是另一个类实现的，主类中包含这个类的指针来调用自己的函数，**接口与实现严格地分开**

<img src="https://raw.githubusercontent.com/mumushu1/Pictures/main/5d811b776eb5846182c42f82be2c77ce.png" alt="image-20250515112828966" style="zoom:80%;" />	

- 复合 VS 委托
    - **复合** 是“我有一个你”
    - **委托** 是“我让你替我做这事”

### 20.inheritance 继承（is-a 的关系）

- 概念：子类继承父类的成员变量和成员函数
- 子类对象中有一部分是父类的东西（可以类比数据库中的子类 - 研究生和本科生的父类是学生，它们都具有学生的属性，加上他们自己独有的属性）
- 继承关系下的构造和析构（编译器的执行顺序）
    - 构造：由内（父类）而外（子类）
    - 析构：由外（子类）而内（父类）
- 进一步剖析继承关系下子类和父类的构造函数
    - 如果父类使用默认构造，则在子类的构造函数中自动调用父类的默认构造
    - **如果父类自己写了构造，则在子类的构造函数中必须显示调用父类的构造函数**

```cpp
class Person {
public:
    Person(string i,string n) : id(i),name(n){}
    void print() const {
        cout << "I'm a person" << endl;
    }
    string res_id() const {
        return this->id;
    }
    string res_name() const {
        return this->name;
    }
protected:
    string id;
    string name;
};

class Student : public Person {
public:
    ****//父类不使用默认构造，则子类的构造函数列表需要调用父类的构造函数
    Student(string id,string name,string school,int grade) : Person(id,name),school(school),grade(grade){}
    string res_school () const {
        return this->school;
    }
    int res_grade() const {
        return this->grade;
    }
private:
    string school;
    int grade;
};
int main() {
    Student s1("101","nap_latte","pku",500);
    cout << "学生的姓名：" << s1.res_name() << endl;
    cout << "学生的学校：" << s1.res_school() << endl;
    cout << "学生的成绩：" << s1.res_grade() << endl;
//    学生的姓名：nap_latte
//    学生的学校：pku
//    学生的成绩：500
    return 0;
}
```

- 三种继承方式
  
  
    | 继承方式 | 父类 public 成员在子类中变成 | 父类 protected 成员在子类中变成 | 变化 |
    | --- | --- | --- | --- |
    | public | public | protected | 不变（is-a 的关系） |
    | protected | protected | protected | 全变为 protect |
    | private | private | private | 全变为 private |

### 21.虚函数

- virtual 函数：希望子类重写（override）这个函数（这个函数在父类已有默认定义）
- pure virtual（纯虚函数）：子类必须重写这个函数（函数在父类中无默认定义）
- 普通函数：不希望子类重写的函数
- 抽象类：包含纯虚函数的类，比如下面的 Shape 类，抽象类只能被继承和实现，不能创建对象（实例化）

```cpp
class Shape {
public:
    virtual void draw() const = 0;//纯虚函数语法：函数 = 0
    virtual void error(const string& msg){//虚函数
        cout << msg << endl;
    }
};

class Rectangle : public Shape {
public:
    void draw() const override {
        cout << "Rectangle draw!" << endl;
    }
};
class circle : public Shape {
public:
    void draw() const override {
        cout << "circle draw!" << endl;
    }
};
int main() {
    Rectangle re;
    circle ci;
    re.draw();
    ci.draw();
    re.error("default error!");
//    Rectangle draw!
//    circle draw!
//    default error!
    return 0;
}
```

- 设计模式之 **Template Method**（模版方法）
    - **在父类中定义算法的骨架，而将其中的某些具体步骤推迟到子类实现**（子类通过重写父类的虚函数）
    - 比如下面的例子，我们需要完成打开文件这个算法，这些算法中有一些通用步骤，比如说检测路径名、文件名等等，但也有些特殊步骤，比如针对文件格式会有不同的处理方法，我们将实现这个方法的函数命名为 Serialize，在父类将其声明为虚函数，在子类中再重写实现具体步骤
    - 下面 main()函数的执行步骤
        1. 子类对象调用父类的 onFileOpen()函数
        2. 执行父类的 onFileOpen()函数，执行到 Serialize()时，跳转到子类对这个虚函数的实现
        3. 执行子类 override 的 Serialize()后回到父类，执行完剩余的 onFileOpen()函数后返回到 main()

<img src="https://raw.githubusercontent.com/mumushu1/Pictures/main/9a0f11a9b01bfd6b7c471dd8de1f205d.png" alt="image-20250515112857913" style="zoom:80%;" />	

- **复合关系 + 继承关系下，编译器执行构造函数的顺序**：① 父类构造函数 ② 子类包含的类的构造函数 ③ 子类构造函数

```cpp
class Shape {
public:
    Shape() {
        cout << "Shape（父类）的构造函数执行了" << endl;
    }
};
class special_rectangle {
public:
    special_rectangle() {
        cout << "special_rectangle（子类复合关系包含的类）的构造函数执行了" << endl;
    }
};

class Rectangle : public Shape {
public:
    special_rectangle sr;
    Rectangle() {
        cout << "Rectangle（子类）的构造函数执行了" << endl;
    }
};

Rectangle re;//子类实例化
```

<img src="https://raw.githubusercontent.com/mumushu1/Pictures/main/3e254ef4290246fe80ddbd5de54d2a11.png" alt="image-20250515112922558" style="zoom:80%;" />	

- 析构次序则完全相反

### 22.观察者模式

- 类似于 Notion 中数据库的不同视图，有表格视图，日历视图，状态栏视图等等
- 当一个对象(Subject)的状态发生变化时，其他所有依赖者(Observer 的子类)都会收到通知并自动更新
- `Observer` 是一个抽象基类，定义了统一的接口 `update()`，不同子类继承 Observer 并重写 update()实现不同视图的更新
- Subject 更新后，需要让所有观察者的数据也更新，但不同观察者更新数据的方式可能不同，因此 Subject 中设置了 Observe *类型的指针，将 update 的任务* *委托**给 Observer

```cpp
class Observer;

class Subject {
    int m_value;
    vector<Observer*> m_views;
public:
    //添加新视图
    void attach(Observer *obs) {
        m_views.push_back(obs);
    }
    //设置 Subject 的新值，并自动通知所有观察者
    void set_val(int value) {
        m_value = value;
        notify();
    }
    //遍历所有观察者，调用它们的 update() 接口，将 subject 对象和 m_value 的值传给观察者
    void notify();
};

class Observer {
public:
    virtual void update(Subject* sub,int value) = 0;//每个观察者都有自己的更新方式，因此设置为虚函数
};

inline void Subject::notify() {
    for (int i = 0; i < m_views.size(); i++)
        m_views[i]->update(this, m_value);
}
```

- 以 Notion 中数据库的不同视图大概举一个例子

```cpp
//被观察者，相当于上面的 subject
class Database {
	vector<Observer*> m_views;
}

class Observer {}//观察者的父类

class Table : public Observer{}//表格视图
class Calendar : public Observer{}//日历视图
class StatusBar : public Observer{}//状态栏视图
```

### 23.组合模式

- 可以让“组合对象”和“单个对象”具有 **一致接口** 进行操作，即可以像处理单个对象一样，统一处理复杂的组合结构
    - 组合对象：类似文件系统中的文件夹
    - 单个对象：类似文件系统中的单个文件

```cpp
class Component {
    int value;
public:
    Component(int val) {value = val;}
    virtual void add(Component*){}
};

//组合对象 - 类似文件系统中的文件夹
class Composite : public Component{
    vector<Component*> c;
public:
    Composite(int val) : Component(val){}
    void add(Component* elem) {//文件夹中可以加入文件
        c.push_back(elem);
    }
};

//叶子结点 - 类似文件系统中的文件
class Primitive : public Component {
public:
    Primitive(int val) : Component(val) {}
};

```

### 24.转换函数

- 转换函数是一种特殊的成员函数，用于将一个 **类的对象转换为其他（你认为合理的）类型**
  - 比如一个分数 class，可以想到将其转化为 double 类型
  - 也可以将其转换为字符串类型，只要你认为这么做合理即可
- 这样的好处就是可以让 **自定义的类对象像内置数据类型一样使用**
- 格式：**关键字 operator 后直接跟转换类型，相当于转换类型作为函数名，因为函数名本身就是返回类型，因此函数不需要声明返回类型**
- 一般需要声明为 const，不会修改当前对象

```cpp
class Fraction {
public:
    Fraction(int n,int d) : numerator(n),denominator(d){}
    operator double() const {
        return (double)numerator / denominator;
    }
    
private:
    int numerator; //分子
    int denominator;//分母
};

int main (){
    Fraction f(1,2);
    double res = 3 + f;
    cout << res << endl;//3.5
    return 0;
}
```

- 程序执行 **`double res = 3 + f`** 的过程
  - 因为执行的是 int 型 + 类的对象，C++本身无法提供这种加法，所以进行以下操作
  - 首先看是否有关于 **① 运算符+的重载**，找是否有 `double operator + (Fraction f,double d){}`
  - 没找到后再去看类成员函数中是否有 **② 转换函数**，即是否可以将类对象转换为其他类型

### 25.non-explicit one argument constructor 非显示单参数构造函数

- 如果想实现 `Fraction res = f + 3`，即一个分数类对象 + 整数，返回分数类对象，编译器在执行时，执行顺序如下
  - 首先找是否有分数类对象+整数的重载 - 没找到
  - 其次尝试是否可将**右操作数**(3)转换为分数 - 找到非显示单参数构造
- 编译器会 ==优先保证左操作数的类型不变==，所以不会先考虑去找转换函数将左操作数 f 转为 int

```c++
class Fraction {
public:
    Fraction(int n,int d = 1) : numerawtor(n),denominator(d){}
    //加法重载
    Fraction operator + (const Fraction& f) {
        //实现两个分数相加的动作，下面实现过程不重要
        Fraction res = Fraction(1);
        return res;
    }
private:
    int numerawtor;
    int denominator;
};

int main() {
    Fraction f(1,2); // 1/2
    Fraction res = f + 3;
    return 0;
}
```

- 但如果（非显示单参数构造+加法重载）和（转换函数）并存，此时执行 `Fraction res = 3 + f`，就会报错
  - 前者是将整数-> 分数类对象，后者将分数类对象-> double，理论上都可完成 3+f 的操作
  - 编译器面对多种可行方案时，就会报错（这点很模糊很难，先放一下）

```c++
//加法重载
Fraction operator + (const Fraction& f) {
    //实现两个分数相加的动作，下面实现过程不重要
    Fraction res = Fraction(1);
    return res;
}
//转换函数
operator double() const {
    return (double) numerawtor / denominator;
}
```

- 如果给构造函数加上关键字 `explicit`，表示不允许构造函数自动执行像刚刚一样 3 转换为 $\frac{3}{1}$ 的操作

```c++
//显示单参数构造函数
explicit  Fraction(int n,int d = 1) : numerawtor(n),denominator(d){}
```

### 26.pointer like classes 像指针一样的类(对象)

- ==本质上是一个类，但类里面会有一个真正的指针==

<img src="https://raw.githubusercontent.com/mumushu1/Pictures/main/7222a7089d8c60698acd7e8a9001f3be.png" alt="image-20250523094705779" style="zoom:80%;" />

- 关于 **智能指针**，本质是 **”自动管理资源的类“**，标准库的智能指针都是一个个 **类模板**，是 **封装了原始指针并重载操作符的类模板**
  - 下面的例子中，`T* px` 是类中真正的指针
  - `share_ptr<Foo> sp (new Foo)`
    - `(new Foo)` 即调用类构造函数，表示 Foo 类型对象的指针赋予为指针 px
    - 创建类的对象 sp，指定 T 为 Foo 结构体类型，即对象的 px 指针指向一个 Foo 类型的结构体
  - `Foo f(*sp)`
    - `*sp` 是调用类中对 *的重载，返回当前对象的* px 指针，而 px 在上文提到是一个指向 Foo 结构体类型的指针， *px 就是指向的结构体
    - 所以 `Foo f(*sp)` 相当于是进行了一步 **拷贝构造**，创建了一个新的结构体对象 f，f 的“值”就是*sp 指向的结构体
  - `sp->method()`
    - sp 是类对象，sp-> method()就是类对象调用类成员函数
    - 只不过在类中对 `->` 做了重载，会返回 px 指针，而 `->` 会继续往下执行 ==（操作符的链式调用）==，即相当于执行 `px->method()`
    - 而 px 是指向 Foo 结构体的指针，当然可以调用 method()

```cpp
template<class T>
    
class share_ptr {
public:
    T& operator *() const{
        return *px;
    }
    T* operator ->() const {
        return px;
    }
    share_ptr(T* p) : px(p) {}

private:
    T* px;
    long* pn;
};
struct Foo {
    void method() {
		cout << "method已被调用" << endl;
    }
};
int main() {
    share_ptr<Foo> sp (new Foo);
    Foo f(*sp);

    sp->method();

    return 0;
}
```

- 关于 **迭代器**

  - 下面是一个迭代器的简单用法

  ```cpp
  list<int> p = {1,2,3,4,5};
  for (list<int>::iterator it = p.begin(); it != p.end(); it++) {
      cout << *it << " " ;
  }
  ```

  - 迭代器也和上面一样，都在类中做了操作符重载，在*it 时，我们希望直接取到容器中的 data，而不是 next 或 prior 指针
  - `return &(operator *());`
    - operator *()调用上面的*()函数，即返回 data 的引用
    - &(operator *())：取 data 的引用的地址，* *会得到实际 data 的地址**，因为引用没有独立的地址
    - 所以返回的指针，这个指针就是指向 data

  ```CPP
  //通过迭代器获取当前节点的值
  reference operator *() const {
      return (*node).data;
  }
  
  //返回指向当前data的指针
  pointer operator ->() const {
      return &(operator *());
  }
  ```

### 27.function like class 像函数一样的类 （仿函数）

- 仿函数本质上是 **重载了()的类 or 结构体**，为什么会有仿函数，这个以后再说，先关注语法层面

- 一个小例子：调用仿函数时，直接`对象+()即可`

```cpp
template<typename T>

class Add {
public:
    T operator () (T a,T b) {
        return a + b;
    }
};
int main() {
    Add<int> add_int;
    cout << add_int(3,4);

    Add<float> add_float;
    cout << add_float(2.2,4.2);

    return 0;
}
```

### 28.模板

#### ① 类模板&函数模板

- 类模板：类的成员变量可以有不同的类型
- 函数模板：一个模板函数，函数的参数可以有很多类型，但一次调用时，只能指定一种类型，比如本例中的 min1 函数，只能两个 int 比或两个 double 比，不能 int 和 double 比，这样 **模版推导会失败**
  - 如果需要两个不同类型的参数，异或是更多，需要写成两个或多个模板参数 `template <typename T1, typename T2>`

- **class VS typename**
  - C++中 `class` 和 `typename` 在模板参数列表中，**完全等价**，`template<class T>` 和 `template<typename T>` 是一样的
  - **除了** 在模板内部的 **嵌套依赖类型中**，必须使用 `typename`，这部分以后再聊，现阶段知道二者大多情况无区别


```cpp
template<typename T>

class Stone {
private:
    T weight;
    T height;
public:
    Stone(int w,int h) : weight(w),height(h){}

    bool operator < (const Stone &sother) const{
        return this->height < sother.height;
    }

    T get_height() const{
        return this->height;
    }
};

template<typename T>
inline const T& min1(const T& a,const T& b) {
    return b < a ? b : a;
}

int main() {
    Stone<int> s1(1,3);
    Stone<int> s2(1,4);
    //模板函数要求两个参数类型一样
    auto more_min = min1(s1,s2);
    cout << more_min.get_height();
    return 0;
}
```

- 类模板和函数模板总结
  - 类模板用于写 **通用的数据结构类，** 函数模板用于写 **通用算法函数**
  - 类模板在创建对象时必须 **指定** 类型，函数模板在调用时 **自动推导类型**

#### ③ 成员模板

> 这里相当模糊，等之后深造吧

- 定义：**一个类（或结构体）中的成员函数是模板函数**
- 下面的代码：先常创建 <iPhone,MacBook> 类型的 pair 对象 Apple，之后使用 Apple 创建了 baseCombo，这个 baseCombo，类型是 <Phone,Laptop> 但其值还是 Apple 的值，即 **内容是子类，类型是父类**
- 这样的作用：支持不同类型对象之间的拷贝（只要他们的类型可以转换：子类可转换为父类，但父类不能转换为子类），**可以用类型不同的对象去构造另一个模板对象，只要他们类型可以转换**

```cpp
class Phone {
public:
    void call() const {cout << "Calling" << endl;}
};

class Laptop {
public:
    void code() const {cout << "Coding" << endl;}
};

template <typename T1,typename T2>
struct Combo {
    T1 item1;
    T2 item2;
    Combo(){}//默认构造
    //成员模版构造
    template <typename U1,typename U2>
    Combo(const Combo<U1,U2>& other) : item1(other.item1),item2(other.item2){}
};

class iPhone : public Phone {};
class MacBook : public Laptop {};

int main() {
    Combo<iPhone,MacBook> Apple;//一个关于苹果公司的手机+笔记本产品组合
    Combo<Phone,Laptop> baseCombo(Apple);//用“苹果组合”构造出基础组合（手机+笔记本组合）
    //basepack的类型是<Phone,Lapop>类型，但是basepack的值还是iPhone和MacBook的值直接拷贝过来的，也就是内容是子类，但类型是父类！
}
```

#### ④模板特化

- 是什么：在编写泛型代码时，有时想为**某些特定类型定义不一样的实现**，就需要用到模板特化

```cpp
template<typename T>
struct MyPrint { //模板类 - 泛化
    void print() {
        cout << "generic type" << endl;
    }
};
template<> //注意这里没有参数
struct MyPrint<int> { //特化模板（只适用于int） - 特化
    void print() {
        cout << "int type" << endl;
    }
};

int main() {
    MyPrint<float> m1;
    MyPrint<int> m2;
    m1.print();//"generic type"
    m2.print();//"int type"
    return 0;
}
```

- 好处：为某些类型提供更高效的实现；对特定类型特殊处理；避免写多个重复的类/函数

#### ⑤ 模板偏特化

- 是什么：只对模板的一部分参数进行特化，**保留其他参数的泛型性**，上面的特化是全特化，与之对应就是现在的偏特化，偏特化是介于“通用模板”和“全特化”之间的一种灵活方案
- ==个数上的偏特化==：即多个参数时，特化部分参数

```cpp
template<typename T1,typename T2> //泛型模板
struct Type {
    static void print() {
        cout << "general template" << endl;
    }
};

template<typename T> //第一个参数任意类型，指定第二个参数为int类型
struct Type<T,int> {
    static void print() {
        cout << "偏特化：第二项为int类型" << endl;
    }
};

int main() {
    Type<double,string> t1;
    Type<double,int> t2;
    t1.print();//"general template"
    t2.print();//"偏特化：第二项为int类型"
    return 0;
}
```

- ==范围上的偏特化==：没有具体指定某个参数特化，而是给出一个**范围**，比如对指针类的所有类型都匹配这个特化模板

```cpp
//通用版本
template<typename T>
struct C {
    void print() const{
        cout << "非指针的输出" << endl;
    }
};

//偏特化版本
template<typename U>
struct C<U*> {
    void print() const{
        cout << "指针的输出" << endl;
    }
};

int main() {
    C<int> c1;
    C<int*> c2;
    c1.print();//非指针的输出
    c2.print();//指针的输出
    return 0;
}
```

#### ⑥ 模板模板参数

- 是什么：一个模板的参数列表中，允许将另一个模板当做参数传入，下面通过一个例子理解
- 一个容器模板

```cpp
template<typename T>
class Container {
public:
    void add(const T& x){}
    T get(int i) const {}
};
```

- 下面针对这个容器模板设置一个管理者模板，其本身也是个模板，需要的参数为
  - 一个模板参数：即需要管理的容器模板，比如`vector`,`list`
  - 一个类型模板：容器中存放的元素类型，比如`int`,`stirng`
  - `template<template<typename> class C,typename T>`语法记一下

```cpp
//容器模板Container
template<typename T>
class Container {
public:
    void add(const T& x){}
    T get(int i) const {}
};

//管理者模板Manger
template<template<typename> class C,typename T>// “C 是一个模板，它本身接受一个类型参数”
class Manger {
    C<T> data;//用传入的容器模板C来实例化一个对象
public:
    void add(const T& x) {data.add(x);}
    T get(int i)const {return data.get(i);}
};

int main() {
    Manger<Container,int> m1;//用自定义的Container存int
    Manger<vector,int> m2;//用vector存int
    Manger<list,int> m3;//用list存int
}
```

### 29.值、指针和引用

#### ①概念初窥

```cpp
int x = 5;
int* p = &x;//&x是对x取地址
int& r = x;

cout << "p = " << p << endl;//p = 0x8b2cfff744
cout << "*p = " << *p << endl;//*p = 5
cout << "&p= " << &p << endl;//&p= 0x8b2cfff738
cout << "r = " << r << endl;//r = 5
```

- 指针也是老生常谈了，p是指针 也是个地址，*p即p指向的值，&p即对指针取地址，也就是指针的地址，下面主要谈下引用
- **定义引用时必须要设初值**，且设完之后就不能绑定到其他对象上了（对比指针可以随意换人绑定）
- 定义指针时也需要初始化，否则它们可能包含垃圾值，导致未定义行为

```cpp
int y = 3;
r = y;//这里不是让r绑定到y上，上面说了引用不能换人，这里相当于是将y的值赋给x⭐⭐⭐
cout << x;//可以看到x的值已经变为3了，这里很像函数的参数传入引用，改引用就相当于改其绑定的对象
```

- 引用和值是**一体的两面**，以上面x和r为例，x的地址和r的大小是一样的，地址也是一样的==（全都是假象）==

```cpp
cout << "&x = " << &x << endl;//x的地址 &x = 0x12c01ff66c
cout << "&r = " << &r << endl;//r的地址 &r = 0x12c01ff66c
```

- 指针、引用、指针的引用

```cpp
int x = 22;

int& r1 = x;//r1是x的引用，即r1和x是一个东西

int* p = &x;//p是指向x的指针

int*& r2 = p;//r2是指针p的引用，即r2和指针p是一个东西
                                 ↓
cout << *r2 << endl;//输出22，*r2相当于*p
```

- 指针指向数组时，则指向数组首元素，且指针可以进行++ --的操作**（仅针对数组指针才有自增自减的运算）**

```cpp
int arr[5] = {1,2,3,4,5};

int *p = arr;

cout << "p = " << p << endl;
cout << "*p = " << *p << endl;

p++;

cout << "p = " << p << endl;
cout << "*p = " << *p << endl;

//输出
p = 0x1ddebff9b0
*p = 1
p = 0x1ddebff9b4  可以看到地址加了4B，即一个int的大小
*p = 2
```

#### ②假象的由来

- **语言层面的真相：引用是一个别名，而不是一个对象**
  - 即不占用新的存储空间，所以上面引用和变量的大小一样，地址一样，引用r就是x，x就是r
  - ==所以对r的任何操作都将作用在x上==
- **编译器的实现层面：引用通常表现为常量指针**：浅色箭头，所以引用的实际大小应该是4个字节（32位系统下），和指针大小一致

<img src="https://raw.githubusercontent.com/mumushu1/Pictures/main/651a8f4725abb46c4d0402f93631ddb3.png" alt="image-20250529101317956" style="zoom:80%;" />	

#### ③引用的常见用途

- 函数传参：最好是传引用或常量引用，因为速度更快
  - 解释一下速度问题：在上篇说过速度更快，但上文又提到引用的大小和地址与其绑定的对象是一样的，那为什么更快
  - 上文提到大小和地址一样是假象了，在传参时，如果传引用的话本质上是传了一个指针，对比如果是传值的话，需要将整个值压栈（值很大就很占时空），而指针的大小是固定4B(32位系统下)
  - 引用一般不拿来声明变量or定义对象，一般就是参数传递的时候去用
- 返回类型：能返回引用就返回引用

#### ④函数名相同，传参方式不同的两个函数

- 以下两个函数是不能并存的，在语法层面上，引用就是值的别名
  - 在调用func时，不管是传值还是传引用，调用方式都是`func(a)`，如果二者可以并存，调用`func(a)`时编译器无法分清是调用的哪个函数

```cpp
void func(int a) {}
void func(int& a){}
```

- 复习一下const，相同的两个函数，一个加const ， 一个不加const，二者是可以并存的 

```cpp
void func() const{}
void func() {}
```

#### ⑤左值引用和右值引用

- C++中，左值和右值是表达式的两种基本分类，它们决定了表达式的结果在内存中的位置和状态
  - 左值：具有**持久状态**的对象，有明确的内存地址，可以多次被赋值
  - 右值：**临时的，没有持久状态的值**，通常没有内存地址，或其内存地址在表达式结束后就失效
- 左值引用就是平时最普通的引用，所以解释了为什么定义引用时要设初值，因为左值是具有持久状态、有地址的对象
- 右值引用：允许为右值创建一个引用，这样就可以对右值进行诸如移动语义这样的复杂操作，这部分留到CS106L再深入学习

### 30.语法层面看不到的底层原理

#### ①虚指针和虚表

- 子类继承父类时，子类的对象也会有内存去存储父类private成员，即不能访问 != 不存储，子类不能访问父类private成员，但创建子类对象时，父类private成员也会占内存
- **虚函数、虚表与虚指针**
  - 当类中有虚函数时，对象（内存）中会有一个指向**虚函数表(vtable)**的指针，成为**虚表指针(vtpr)**，既然是指针，位数和地址位数就是一样的，32位系统下占4B
  - 每个含虚函数的类都有一个虚表，用于支持运行时多态
  - ==虚指针和虚表就相当于页表基址寄存器和页表，不太确切，大概这个意思==
  - 每个类的虚函数表，包含自己的虚函数，以及从父类那继承来的虚函数；如果自己重写了父类的虚函数，那么只存储重写后的即可，父类原本的虚函数就不会存储

```cpp
class A {
public:
    virtual void vfun1();
    virtual void vfunc2();
    void func1();
    void func2();
private:
    int m_data1;
    int m_data2;
};

class B : public A {
public:
    virtual void vfunc1();
    void func2();
private:
    int m_data3;
};

class C : public B {
public:
    virtual void vfunc1();
    void func2();
private:
    int m_data4;
};
```

![image-20250531160620045](https://raw.githubusercontent.com/mumushu1/Pictures/main/1bcc529a24c2e6ecdd3059299f3a68e8.png)

- 如果一个指针p指向C类的对象，通过指针调用C类中的虚函数时，编译器发生的动作，用C语言可以表示为`*(p->vptr)[n]`，即通过指针找到虚指针，取其地址就是虚表的地址，`[n]`即为调用虚表中第n个虚函数
- **静态绑定与动态绑定**
  - 绑定：程序在运行时决定调用哪个函数的过程
  - 静态绑定：在**编译期**就决定调用哪个函数，对应汇编中的`call`指令，`call 0x 1A234B7F`，默认情况下，非虚函数都是静态绑定；效率高，但缺乏多态性
  - 动态绑定：在**运行时**才决定调用哪个函数，用于虚函数，**程序在运行时会查虚函数表**，来决定调用哪个类的虚函数，比如上图中如果C对象调用`vfunc1()`，编译器在运行时才查到`vfucn1()`是C类对象的才去调用
  - 动态绑定的三个条件
    - 必须通过指针去调用
    - 指针**向上转型**（以上图为例，就是定义指向A对象的指针，其引用C，即`A* p1 = &c`，子类C->父类A）
      - 类比生活中这很合理，比如狗(子类)->动物(父类)，但动物不能->狗，向上转型是安全的
    - 虚函数

#### ②再谈this

- **简单说：this是一个指针，通过对象调用函数，对象的地址就是this**

```cpp
CDocument::
OnFileOpen(){
    ...
    Serialize();
    ...
}
virtual Serialize();

class CMyDoc : public CDocument{
    virtual Serialize(){...}
};

int main(){
    CMyDoc myDoc;//子类对象
    myDoc.OnFileOpen();
}
```

- 从main函数出发去看，定义了一个子类对象`myDoc`，子类对象调用父类的`OnFileOpen()`
  - 这一步当然是合理的，因为子类继承父类的成员函数
  - 对于成员函数，都有一个默认的参数（this指针），**当`myDoc`调用函数时，这个函数的默认参数是子类对象的this指针**
  - 所以调用此函数的流程是，首先会执行父类的`onFileOpen()`，执行到`Serialize()`时，实际上是`this->Serialize()`，这个this是子类对象的this，`this->Serialize()`即为`*(this->vptr)[n](this)`：前一部分就是动态绑定找虚函数，后面的this是默认参数
- 回顾上面动态绑定的三个条件：首先是指针调用（这里是this指针），其次是向上转型（调用是子类的对象调用），最后是虚函数（完全满足）

#### ③关于动态绑定(Dynamic Binding)

- 前面说过，对于一个静态绑定，对应一个汇编中的call指令，而且**call的一定是一个确定的地址**
- 而对于动态绑定，也是call，但call的内容不同，这里汇编的含义我们就不去了解了，其实现的功能就是`*(p->vptr)[n]`

<img src="https://raw.githubusercontent.com/mumushu1/Pictures/main/b4bd5a0c226347f78a0dfa043e1dfe2d.png" alt="image-20250605094650985" style="zoom:50%;" />	

### 31.再谈const

#### ①const基本概念

- **一句话明白const ：用于指示变量的值不可修改**

- const定义的变量必须初始化，**全局const变量一般存储在只读数据段，局部const变量（例如函数中）则存储在函数栈中**

- 编译器如何处理const变量：**编译器在编译过程中将所有用到该变量的地方都替换成对应的值**

  - 回想一下第33小结关于extern关键字，我们说过头文件用于声明，源文件用于实现
  - 对于const变量，有两种声明（定义）方式
    - 头文件extern声明、源文件再定义（33小节介绍的常规方法，**没副本，所有源文件共享这一个const变量**）
    - 头文件中直接完成对const变量的定义
      - 这样的合理性：编译器会对这个const变量每个用到的地方生成一个**副本**，也就是两个变量（实际上是一个值）
      - 所以不会出现重定义的问题，不会报错

  ```cpp
  const int MAXSIZE = 100;//注意不加extern，extern是用于声明的关键字
  ```

#### ②const与成员函数

- ==在实现一个成员函数时，就需要考虑要不要加const，这是一个可以很早期完成的工作==

|                                             | 对象是const（成员变量不能改变） | 对象不是const（成员变量可以改变） |
| ------------------------------------------- | ------------------------------- | --------------------------------- |
| **调用的函数是const（保证不改变data）**     | ✔️                               | ✔️                                 |
| **调用的函数不是const（不保证不改变data）** | ❌                               | ✔️                                 |

- 加const和不加const，算作“函数签名”的一部分，即加不加const是两个函数，可以共存（对比函数的返回类型，不属于签名的部分）
- **C++规则怪谈：当成员函数的const版本和non-const版本同时存在时，const对象只能调用const版本的，non-const对象只能调用non-const版本的**，即表格右上角的情况在此时不会出现

#### ③const与引用

- 常量也可以绑定引用，称为对常量的引用，对常量的引用不可修改其绑定的对象

```cpp
const int MAXSIZE = 100;
const int& r = MAXSIZE;
```

- 一个常量引用，绑定非常量，是被允许的

```cpp
int x = 1;
const int& r = x;
```

- 以下代码编译是通过的（没有const是非法的），因为用了const，C++会允许**临时变量的绑定**，实际后台：
  - `const int temp  = pi;`
  - `const int &r = temp;`

```cpp
double pi = 3.14;
const int& r = pi;
```

#### ④const与指针

- 指针也是同理的，也可以指向常量，且不可通过指针修改常量值，常量对象只能由**常量指针**指向，非常量对象可以由常量指针指向（有点绕，多读两遍就懂了）
  - ==**常量指针：**指向的内容是常量的指针，不能修改内容（因为指的常量），但可以指向别的地方==

```cpp
const int MAXSIZE = 100;
const int* p = &MAXSIZE;

int x = 1;
const int* q = &x;
```

- ==**指针常量**：本身是常量的指针，不能修改指向的地方（因为指针就是常量），但可以修改内容==

> 二者都是指针，指针常量表示这个指针是个常量，不能改地址（指向谁），但能改指向对象的值；常量指针表示这个指针指向常量，所以不能改指向对象的值，但可以更改地址（指向谁）

#### ⑤顶层const与底层const

- 顶层const：表示**任意对象**是常量，这一点对任何数据类型都适用 - 算术类型 、类、指针等
- 底层const：表示

### 32.详解new和delete

#### ①基本认知

- 对于类的对象而言，new会调用构造函数，delete会调用析构函数
- `new、delete`：分配/释放一个对象
- `new[]、delete[]`：分配/释放一组对象
- new的底层原理
  1. 调用`operator new(size)`分配内存（底层就是malloc）
  2. 在分配好的内存上调用构造函数
- delete的底层原理
  1. 先调用析构函数
  2. 调用`operator delete(ptr)`释放内存（底层就是free(ptr)），ptr指向这片空间的首地址
  3. free通过查找隐藏的头部得知释放的内存块大小
     - delete[]如何知道元素个数：new[]额外分配了空间，在数组前存储了元素个数（用户看不到）
- new需要知道分配多大内存，需要一个size，而delete

- 可以通过重载`operator`来自定义分配器

#### ②重载operator new、operator delete、operator new[]、operator delete[]

- 下面这段代码影响无远弗届，但目前尚未参透，留得之后学习

```cpp
//封装 malloc/free 的通用内存分配器,可以用来分配任何类型的内存
//*void:通用指针类型，可以指向任何类型
//size_t:无符合整数类型
void* myAlloc(size_t size) {
    return malloc(size);
}

void myFree(void* ptr) {
    return free(ptr);
}

inline void* operator new(size_t size) {
    cout << "这是重载的new" << endl;
    return myAlloc(size);
}

inline void* operator new[](size_t size) {
    cout << "这是重载的new[]" << endl;
    return myAlloc(size);
}

inline void operator delete(void* ptr) {
    cout << "这是重载的delete" << endl;
    return myFree(ptr);
}

inline void operator delete[](void* ptr) {
    cout << "这是重载的delete[]" << endl;
    return myFree(ptr);
}
```

- 也可以对类成员函数new、new[]、delete、delete[]进行重载，用做内存池，这里有待继续研究

### 33.extern关键字

- 我们知道，头文件一般作为声明，源文件用来做实现，现在设想这样一种场景，在头文件中定义变量
- `test.h`中定义x,y，`test.cpp`和`main.cpp`都包含`test.h`

```cpp
//test.h
int x = y = 1;

//main.cpp
std::cout << x << y;
```

- 程序在链接时会报错：`multiple definition of x`,`multiple definition of  y`,原因是**每个包含test.h的文件，都会将这段代码赋值到每个源文件中，即test.cpp和main.cpp都定义了一遍x和y**，这就违反了C++的**ODR**
- ODR：One Definition Rule，唯一定义规则，在一个程序中，某个全局变量（或函数、类、模板等），**最多只能有1个定义，但可以在多个地方声明**

> 也就是说C++的include，会把include的那个文件的内容（变量、函数等等）复制一遍，对于变量来说，如果include的那个文件定义了该变量，那么在include它时就会将这个变量再定义一遍，导致出错。

- 如何解决：引入**extern**关键字
- 一个一般化的做法

```cpp
//test.h
extern std::string str;;//不要赋值，赋值后就是定义了

//test.cpp
std::string str = "Hello World";;//对变量赋值完成定义

//main.cpp
std::cout << str
```

- 总结
  - 头文件只做变量的声明，不做变量的定义
  - 头文件声明变量可以采用extern的方式

