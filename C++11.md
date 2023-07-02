##  1.CMyString重写string

```cpp
#include <cstring>
#include <iostream>
#include <vector>
using namespace std;

class CMyString
{
public:
    /* 默认构造函数 */
    CMyString(const char *str = nullptr)
    {
        cout << " CMyString(const char *str = nullptr) " << endl;
        if (str != nullptr)
        {
            mptr = new char[strlen(str) + 1]; // 加1是因为结尾\0
            strcpy(mptr, str);
        }
        else
        {
            mptr = new char[1];
            *mptr = '\0';
        }
    }

    /* 析构函数 */
    ~CMyString()
    {
        cout << " ~CMyString() " << endl;
        delete[] mptr;
        mptr = nullptr;
    }

    /* 左值引用的拷贝构造函数 */
    CMyString(const CMyString &str)
    {
        cout << "CMyString(const CMyString &str)" << endl;
        mptr = new char[strlen(str.mptr) + 1];
        strncpy(mptr, str.mptr, strlen(str.mptr));
    }

    /* 右值引用的拷贝构造函数 */
    CMyString(CMyString &&str)
    {
        cout << "CMyString(CMyString&& str)" << endl;
        mptr = str.mptr;
        str.mptr = nullptr;
    }

    /* 重载拷贝复制运算符 */
    CMyString &operator=(const CMyString &str)
    {
        cout << "CMyString& operator=(const CMyString &str)" << endl;
        if (this == &str)
            return *this;

        delete[] mptr;
        mptr = new char[strlen(str.mptr) + 1];
        strcpy(mptr, str.mptr);
        return *this;
    }

    const char *c_str() const { return mptr; }

private:
    char *mptr;

    /* 声明类的友元函数 */
    friend CMyString operator+(const CMyString &lhs, const CMyString &rhs);
    friend ostream &operator<<(ostream &out, const CMyString &str);
};

CMyString operator+(const CMyString &lhs,
                    const CMyString &rhs)
{
    // char* ptmp = new char[strlen(lhs.mptr) + strlen(rhs.mptr) + 1];  // 此处new需要对应的delete
    CMyString tmpStr;
    tmpStr.mptr = new char[strlen(lhs.mptr) + strlen(rhs.mptr) + 1];
    strcpy(tmpStr.mptr, lhs.mptr);
    strcat(tmpStr.mptr, rhs.mptr);
    // delete[] ptmp;
    return tmpStr;
    // return CMyString(ptmp)
}

CMyString GetString(CMyString &str)
{
    const char *pstr = str.c_str();
    CMyString tmpStr(pstr);
    return tmpStr;
}

ostream &operator<<(ostream &out, const CMyString &str)
{
    out << str.mptr;
    return out;
}

int main()
{
    CMyString str1("Hello ");
    CMyString str2("World!");
    CMyString str3 = str1 + str2;
    cout << str3 << endl;
    return 0;
}

/* 输出结果 */
/*
 CMyString(const char *str = nullptr)
 CMyString(const char *str = nullptr)
 CMyString(const char *str = nullptr)
CMyString(CMyString&& str)
 ~CMyString()
Hello World!
 ~CMyString()
 ~CMyString()
 ~CMyString()
 */

int main()
{
    CMyString str1 = "aaa";

    vector<CMyString> vec;  // 这是外部对象
    vec.reserve(10);
    cout << "---------------------" << endl;
    vec.push_back(str1);  // 拷贝一个已经存在的对象，因该匹配左值引用的拷贝构造
    vec.push_back(CMyString("bbb"));  // 拷贝一个临时对象，应该匹配右值引用的拷贝构造
    cout << "---------------------" << endl;
    return 0;
}

/* 输出结果 */
/*  
 CMyString(const char *str = nullptr)
---------------------
CMyString(const CMyString &str)
 CMyString(const char *str = nullptr)
CMyString(CMyString&& str)
 ~CMyString()
---------------------
 ~CMyString()
 ~CMyString()
 ~CMyString()
 */


```



### 引用折叠

- 右值引用变量本身仍是左值

- 一个函数参数为右值时例如int add(int &&);时
  - 我们向add函数传入左值时，左值引用实体 初始化 右值引用参数仍是一个左值
  - 同样的我们向add函数传入一个右值时，右值引用实体初始化左值引用参数仍是一个右值



### std::move()

- move()函数是一个移动语义，得到右值类型

- 将传入的变量强制转换成为右值引用的变量

  

### std::forward()

- 类型的完美转发，能够识别左值与右值类型
- 将传入的参数分析，如果传入的是一个左值，那么就匹配一个参数为左值的函数
- 如果传入的参数是一个右值，那么就匹配一个参数为右值的函数



有这样一个类:

```cpp
#include <iostream>
using namespace std;

class Test
{
public:
    /* 默认参数的构造函数功能较多 */
    Test(int data = 10) : ma(data)
    {
        cout << "a Test(int data) " << endl;  // a
    }

    ~Test()
    {
        cout << "b ~Test()" << endl;  // b
    }

    Test(const Test &t) : ma(t.ma)
    {
        cout << "c Test(const Test &t)" << endl;  // c
    }

    void operator=(const Test &t)
    {
        cout << "d Test& operator=(const Test &t)" << endl;  // d
        ma = t.ma;
    }

    int getData() const { return ma; }

private:
    int ma;
};

int main()
{
    Test t1;  // 调用构造函数
    Test t2(t1);  // 调用拷贝构造函数
    Test t3 = t1;  // 调用拷贝构造，为什么没有调用等号赋值的拷贝构造呢？，因为t3是以定义的方式被一个对象初始化，所以使用拷贝构造函数
    
	/*
     * 一般来说这条语句应该是先调用构造函数初始化临时对象Test(20)，然后使用拷贝构造初始化t4，最后临时对象析构
     * 但是，C++编译器对于对象构造的优化：用临时对象生成新对象的时候，临时对象不产生了，直接构造新对象
     * 与Test t4(20);没有区别
     */
    Test t4 = Test(20);
    
    cout << "----------------" << endl;
    t4 = t2;  // 调用等号赋值运算符的拷贝构造函数
    
    t4 = Test(30);  // 因为t4已经定义过了，这里会显示生成临时对象
    
    t4 = (Test)30;  // int -> Test, 合法；首先编译器会自动匹配一个参数为int的构造函数，显示生成临时对象,如果没有会编译错误
    
    t4 = 20;  // 也是int -> Test,合法，同样隐式生成临时对象  
    
    
    cout << "================" << endl;
    // Test *p = &Test(40);  // p指向的是一个已经析构的临时对象报错，右值无法获得地址
    
    const Test &ref = Test(50);  // 左值引用无法匹配右值引用，报错，应该加上const
    
    cout << "================" << endl;
    return 0;
}

/* 预计输出：acca---dadb--adba--dbbbbb */


```



观察分析代码：

```cpp
#include <iostream>
using namespace std;

class Test
{
public:
    /* 默认参数的构造函数功能较多 */
    Test(int a = 5, int b = 5) : ma(a), mb(b)
    {
        cout << "a Test(int, int) " << endl; // a
    }

    ~Test()
    {
        cout << "b ~Test()" << endl; // b
    }

    Test(const Test &src) : ma(src.ma), mb(src.mb)
    {
        cout << "c Test(const Test&)" << endl; // c
    }

    void operator=(const Test &src)
    {
        cout << "d Test& operator=(const Test &)" << endl; // d
        ma = src.ma;
        mb = src.mb;
    }

    int getData() const { return ma; }

private:
    int ma;
    int mb;
};

Test t1(10, 10); // a

/* a为构造， b为析构， c为拷贝构造， d为赋值运算 */
int main()
{
    cout << "---------------" << endl;
    Test t2(20, 20);               // a
    Test t3 = t2;                  // c
    static Test t4 = Test(30, 30); // a

    cout << "---------------" << endl;
    t2 = Test(40, 40);   // adb
    t2 = (Test)(50, 50); // adb
    t2 = 60;             // adb

    cout << "---------------" << endl;
    Test *p1 = new Test(70, 70); // a
    Test *p2 = new Test[2];      // aa

    cout << "---------------" << endl;
    // Test *p3 = &Test(80, 80);   // ab

    cout << "---------------" << endl;
    const Test &p4 = Test(90, 90); // a
    delete p1;                     // b
    delete[] p2;                   // bb
    return 0;                      // bbbb
}
Test t5(100, 100); // 程序开始时全局变量先进行构造

/* aaaca -- adbadbadb -- aaa -- abbbbbbb */

```



观察下面这行代码，写出输出结果

```cpp
#include <iostream>

using namespace std;

class Test
{
public:
    /* 默认参数的构造函数功能较多 */
    Test(int data = 10) : ma(data)
    {
        cout << "a Test(int data) " << endl;  // a
    }

    ~Test()
    {
        cout << "b ~Test()" << endl;  // b
    }

    Test(const Test& t) : ma(t.ma)
    {
        cout << "c Test(const Test &t)" << endl;  // c
    }

    void operator=(const Test& t)
    {
        cout << "d Test& operator=(const Test &t)" << endl;  // d
        ma = t.ma;
    }

    int getData() const { return ma; }

private:
    int ma;
};

/* 不能返回局部变量或临时对象的指针或者引用 */
Test GetObject(Test t)
{
    int val = t.getData();
    Test tmp(val);
    return tmp;  // 两个析构
}

int main()
{
    Test t1;  // 构造函数
    Test t2;  // 构造函数
    t2 = GetObject(t1);  // 在参数处左值引用的拷贝构造， 构造函数， 拷贝构造， 拷贝构造
    return 0;  // 两个析构
}

/* 不同平台有不同差异Windows多了一次析构 */
/* Windows结果 */
/*
a Test(int data)
a Test(int data)
c Test(const Test &t)
a Test(int data)
c Test(const Test &t)
b ~Test()
b ~Test()
d Test& operator=(const Test &t)
b ~Test()  // 这次析构是因为调用=复制运算符会在main函数栈帧上拷贝一个临时对象，进行释放
b ~Test()
b ~Test()
 */

/* Linux结果 */
/*
a Test(int data)
a Test(int data)
c Test(const Test &t)
a Test(int data)
d Test& operator=(const Test &t)
b ~Test()
b ~Test()
b ~Test()
b ~Test()
 */

```



## 2.左值与右值

左值：

- 有地址空间
- 有名字 

右值：

- 没有名字（临时量）
- 没有内存



```cpp
#include <iostream>

using namespace std;


int main(int args, char* argv[]) 
{
    cout <<" --------------" <<endl;
    int a = 20;
    int &b = a;  // 可以引用，因为a是一个左值，可以初始化左值引用
    // int &&c = a;  // 不可以引用，无法把一个左值实体绑定到一个右值引用

    /*
    int temp = 20;
    const int &c = tmp;
    */
    const int &c = 20;

    /*
    int temp = 20;
    int &&d = tmp;
    */
    int &&d = 20;  // 可以引用，可以把一个右值绑定到一个右值引用上，也就是，可以用右值初始化右值引用

    // int &&f = d;  // 不可以引用，因为右值引用变量本身仍是左值
    return 0;
}

```





## 3.智能指针

[施磊老师的博客，深入掌握C++智能指针](https://blog.csdn.net/QIANGWEIYUAN/article/details/88562935)

```cpp
#include <iostream>

using namespace std;

/*
 * 智能指针：保证能做到资源的自动释放！！
 * 利用栈上的对象超出作用域自动析构的体征做到资源释放。
 */
template <typename T>
class CSmartPtr
{
public:
    CSmartPtr(T *ptr = nullptr)
        : mptr(ptr) {}
    ~CSmartPtr() { delete mptr; }

    T &operator*() { return *mptr; }

    T *operator->() { return mptr; }

private :
    T *mptr;
};

int main(int args, char *argv[])
{
    cout << " --------------" << endl;
    CSmartPtr<int> ptr1(new int);
    *ptr1 = 20;
    
    class Test
    {
        public:
            void test() { cout << "call Test::test" << endl; }
    };
    CSmartPtr<Test> ptr2(new Test());
    // (ptr2.operator->()) -> test();
    ptr2->test(); // (*ptr2).test();

    return 0;
}
```

### auto_ptr

包含在memory头文件上，在进行拷贝构造时**永远让最后一个指针管理空间，之前的指针都置nullptr坑爹**

尽量不要在使用容器保存智能指针



### scoped_ptr

直接删除拷贝构造与赋值运算符

```cpp
scoped_ptr(const scoped_ptr<T>&) = delete;

scoped_ptr<T>& operator=(const scoped_ptr<T>&) = delete;
```

推荐使用unique_ptr；



### unique_ptr

```cpp
unique_ptr(const unique_ptr<T>&) = delete;
unique_ptr<T>& operator=(const unique_ptr<T>&) = delete;

/* 提供了右值引用的拷贝构造与赋值运算符重载函数 */
unique_ptr(unique_ptr<T> &&src) ;
unique_ptr<T>& operator=(unique_ptr<T>&& src) ;

// eg:
template<typename T>
unique_ptr<T> getSmartPtr()
{
    unique_ptr<T> ptr(new T());
    return ptr;
}
unique_ptr<int> ptr1 = getSmartPtr<int> ();


```



具有**引用计数功能**的智能指针： shared_ptr 好 weak_ptr

引用计数：

- 多个智能指针可以管理同一个资源
- 给每一个对象资源匹配一个引用计数
- 当一个智能指针引用这个资源的时候，这个资源相应的引用计数就+1
- 当一个智能指针出作用域不在使用这个资源的时候，这个资源相应的引用计数就-1。当这个资源引用计数不为0时，能够析构只能指针，但是不会释放资源。只有引用计数为0时，才能析构并且释放资源



### 模仿具有引用计数功能的智能指针

```cpp
#include <iostream>
#include <memory>
#include <algorithm>
using namespace std;

template <typename T>
class RefCnt
{
public:
    RefCnt(T *ptr = nullptr) : mptr(ptr)
    {
        if (mptr != nullptr)
            mcount = 1;
    }

    void addRef() { mcount++; }  // 增加资源的引用计数

    int delRef() { return --mcount; }
    int getCount() { return mcount; }

private:
    T *mptr;
    int mcount;
};

/* 模仿shared_ptr，但是使用int进行计数，线程不安全，使用atomic_int */
template <typename T>
class CSmartPtr   
{
public:
    CSmartPtr(T *ptr = nullptr)
        : mptr(ptr)
    {
        mpRefCnt = new RefCnt<T>(mptr);
    }

    ~CSmartPtr()
    {
        if (mpRefCnt->delRef() == 0)
        {
            delete mptr;
            mptr = nullptr;
        }
    }

    T &operator*() { return *mptr; }

    T *operator->() { return mptr; }

    CSmartPtr(const CSmartPtr<T> &src)
        : mptr(src.mptr), mpRefCnt(src.mpRefCnt)
    {
        if (mptr != nullptr)
            mpRefCnt->addRef();
    }

    CSmartPtr<T>& operator=(const CSmartPtr<T>& src)
    {
        if (this == &src)  // 防止自赋值 
            return *this;

        // mpRefCnt->delRef();
        if (mpRefCnt->delRef() == 0)
            delete mptr;

        mptr = src.mptr;
        mpRefCnt = src.mpRefCnt;
        mpRefCnt->addRef();
        return *this;
    }

    int getCount() { return mpRefCnt->getCount(); }

private:
    T *mptr;          // 指向资源的指针
    RefCnt<T> *mpRefCnt; // 指向给资源引用计数对象的指针
};

int main(int args, char *argv[])
{
    cout << " --------------" << endl;
    CSmartPtr<int> ptr1(new int);
    CSmartPtr<int> ptr2(ptr1);
    CSmartPtr<int> ptr3;
    ptr3 = ptr2;
    *ptr1 = 20;
    cout << *ptr3 << endl;
    cout << ptr3.getCount() << endl;
    return 0;
}
```



### shared_ptr 与 weak_ptr的问题

**shared_ptr**

- 强智能指针：可以改变资源的引用计数
- 提供了重载的operator*()， 与operator->()能够操作数据



**weak_ptr**

- 弱智能指针：不会改变资源的引用计数
- 没有提供重载的operator*()， 与operator->()不能操作数据



问题1：**强智能指针循环引用(交叉引用)是什么问题，什么结果，怎么解决**

产生问题：导致new出来的资源无法得到释放，资源泄露问题

解决方案：定义对象的时候使用强智能指针，引用对象的时候用弱智能指针



问题2：**类里面使用weak_ptr但是还需要使用被引用对象内的方法**

解决方案：使用lock方法进行提升，提升成功返回被引用对象的指针，否则返回nullptr



```cpp
#include <iostream>
#include <memory>
using namespace std;

/*
 * shared_ptr：强智能指针：可以改变资源的引用计数
 * weak_ptr：弱智能指针：不会改变资源的引用计数
 * weak_ptr 观察 shared_ptr 观察 内存（资源）
 * 弱智能指针只能观察资源无法使用资源
 */

/*
 * 强智能指针循环引用(交叉引用)是什么问题，什么结果，怎么解决
 * 问题：导致new出来的资源无法得到释放，资源泄露问题
 * 定义对象的时候使用强智能指针，引用对象的时候用弱智能指针
 * 弱智能指针没有提供* ->运算符的重载函数，所以不能使用资源
 */

class B;
class A
{
public:
    A() { cout << "a1 A()" << endl; }
    ~A() { cout << "a2 ~A()" << endl; }
    void testA() { cout << "这是一个非常好用的方法" << endl; }
    weak_ptr<B> ptr_b; // 引用其他对象使用弱智能指针
};

class B
{
public:
    B() { cout << "b1 B()" << endl; }
    ~B() { cout << "b2 ~B()" << endl; }
    void func()
    {
        // ptr_a->testA();  // 弱智能指针不能使用方法
        shared_ptr<A> ps = ptr_a.lock();  // 提升方法
        if (ps != nullptr)
        {
            ps->testA();
        }
    }
    weak_ptr<A> ptr_a; // 引用其他对象使用弱智能指针
};

int main(int args, char *argv[])
{
    cout << " --------------" << endl;
    shared_ptr<A> pa(new A()); // 定义或者创建对象的时候使用强智能指针
    shared_ptr<B> pb(new B()); // 定义或者创建对象的时候使用强智能指针

    pa->ptr_b = pb;
    pb->ptr_a = pa;

    cout << pa.use_count() << endl;
    cout << pb.use_count() << endl;
    pb->func();
    return 0;
}
```



### 多线程访问共享对象的问题(线程安全问题)

当我们使用普通指针再多线程进行访问对象时，很有可能出现线程1正在访问一个对象，当线程1结束后销毁对象，那么此时的指针就成为了野指针，其他线程通过这个指针访问一个已经析构的对象就会报错。

```cpp
#include <iostream>
#include <thread>
#include <memory>
using namespace std;

class A
{
public:
    A() { cout << "A()" << endl; }
    ~A() { cout << "~A()" << endl; }
    void testA() { cout << "非常好用的方法" << endl; }
};

/* 子线程 */
void handler01(A *q)
{
    this_thread::sleep_for(chrono::seconds(2));
    
    /* q访问A对象的时候，需要侦测A对象是否还存活 */
    q->testA();
}

int main(int args, char *argv[])
{
    cout << " --------------" << endl;

    shared_ptr<A> p(new A());
    thread t1(handler01, p);
    t1.join();

    return 0;
}
```



因此再多线程中我们可以使用具备引用计数功能的智能指针，这样多个线程通过智能指针访问这个对象时，只有再引用计数为0才会析构这个对象释放空间。是线程安全的。

```cpp
#include <iostream>
#include <thread>
#include <memory>
using namespace std;

class A
{
public:
    A() { cout << "A()" << endl; }
    ~A() { cout << "~A()" << endl; }
    void testA() { cout << "非常好用的方法" << endl; }
};

/* 子线程 */
void handler01(weak_ptr<A> pw)
{
    this_thread::sleep_for(chrono::seconds(2));
    
    /* q访问A对象的时候，需要侦测A对象是否还存活 */
    shared_ptr<A> sp = pw.lock();
    if (sp != nullptr)
        sp->testA();
    else
        cout << "A对象已经析构，不能再访问" << endl;
}

int main(int args, char *argv[])
{
    {
        shared_ptr<A> p(new A());
        thread t1(handler01, p);
        t1.detach();
    }

    this_thread::sleep_for(chrono::seconds(4));

    // t1.join();

    return 0;
}
```



### 自定义删除器

智能指针的删除器

智能指针：能够保证堆区资源绝对释放 delete ptr;

自定义删除器，很多情况下，我们使用智能指针管理文件资源，等等其他资源。

unique_ptr() { 是一个函数对象的调用 deletor(ptr); }，使用函数对象我们就可以使用lambda表达式精简

```cpp
#include <iostream>
#include <memory>
#include <cstdio>
#include <functional>
using namespace std;

/*
 * 智能指针的删除器
 * 智能指针：能够保证资源绝对释放 delete ptr;
 * unique_ptr() { 是一个函数对象的调用 deletor(ptr); }
 * 自定义删除器，很多情况下，我们使用智能指针管理文件资源，等等其他资源
 * 我们就可以自定义删除器释放不同类型的资源
 */

/*
template <typename T>
class Deletor
{
public:
    void operator()(T *ptr)
    {
        delete ptr;
    }
};
 */

template <typename T>
class MyFileDeletor
{
public:
    void operator()(T *ptr) const
    {
        cout << "call MyFileDeletor.operator()" << endl;
        fclose(ptr);
    }
};

template <typename T>
class MyDeletor
{
public:
    void operator()(T *ptr) const
    {
        cout << "call MyDeletor.operator()" << endl;
        delete[] ptr;
    }
};

int main(int args, char *argv[])
{
    cout << " --------------" << endl;

    /* unique两个模板参数，第一个是指针类型，第二个删除器 */
    // unique_ptr<int, MyDeletor<int>> ptr1(new int[100]);
    // unique_ptr<FILE, MyFileDeletor<FILE>> ptr2(fopen("./thread.cpp", "r"));

    unique_ptr<int, function<void(int *)>> ptr1(new int[100],
        [](int *p) -> void{                                               
            cout << "call lambda release new int[100]" << endl;        
            delete[] p;
        });  // 通过lambda表达式释放资源

    unique_ptr<int, function<void(int *)>> ptr2(new int[100],
        [](int *p) -> void{
            cout << "call lambda release new int[100]" << endl;                                                                                
            delete[] p;
        }); // 通过lambda表达式释放资源

    return 0;
}
```





## 4.绑定器

### bind1st && bind2nd

将一个二元函数对象通过绑定一个固定的参数变成一个一元函数对象

bind1st将参数绑定到二元函数对象的第一个参数上，bind2nd将参数绑定到二元函数对象的第二个参数上



**下面是一个向vector中插入一个固定值**

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
#include <functional>
#include <ctime>
using namespace std;


template<typename Container>
void showContainer(Container &con)
{
    typename Container::iterator it = con.begin(); // 不加typename编译器不知道Container是一个类型还是变量
    for (; it != con.end(); ++it)
    {
        cout << *it << " ";
    }
    cout << endl;
}


int main(int args, char* argv[]) 
{
    vector<int> vec;
    srand(time(nullptr));  // 种随机种子
    for (int i = 0; i < 20; ++i)
    {
        vec.push_back(rand() % 100 + 1);
    }

    showContainer(vec);
    sort(vec.begin(), vec.end());  // 默认的迭代器以及排序方式
    showContainer(vec);

    sort(vec.begin(), vec.end(), greater<int>());  // 从大到小排序

    /* 
     * 把70按顺序插入到vec容器中，找到第一个小于70的数字 
     * 需要一个一元函数对象，operator()(const T &val)
     * greater比较器，实质上是一个二元函数对象，提供了重载()运算符 a > b
     * 绑定器 + 二元函数对象  ——————>   一元函数对象
     * less a < b
     * bind1st：+ greater bool operator() (70, const _Ty &_Right) 
     * bind2st: + less bool operator()(const _Ty &_Left, 70)
     */

    // auto it1 = find_if(vec.begin(), vec.end(), bind1st(greater<int>(), 70));
    auto it1 = find_if(vec.begin(), vec.end(), bind2nd(less<int>(), 70));
    if (it1 != vec.end())
    {
        vec.insert(it1, 70);
    }
    showContainer(vec);
    return 0;
}
```



### find_if

查找函数，返回指定位置的迭代器

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
#include <functional>
#include <ctime>
using namespace std;

template <typename Container>
void showContainer(Container &con)
{
    typename Container::iterator it = con.begin(); // 不加typename编译器不知道Container是一个类型还是变量
    for (; it != con.end(); ++it)
    {
        cout << *it << " ";
    }
    cout << endl;
}

/* 遍历这两个迭代器中间的元素，如果满足comp函数对象的运算，就返回当前元素的迭代器 */
template <typename Iterator, typename Compare>
Iterator my_find_if(Iterator first, Iterator last, Compare comp)
{
    for (; first != last; ++first)
    {
        if (comp(*first)) // 调用函数对象的operator()(*first)方法
            return first;
    }
    return last;
}

template <typename Compare, typename T>
class _mybind1st // 绑定器是还是函数对象的应用
{
public:
    _mybind1st(Compare comp, T val)
        : _comp(comp), _val(val) {}

    bool operator()(const T &second)
    {
        return _comp(_val, second); // 底层调用的还是一个二元函数对象，如greater
    }

private:
    Compare _comp;
    T _val;
};

/* mybind1st(greater<int>(), 70)) */
template <typename Compare, typename T>
_mybind1st<Compare, T> mybind1st(Compare comp, const T &val)
{
    /* 直接使用函数模板好处是可以进行类型的推演 */
    return _mybind1st<Compare, T>(comp, val);
}

int main(int args, char *argv[])
{
    vector<int> vec;
    srand(time(nullptr)); // 种随机种子
    for (int i = 0; i < 20; ++i)
    {
        vec.push_back(rand() % 100 + 1);
    }

    showContainer(vec);
    sort(vec.begin(), vec.end()); // 默认的迭代器以及排序方式
    showContainer(vec);

    sort(vec.begin(), vec.end(), greater<int>()); // 从大到小排序

    /*
     * 把70按顺序插入到vec容器中，找到第一个小于70的数字
     * 需要一个一元函数对象，operator()(const T &val)
     * greater比较器，实质上是一个二元函数对象，提供了重载()运算符 a > b
     * 绑定器 + 二元函数对象  ——————>   一元函数对象
     * less a < b
     * bind1st：+ greater bool operator() (70, const _Ty &_Right)
     * bind2st: + less bool operator()(const _Ty &_Left, 70)
     */

    auto it1 = my_find_if(vec.begin(), vec.end(), mybind1st(greater<int>(), 70));
    auto it1 = find_if(vec.begin(), vec.end(), bind2nd(less<int>(), 70));
    if (it1 != vec.end())
    {
     vec.insert(it1, 70);
    }

    showContainer(vec);
    
    return 0;
}


/* lambda表达式同样可以实现 */
/*
    auto it2 = find_if(vec.begin(), vec.end(), [&](int b) -> bool
                       { 
                            for(int i : vec)
                                {
                                if (i < 72)
                                    return true;                        
                                } 
                            return false; 
                       });
    vec.insert(it2, 72);
    showContainer(vec);
*/
```



### function

function是一个类模板，可以把普通函数，成员函数， lambda打包成对象使用

```cpp
#include <iostream>
#include <vector>
#include <functional> // 使用function函数对象的类型
#include <algorithm>
#include <ctime>
#include <string>
#include <map>
using namespace std;
/*
 * C++11提供的绑定器与函数对象
 * bind function
 * C++ STL bind1st与bind2nd 本身还是一个函数对象
 *
 * function ：绑定器，函数对象，lambda表达式他们只能使用在一条语句中
 */

/*
 * 1.用函数类型实例化function
 * 2.通过function调用operator()()方法时，传入对应的参数
*/



void hello1()
{
    cout << "hello world!" << endl;
}

void hello2(string str) // 对应的函数指针 void (*pfunc)(string)
{
    cout << str << endl;
}

int sum(int a, int b)
{
    return a + b;
}

class Test
{
public:
    /* 成员方法的调用必须依赖于对象 对应的函数指针 void (Test::*pfunc)(string) */
    void hello(string str) { cout << str << endl; }
};

void doShowAllBooks() { cout << "查看所有书籍信息" << endl; }
void doRorrow() { cout << "借书" << endl; }
void doBack() { cout << "还书" << endl; }
void doQueryBooks() { cout << "查询书籍" << endl; }
void doLoginOut() { cout << "注销" << endl; }

int main()
{
    int choice = 0;
    map<int, function<void()>> actionMap;
    actionMap.insert(make_pair(1, doShowAllBooks));
    actionMap.insert(make_pair(2, doRorrow));
    actionMap.insert(make_pair(3, doBack));
    actionMap.insert(make_pair(4, doQueryBooks));
    actionMap.insert(make_pair(5, doLoginOut));

    while (true)
    {
        cout << "---------------------" << endl;
        cout << "  1.查看所有书籍信息  " << endl;
        cout << "  2.借书  " << endl;
        cout << "  3.还书  " << endl;
        cout << "  4.查找书籍  " << endl;
        cout << "  5.注销登录  " << endl;
        cout << "---------------------" << endl;
        cout << "请选择:" << endl;
        cin >> choice;

        auto it = actionMap.find(choice);
        if (it == actionMap.end())
            cout << "输入无效，请重新输入" << endl;
        else
            it->second();

#if 0
        /* 开闭原则（Open-Close Principle，简称OCP）是指一个软件实体（类、模块、方法等）应该对扩展开放，对修改关闭。 */
        switch (choice)  // switch不好，因为这块代码无法闭合，无法做到“开闭原则”
        {
        case 1:
            break;
        case 2:
            break;
        case 3:
            break;
        case 4:
            break;
        case 5:
            break;
    
        default:
            break;
        }
        #endif
    }
    return 0;
}

#if 0
int main(int args, char *argv[])
{
    cout << " --------------" << endl;

    /* 从function的类模板定义处，看到希望使用一个函数类型实例化function模板 */
    function<void()> func1(hello1);
    /* func1对象调用其operator()()函数
     * 即 func1.operator()();
     * 也就是在重载()运算符函数内部调用hello1函数
     */
    func1();
    func1.operator()();

    function<void(string)> func2 = hello2;
    func2("你好");
    func2.operator()("句鱼");

    function<int(int, int)> func3 = sum;
    cout << func3(1, 2) << endl;

    function<int(int, int)> func4 = [](int a, int b) -> int
    { return a + b; };
    cout << func4(20, 60) << endl;

    function<void(Test *, string)> func5 = &Test::hello;
    Test t;
    func5(&t ,"call Test::hello!");

    return 0;
}
#endif
```



**function类模板实现原理**

```cpp
#include <iostream>
#include <string>
#include <typeinfo>
#include <functional>

using namespace std;

/*
 * function函数对象实现原理
 * 普通函数与类成员方法
 */

template <typename Fty>
class myfunction
{
};

/* 通过可变参特例化函数对象，使之与任意类型进行匹配 */
template <typename R, typename... A>
class myfunction<R(A...)>
{
public:
    using PFUNC = R (*)(A...);
    myfunction(PFUNC pfunc) : _pfunc(pfunc) {}
    R operator()(A... arg)
    {
        return _pfunc(arg...);
    }

private:
    PFUNC _pfunc;
};

#if 0
template <typename R, typename A1>
class myfunction<R(A1)>
{
public:
    using PFUNC = R (*)(A1);
    myfunction(PFUNC pfunc) : _pfunc(pfunc) {}
    R operator()(A1 arg)
    {
        return _pfunc(arg);
    }

private:
    PFUNC _pfunc;
};

template <typename R, typename A1, typename A2>
class myfunction<R(A1, A2)>
{
public:
    using PFUNC = R (*)(A1, A2);
    myfunction(PFUNC pfunc) : _pfunc(pfunc) {}
    R operator()(A1 arg1, A2 arg2)
    {
        return _pfunc(arg1, arg2);
    }

private:
    PFUNC _pfunc;
};

#endif

void hello(string str) { cout << str << endl; }

int sum(int a, int b) { return a + b; }



int main(int args, char *argv[])
{
    cout << " --------------" << endl;
    myfunction<void(string)> func1 = hello;
    func1("hello world");

    myfunction<int(int, int)> func2(sum);
    cout << func2(10, 20) << endl;

    return 0;
}

```





### 模板特例化与实参类型推演

```cpp
#include <iostream>
#include <cstring>
#include <typeinfo>
using namespace std;

/*
 * 模板的完全特例化和非完全特例化(部分特例化)
 * 模板的实参推演
 */

int sum(int a, int b) { return a + b; }

template <typename R, typename A1, typename A2>
void func2(R (*a)(A1, A2))
{
    cout << typeid(R).name() << endl;
    cout << typeid(A1).name() << endl;
    cout << typeid(A2).name() << endl;
}

template <typename T> // T包含了所有的大的类型，返回值，所有形参的类型都取出来
void func(T a)
{
    cout << typeid(T).name() << endl;
}

class Test
{
public:
    int sum(int a, int b) { return a + b; }
};

template <typename R, typename T, typename A1, typename A2>
void func3(R(T::*a)(A1, A2))
{
    cout << typeid(R).name() << endl;
    cout << typeid(T).name() << endl;
    cout << typeid(A1).name() << endl;
    cout << typeid(A2).name() << endl;
}

int main()
{
    func(10); // 不使用<>，能够自行推演
    func("aaa");
    func(sum);
    func2(sum);
    func3(&Test::sum);
}

#if 0
/* 模板特例化 */
template <typename T>
class Vector
{
public:
    Vector() { cout << "call Vector template init" << endl; }
};

template <>  // 特例化语法
class Vector<char *>
{
public:
    Vector() { cout << "call Vector<char *> template init" << endl; }
};

/* 只针对指针类型提供部分特例化版本 */
template<typename Ty>
class Vector<Ty*>
{
public:
    Vector() { cout << "call Vector<Ty *> template init" << endl; }
};

/* 针对函数指针提供的部分特例化（） */
template<typename R, typename A1, typename A2>
class Vector<R(*)(A1, A2)>
{
    public:
        Vector() { cout << "call Vector<R(*)(A1, A2)> template init" << endl; }
};

/* 针对函数类型提供的部分特例化 */
template <typename R, typename A1, typename A2>
class Vector<R(A1, A2)>
{
    public:
        Vector() { cout << "call Vector<R(A1, A2)> template init" << endl; }
};

int sum(int a, int b) { return a + b; }

int main()
{
    Vector<int> vec1;
    Vector<char *> vec2;
    Vector<int *> vec3;
    Vector<int (*)(int, int)> vec4;
    Vector<int(int, int)> vec5;

    /* 注意区分函数类型与函数指针类型 */
    typedef int (*PFUNC1)(int, int);
    PFUNC1 pfunc1 = sum;
    pfunc1(10, 20);

    typedef int PFUNC2(int, int);
    PFUNC2 *pfunc2 = sum;
    cout << (*pfunc2)(10, 20) << endl;
}
#endif;

#if 0
template<typename T>
bool compare(T a, T b)
{
    cout << "template compare" << endl;
    return a > b;
}

/* 完全特例化 */
template <>
bool compare<const char *>(const char *a, const char *b)
{
    cout << "compare<const char *>(const char *a, const char *b)" << endl;
    return strcmp(a, b);
}

    int main(int args, char *argv[])
{
    cout <<" --------------" <<endl;
    compare(10, 20);  // 经过参数推演为int，实例化一个int为参数的compare函数
    compare("aaa", "bbb");  // 推演参数类型为const char *
    return 0;
}
#endif
```



### bind函数

**C++绑定器 bind绑定器的返回结果仍是一个函数对象**

```cpp
#include <iostream>
#include <functional>
#include <string>

using namespace std;


/*
 * C++绑定器 bind绑定器的返回结果仍是一个函数对象
 * function是一个类模板，bind是一个函数模板，可以自动推演模板类型参数，就是用于给函数绑定参数
 */

void hello(string str) { cout << str << endl; }
int sum(int a, int b) { return a + b; }

class Test
{
public:
    int sum(int a, int b) { return a + b; }
};

int main(int args, char *argv[])
{
    cout << " --------------" << endl;

    /* 返回函数对象，调用()重载运算符函数 */
    bind(hello, "hello bind!")(); // bind(hello, "hello bind!").operator()();
    auto f1 = bind(sum, 10, 20); 
    cout << f1() << endl;

    cout << bind(&Test::sum, Test(), 20, 30)() << endl;

    /* 参数占位符，绑定器除了语句无法继续使用，通过function保存结果 */
    bind(hello, placeholders::_1)("hello bind2");

    /* 此处把绑定器返回的函数对象就复用起来了 */
    function<void(string)> func1 = bind(hello, placeholders::_1);
    func1("你好，句鱼");

    return 0;
}
```



### lambda表达式

语法：\[捕获外部变量](形参列表) -> 返回值类型 {操作代码};

捕获列表

- [] ：表示不捕获任何外部变量，此种捕获方式编译器生成的默认函数是const函数，即常函数
- [=]： 进行值捕获
- [&]：引用捕获
- [this]：捕获外部的this指针，这样才能调用类成员或类成员方法
- [=, &a]：以值捕获方式捕获其他所有外部变量，但是变量a的捕获方式为引用捕获
- [a, b]：以值捕获方式捕获外部变量a，b
- [a, &b]：以值捕获方式捕获a，引用捕获b



lambda表达式原理：编译器生成一个为命名的相应模板类对象，并且在这个类中提供了operator()()；函数

```cpp
#include <iostream>

using namespace std;


/* func1函数对象中的lambda实现原理 */
template<typename T>
class TestLambda
{
    public:
    void operator()() const
    {
        cout << "hello world!" << endl;
    }
};

int main(int args, char* argv[]) 
{
    cout <<" --------------" <<endl;

    auto func1 = []() -> void
        { 
            cout << "hello world" << endl; 
        };
    func1();

    auto func2 = [](int a, int b) -> int
    { return a + b; };
    cout << func2(10, 20) << endl;

    TestLambda<void> t1;
    t1();

    int a = 10, b = 20;
    auto func3 = [&a, &b]()
    {
        int temp = a;
        a = b;
        b = temp;
    };
    func3();
    return 0;
} 
```



```cpp

#include <iostream>
#include <algorithm>
#include <functional>
#include <vector>
#include <map>
#include <memory>
#include <queue>

using namespace std;

/*
 * 既然lambda只能使用在语句中，
 * 如果想跨语句使用之前定义好的lambda表达式
 * 使用function类型来表示函数对象类型
 */

class Data
{
public:
    Data(int val1 = 10, int val2 = 20) : ma(val1), mb(val2) {}
    /* 我们不仅比较ma还需要比较mb怎么实现? */
    bool operator<(const Data &data) const { return ma < data.ma; }
    bool operator>(const Data &data) const { return ma > data.ma; }
    int ma;
    int mb;
};

int main(int args, char *argv[])
{
    /* 智能指针自定义删除器 */
    unique_ptr<FILE, function<void(FILE *)>> ptr1(fopen("C:/Users/18174/Desktop/password.txt", "r"),
                                                  [](FILE *fptr)
                                                  { fclose(fptr); });

    /* 优先级队列:把自定义类型放进优先级队列需要重载 ><运算符 */
    priority_queue<Data> queue;
    queue.push(Data(10, 20));
    queue.push(Data(15, 15));
    queue.push(Data(20, 20));

    using FUNC = function<bool(Data&, Data&)>;
    priority_queue<Data, vector<Data>, FUNC> maxHeap([](Data &d1, Data &d2) -> bool
                                                       { return d1.ma > d2.ma; });
    maxHeap.push(Data(10, 20));
    maxHeap.push(Data(15, 15));
    maxHeap.push(Data(20, 20));
    return 0;
}

#if 0
int main(int args, char* argv[])
{
    cout << " --------------" << endl;
    map<int, function<int(int, int)>> caculateMap;
    caculateMap[1] = [](int a, int b) -> int
    { return a + b; };

    caculateMap[2] = [](int a, int b) -> int
    { return a - b; };

    caculateMap[3] = [](int a, int b) -> int
    { return a * b; };

    caculateMap[4] = [](int a, int b) -> int
    { return a / b; };


    cout << "选择：";
    int ch;
    cin >> ch;

    cout << "10 + 15 = " << caculateMap[ch](10, 15) << endl;
    return 0;
}
#endif
```



## 5.C++11知识点总结

### 关键字与语法

1. auto: 可以根据右值推导出右值的类型。然后等号左面变量的类型就已知了
2. nullptr: 给指针专用(能够和整数进行区别)，NULL是宏定义，其值为0
3. foreach: 可以遍历数组与容器，底层就是通过指针或者迭代器来实现的(增强for循环)
4. 右值引用: move移动语义函数，forward类型完美转发
5. 模板的新特性: 模板可变参，typename A... 表示可变参



### 绑定器与函数对象

1. function: 函数对象
2. bind: 绑定器
3. lambda表达式



### 智能指针

1. shared_ptr与weak_ptr
2. unique_ptr



### STL容器

1. set和map： 红黑树O(log2n)
2. unordered_set, unordered_map：哈希表O(1) (无序容器)
3. array: 数组，轻量级数组(大小固定)
4. forward_list: 前向列表，轻量级list



### thread

```cpp
#include <iostream>
#include <thread>
using std::cout;
using std::endl;

/*
 * C++语言级别的多线程编程  -->跨平台
 * thread/mutex/condition_variable
 * lock_quard/unique_lock
 * atomic 基于CAS操作的原子类型
 * sleep_for
 * thread类在windows下是createThread，linux下是pthread_create，linux使用strace命令查看跟踪打印
 */

/*
 * 怎么创建并启动一个线程:创建一个线程对象，传入线程所需要的函数与参数，线程就会自动执行
 * 线程分离：主线程运行完成，查看如果当前进程还有为运行完的子线程，进程就会异常终止，设置线程分离能避免这个错误
 * 子线程结束：子线程函数运行完成，线程就结束了
 * 主线程处理子线程：join等待子线程结束， detach设置线程分离
 */


void threadHandel(int time)
{
    std::this_thread::sleep_for(std::chrono::seconds(time));
    cout << "thread hello thread1" << endl;
}

int main(int args, char *argv[])
{
    cout << " --------------" << endl;
    /* 定义了一个线程对象，传入一个线程函数（线程的入口函数） */
    std::thread t1(threadHandel, 2);

    /* 主线程等待子线程结束，主线程继续向下执行 */
    // t1.join(); 

    /* 把子线程设置为分离线程 */
    t1.detach();
    cout << "thread1 ok！！！" << endl;
    return 0;
}
```



**模拟买票程序**

```cpp
#include <iostream>
#include <thread>
#include <list>
#include <mutex>
#include <atomic>
// #include <>
using std::cout;
using std::endl;
using std::list;
using std::thread;

/*
 * 多线程程序的线程安全
 * 竟态条件：多线程程序执行的结果是一致的，不会随着CPU对线程不同的调用顺序，而产生不同的运行结果
 *
 */

/* 模拟车站三个窗口买票的程序 lock_guard unique_lock */

int ticketCount = 100; // 100张车票，三个窗口一起卖
std::mutex mtx;

/* 模拟买票的线程函数 */
void sellTicket(int index)
{

    while (ticketCount > 0)  // 锁 + 双重判断
    {
        /* 对临界区代码段 保证原子操作 进行线程间互斥 使用互斥锁实现 */
        mtx.lock();
        if (ticketCount > 0)
        {
            cout << "窗口：" << index << "卖出第" << ticketCount << "张票" << endl;
            ticketCount--;
        }
        mtx.unlock();
        std::this_thread::sleep_for(std::chrono::milliseconds(100)); // 卖一张票睡眠100毫秒
    }

    /*
     * 输出-1的问题，当只剩一张票的时候，线程1获得锁，
     * 但是线程2也进入while循环没有抢到锁，
     * 导致剩余1张票的时候卖了两次
     * */
}

int main(int args, char *argv[])
{
    
    list<thread> tlist;
    for (int i = 1; i <= 3; ++i)
    {
        tlist.push_back(thread(sellTicket, i));
    }

    for (thread &t : tlist)
    { 
        t.join();
    }

    cout << "所有窗口买票结束" << endl;

    return 0;
}
```



当我们使用互斥锁mutex时，我们需要手动加锁与解锁，当加锁成功后，有可能因为一些其他因素导致无法释放锁，因此使用**lock_guard**

**lock_guard:**  超出作用域能够自动释放锁，并且禁用拷贝构造与赋值重载运算符 类似于智能指针scoped_ptr

优化

```cpp
#include <iostream>
#include <thread>
#include <list>
#include <mutex>
#include <atomic>
// #include <>
using std::cout;
using std::endl;
using std::list;
using std::thread;

/*
 * 多线程程序的线程安全
 * 竟态条件：多线程程序执行的结果是一致的，不会随着CPU对线程不同的调用顺序，而产生不同的运行结果
 *
 */

/* 模拟车站三个窗口买票的程序 lock_guard unique_lock */

int ticketCount = 100; // 100张车票，三个窗口一起卖
std::mutex mtx;

/* 模拟买票的线程函数 */
void sellTicket(int index)
{

    while (ticketCount > 0)  // 锁 + 双重判断
    {
        /* 对临界区代码段 保证原子操作 进行线程间互斥 使用互斥锁实现 */
        {
            /* 只在作用域内生效，超出作用域析构自动释放锁，类似scoped_ptr无拷贝构造与赋值重载函数 */
            std::lock_guard<std::mutex> lock(mtx);

            // std::unique_lock<std::mutex> lock(mtx);
            // lock.lock();
            if (ticketCount > 0)
            {
                cout << "窗口：" << index << "卖出第" << ticketCount << "张票" << endl;
                ticketCount--;
            }
            // lock.unlock();
        }    
        std::this_thread::sleep_for(std::chrono::milliseconds(100)); // 卖一张票睡眠100毫秒
    }

    /*
     * 输出-1的问题，当只剩一张票的时候，线程1获得锁，
     * 但是线程2也进入while循环没有抢到锁，
     * 导致剩余1张票的时候卖了两次
     * */
}

int main(int args, char *argv[])
{
    
    list<thread> tlist;
    for (int i = 1; i <= 3; ++i)
    {
        tlist.push_back(thread(sellTicket, i));
    }

    for (thread &t : tlist)
    {
        t.join();
    }

    cout << "所有窗口买票结束" << endl;

    return 0;
}
```



### 生产者消费者模型的简单实现

```cpp
#include <iostream>
#include <thread>
#include <condition_variable>
#include <queue>

using namespace std;

/*
 * 线程同步通信机制
 * 线程互斥：代码存在竟态条件
 * 线程通信：一个线程的执行需要另一个线程执行某部分操作并进行通知
 * 生产者生产一个物品，通知消费者消费一个物品，消费完了再通知生产者生产
 */

std::mutex mtx;        // 定义互斥锁，用于线程互斥
condition_variable cv; // 定义条件变量，用于线程通信
class Queue
{
public:
    void put(int val) // 生产物品
    {
        unique_lock<std::mutex> lck(mtx);
        while (!que.empty())
        {
            /* 队列不为空，生产者应该通知消费者去消费，消费完毕再进行生产并进入阻塞状态，释放锁 */
            cv.wait(lck);  // 进入等待状态，释放锁 
        }
        que.push(val);
        cv.notify_all();  // 唤醒其他因未抢到锁而阻塞的线程
        cout << "生产者 生产：" << val << "号物品" << endl;
    }

    int get() // 消费物品
    {
        unique_lock<std::mutex> lck(mtx);
        while (que.empty())
        {
            
            cv.wait(lck); // 进入等待状态，释放锁
        }

        int val = que.front();
        que.pop();
        cv.notify_all();
        cout << "消费者 消费：" << val << "号物品" << endl;
        return val;
    }

private:
    queue<int> que;
};

void producer(Queue *que) // 生产者线程
{
    for (int i = 1; i <= 10; ++i)
    {
        // lock_guard<std::mutex> guard(mtx);
        que->put(i);
        std::this_thread::sleep_for(std::chrono::milliseconds(100));
    }
}

void consumer(Queue *que) // 消费者线程
{
    for (int i = 1; i <= 10; ++i)
    {
        // lock_guard<std::mutex> guard(mtx);
        que->get();
        std::this_thread::sleep_for(std::chrono::milliseconds(100));
    }
}

int main(int args, char *argv[])
{
    Queue que; // 两个线程共享的队列
    cout << " --------------" << endl;
    thread t1(producer, &que);
    thread t2(consumer, &que);
    t1.join();
    t2.join();
    return 0;
}
```



### atomic

基于CAS的原子类型， CAS无锁操作，简单理解为再硬件层面上对总线进行加锁(CPU于内存之间的通信通过总线)。

```cpp
#include <iostream>
#include <thread>
#include <atomic>
#include <queue>
#include <list>

using namespace std;

volatile atomic_bool isReady = false;  // volatile关键字修饰共享变量，作用是防止多线程访问共享变量进行缓存，实现实时读取
volatile atomic_int mycount = 0;

void task()
{
    while (!isReady)
    {
        this_thread::yield();  // 线程出让当前CPU时间片，等待下一次调度
    }

    for (int i = 0; i < 100; ++i)
    {
        mycount++;
    }
}

int main(int args, char* argv[])
{
    cout << " --------------" << endl;
    list<thread> tlist;
    for (int i = 0; i < 10; ++i)
    {
        tlist.push_back(thread(task));
    }

    this_thread::sleep_for(chrono::seconds(3));
    isReady = true;

    for (thread& t : tlist)
    {
        t.join();
    }

    cout << mycount << endl;
    return 0;
}
```



## 6.设计模式

### 单例模式

单例模式：一个类不管创建多少次对象，永远只能得到该类型的一个对象的实例

应用:日志模块，数据库模块

控制类对象的个数 -> 控制构造函数

**具体代码实现方式**

1. 构造函数私有化
2. 私有化一个唯一类的派生对象(通过static实现)
3. 公有化获取类的唯一实例对象的接口方法



1. **饿汉单例模式**

   - 还没有获取实例对象，实例对象就已经产生了，线程安全

   - ```cpp
     #include <iostream>
     
     using namespace std;
     
     
     
     class Singleton
     {
     public:
         /* 3.获取类的唯一实例对象的接口方法 */
         static Singleton *getInstance()
         {
             return &instance;
         }
     
     private:
         /* 2.定义一个唯一的类的实例对象 */
         static Singleton instance;
         /* 1.构造函数私有化 */
         Singleton()
         {
         }
     
         Singleton(const Singleton &) = delete;
         Singleton &operator=(const Singleton &) = delete;
     };
     
     Singleton Singleton::instance;
     
     int main(int args, char *argv[])
     {
         cout << " --------------" << endl;
         Singleton *p1 = Singleton::getInstance();
         Singleton *p2 = Singleton::getInstance();
         Singleton *p3 = Singleton::getInstance();
         // Singleton t1 = *p1; // 尝试访问已经删除的函数，报错
         return 0;
     }
     
     ```

     

2. 懒汉单例模式

   - 唯一的实例对象，直到第一次获取它的时候才产生。更实用

   - 线程安全的懒汉单例模式

   - ```cpp
     #include <iostream>
     #include <mutex>
     using namespace std;
     
     std::mutex mtx;
     
     class Singleton
     {
     public:
         /* 3.获取类的唯一实例对象的接口方法 */
         static Singleton *getInstance()
         {
     
             if (instance == nullptr)
             {
                 /* 当锁定义在if内中，可以减轻锁的粒度，但是需要锁+双重判断保证线程互斥、线程安全 */
                 lock_guard<std::mutex> guard(mtx);
                 if (instance == nullptr)
                 {
                     /* 开辟内存，构造对象，给instance赋值 */
                     instance = new Singleton();
                 }
             }
             return instance;
         }
     
     private:
         /* 2.定义一个唯一的类的实例对象 */
         static Singleton *volatile instance;
         /* 1.构造函数私有化 */
         Singleton()
         {
         }
     
         Singleton(const Singleton &) = delete;
         Singleton &operator=(const Singleton &) = delete;
     };
     
     Singleton *volatile Singleton::instance = nullptr;
     
     int main(int args, char *argv[])
     {
         cout << " --------------" << endl;
         Singleton *p1 = Singleton::getInstance();
         Singleton *p2 = Singleton::getInstance();
         Singleton *p3 = Singleton::getInstance();
         // Singleton t1 = *p1; // 尝试访问已经删除的函数，报错
         return 0;
     }
     
     ```

     ​	

   - 不使用锁实现的线程安全的懒汉单例模式

   - ```cpp
     #include <iostream>
     #include <mutex>
     using namespace std;
     
     class Singleton
     {
     public:
         static Singleton *getInstance()
         {
             /* 通过静态局部变量特性实现的懒汉实例，线程安全 */
             /* 函数静态局部变量的初始化，在汇编指令上就已经自动添加线程互斥指令了 */
             static Singleton instance;
             return &instance;
         }
     
     private:
     
         Singleton()
         {
         }
     
         Singleton(const Singleton &) = delete;
         Singleton &operator=(const Singleton &) = delete;
     };
     
     
     int main(int args, char* argv[]) 
     {
         cout <<" --------------" <<endl;
         
         return 0;
     }
     
     ```

### 工厂模式

[工厂模式大秦坑王](https://blog.csdn.net/QIANGWEIYUAN/article/details/88792594?spm=1001.2014.3001.5502)

 简单工厂 Simple Factory（非标准oop的设计方法）

 工厂方法 Factory Method（标准oop的设计方法之一）

 抽象工厂 AbStract Factory（标准oop的设计方法之一）



**简单工厂**

```cpp
#include <iostream>
#include <string>
#include <memory>
using namespace std;

/*
 * 简单工厂 Simple Factory
 * 工厂方法 Factory Method
 * 抽象工厂 AbStract Factory

 * 工厂模式：主要是封装了对象的创建

 */

class Car
{
public:
    Car(string name) : _name(name) {}
    virtual void show() = 0;

protected:
    string _name;
};

class BMW : public Car
{
public:
    BMW(string name) : Car(name) {}

protected:
    void show()
    {
        cout << "获取了一辆宝马汽车" << _name << endl;
    }
};

class Audi : public Car
{
public:
    Audi(string name) : Car(name) {}

protected:
    void show()
    {
        cout << "获取了一辆奥迪汽车" << _name << endl;
    }
};

enum class CarType
{
    BMW,
    AUDI
};

class SimpleFactory
{
public:
    Car *createCar(CarType ct)
    {
        switch (ct)
        {
        case CarType::BMW:
            return new BMW("X1");

        case CarType::AUDI:
            return new Audi("A6");
        default:
            cerr << "传入工厂的参数不正确" << endl;
            break;
        }
        return nullptr;
    }
};

int main(int args, char *argv[])
{
    cout << " --------------" << endl;


    unique_ptr<SimpleFactory> factory(new SimpleFactory());
    unique_ptr<Car> up1 (factory->createCar(CarType::BMW));
    unique_ptr<Car> up2 (factory->createCar(CarType::AUDI));
    up1->show();
    up2->show();
    return 0;
}
```



**工厂方法**

```cpp
#include <iostream>
#include <string>
#include <memory>
using namespace std;

/*
 * 简单工厂 Simple Factory
 * 工厂方法 Factory Method
 * 抽象工厂 AbStract Factory

 * 工厂模式：主要是封装了对象的创建

 */

class Car
{
public:
    Car(string name) : _name(name) {}
    virtual void show() = 0;

protected:
    string _name;
};

class BMW : public Car
{
public:
    BMW(string name) : Car(name) {}

protected:
    void show()
    {
        cout << "获取了一辆宝马汽车" << _name << endl;
    }
};

class Audi : public Car
{
public:
    Audi(string name) : Car(name) {}

protected:
    void show()
    {
        cout << "获取了一辆奥迪汽车" << _name << endl;
    }
};

/* 工厂方法 */
class Factory
{
public:
    virtual Car *createCar(string name) = 0;
};

/* 宝马工厂 */
class BMWFactory : public Factory
{
public:
    Car* createCar(string name)
    {
        return new BMW(name);
    }
};

/* 奥迪工厂 */
class AUDIFactory : public Factory
{
public:
    Car *createCar(string name)
    {
        return new Audi(name);
    }
};

int main(int args, char* argv[]) 
{
    cout <<" --------------" <<endl;
    unique_ptr<Factory> bmwfty(new BMWFactory());
    unique_ptr<Factory> audifty(new AUDIFactory());
    unique_ptr<Car> bmw(bmwfty->createCar("X1"));
    unique_ptr<Car> audi(audifty->createCar("A6"));

    return 0;
}

```



**抽象工厂**：对一组有关联关系的产品簇提供产品对象的统一创建，客户不需要自己new对象，不需要了解对象创建的详细过程



















## 7.面试题目

### static 

1. 回答方式：说清楚修饰变量于函数，说一下static在编译链接的过程，说一下static对于符号的影响，是否线程安全


- 面向过程

  - static可以修饰全局变量和函数，static修饰全局变量和函数其他文件不可使用(在符号表中，被static修饰后，符号的作用域就从g(glonal）变成b(local) )
  - static修饰局部变量会产生符号，数据就放到.data(已经初始化)或者.bss(为初始化),普通局部变量不产生符号而是通过栈帧ebp-偏移量进行访问
  - static在变量在初始化时是线程安全的，也只有初始化的时候才是线程安全的

- 面向对象

  - static修饰成员变量以及成员方法(修饰成员方法不在产生this指针)

  

### this用处

- 一个类型可以实例化很多对象，每个对象都有私有成员变量，共享一套成员方法。每一套成员方法都含有一个默认参数，这个参数就是this指针，当对象调用一个方法时，编译器就会默认传入一个当前对象的地址初始化this指针，用于区分不同对象

  ```cpp
  class Test
  {
  public:
      void test();
  };
  int main()
  {
      Test t;
      t.test();  // 编译器编译:t.tes
      t(&t);
  }
  ```

- 使用lambda表达式可以捕获this

- 静态成员方法不能访问this



### C++的new和delete，什么时候用new[]方式申请，使用delete释放

- 本质上是运算符重载operator new() 和 operator delete()
- 如果是自定义类型而且提供了析构函数，那么用new[]申请就一定需要匹配delete[]，其他情况下可以new[]申请delete释放



### C++继承

- 继承属于类与类之间的关系

- 实现代码复用

- 通过继承，在基类里面给所有派生类保留统一的纯虚函数接口，等待派生类进行重写，通过使用多态可以通过基类的指针或者引用访问不同派生类对象的同名覆盖方法。

- 虚继承实现方式：

  ```cpp
  #include <iostream>
  
  using namespace std;
  
  /*
      1. 在菱形继承中存在二义性问题，使用作用域能够在一定程度上进行区分
      2. 可以使用虚继承进行解决
      3. 虚继承实现原理：
          # 普通继承会将基类中的内容拷贝一份在子类中，
            如果派生过多，就会导致浪费存储空间和二义性问题（菱形继承）
          # 虚继承底层实现原理与编译器相关，通过虚基类指针和虚基类表实现，
            每个虚继承的子类都有一个虚基类指针（占用一个指针的存储空间，4字节）
            和虚基类表（不占用类对象的存储空间）
            （需要强调的是，虚基类依旧会在子类里面存在拷贝，
            只是仅仅最多存在一份而已，并不是不在子类里面了）；
            当虚继承的子类被当做父类继承时，虚基类指针也会被继承。
          # 当派生类想要访问父类中的成员就能通过指针偏移实现（两个问题，多态实现原理类似，为什么不将这两个指针合并）
          # 在不同编译条件下C++实现多态存在区别，msvc上虚函数表与虚基类表是不同的，mingw是相同的
          # msvc: 当虚继承的基类含有虚函数时，派生类会继承两个指针：虚函数表指针（指向虚函数表），虚基类指针（指向虚基类表）
          # mingw: 无论虚继承的基类是否含有虚函数，派生类只会继承一个指针，虚基类指针，因此实现多态只靠虚基类指针偏移
  */
  
  class Father
  {
  public:
      int num;
      virtual void func(){};
  };
  
  class Son1 : public Father
  {
  public:
      int son1_id;
      int all = 10;
  };
  
  class Son2 : public Father
  {
  public:
      int son2_id;
      int all = 20;
  };
  
  class Test : public Son1, public Son2
  {
  public:
      // num = 10;  报错，二义性，不知道num是Son1的还是Son2的
      // 使用作用域能够解除二义性
      int self_id;
      Test()
      {
          this->self_id = Son1::num;
          // cout << this->all << endl;
      }
  };
  
  class A
  {
  public:
      int father_id = 10;
  };
  
  // 虚继承基类
  class a1 : virtual public A
  {
  public:
      int a1_id;
      a1()
      {
          this->father_id = 20;
      }
  };
  
  class a2 : virtual public A
  {
  public:
      int a2_id;
      a2()
      {
          this->father_id = 30;
      }
  };
  
  class b : public a1, public a2
  {
  public:
      b() : a1(), a2()
      {
          // 不会报错
          cout << this->father_id << endl;
      }
  };
  
  int main(int args, char *argv[])
  {
      cout << " --------------" << endl;
      b ob;
      return 0;
  }
  ```

  

### C++继承多态，空间配置器，vector与list的区别，map与多重map

- 多态：静态多态(编译时期)、动态多态(运行时多态)
  - 静态多态：函数重载与模板
  - 动态多态：虚函数
- 空间配置器：给容器使用，主要作用就是把对象的内存开辟和对象的构造分开，把对象内存释放与对象析构分开
  - [空间配置器](https://blog.csdn.net/weixin_39640298/article/details/88766223)

- vector和list的区别：数组和链表的区别

- map是映射表不允许key重复，multi_map是多重映射表允许key重复，底层实现的**红黑树**
  - 红黑树的五个性质，插入的三种方式(最多旋转两次)，删除(最多旋转三次)的四种情况



### C++防止内存泄漏?智能指针

- 内存泄漏: 分配的堆内存没有释放，也再没有机会释放
- 智能指针: auto_ptr, scoped_ptr, unique_ptr, shared_ptr， weak_ptr。





## 8.海量数据求top k问题

**大小堆**

```cpp
#include <iostream>
#include <vector>
#include <queue>
#include <functional>
using namespace std;

/*
 * 海量数据求topk问题
 * 求最大/最小的前k个元素
 * 求最大/最小的第k个元素

 * 解法1：大小堆 -> priority_queue
 * 100000个整数，找到前10大的元素：先用前10个整数创建一个小根堆(最小值在堆顶)，
 * 遍历剩下的整数，如果整数比堆顶元素大，就删除堆顶元素，
 * 然后再把整数入堆，遍历完所有整数，小根堆里面的元素即为解
 * 如果找第k小/大只需访问堆顶元素就行
 * 时间复杂度：O(n)

 * 大根堆 -> topk 小
 * 小根堆 -> topk 大


 * 解法2：快排分割函数
 * 经过快排分割函数，能够在O(log2n)的时间内，把小于基准数的整数调整到左边
 * 把大于基准数的整数调整到右边，基准数(index)就可以认为是第(index + 1)小的整数了
 */

int main(int args, char *argv[])
{
    cout << " --------------" << endl;
    /* 求vector容器中元素最大的前10个数字 */
    vector<int> vec;
    for (int i = 0; i < 10000; ++i)
    {
        vec.push_back(rand() + i);
    }

    /* 定义小根堆 */
    priority_queue<int, vector<int>, greater<int>> minHeap;
    int k = 0;
    for (; k < 10; ++k)
    {
        minHeap.push(vec[k]);
    }

    /* 遍历vector */
    for (int &i : vec)
    {
        if (i > minHeap.top())
        {
            minHeap.pop();
            minHeap.push(i); 
        }
    }

    /* 打印结果 */
    while(!minHeap.empty())
    {
        cout << minHeap.top() << " ";
        minHeap.pop();
    }
    cout << endl;

    return 0;
}
```



**快排分割函数**

```cpp
#include <iostream>
#include <vector>
#include <queue>
using namespace std;

/* 数组分割函数，i,j是范围 */
int partation(vector<int> &arr, int i, int j)
{
    int k = arr[i];
    while (i < j)
    {
        /* 从右向左找比基准数小的 */
        while (i < j && arr[j] >= k)
            --j;
        if (i < j)
            arr[i++] = arr[j];

        /* 从左向右找比基准数大的 */
        while (i < j && arr[i] < k)
            ++i;
        if (i < j)
            arr[j--] = arr[i];
    }
    arr[i] = k;
    return i;
}

int selectNoK(vector<int> &arr, int i, int j, int k)
{
    int pos = partation(arr, i, j);
    if (pos == k - 1)
        return pos;  // 返回位置
    else if (pos < k - 1)
        return selectNoK(arr, pos + 1, j, k);
    else
        return selectNoK(arr, i, pos - 1, k);
}

int main(int argc, char const *argv[])
{
    /* 求vector中元素第10小的元素  前十小的 */
    vector<int> vec;
    for (int i = 0; i < 100000; ++i)
        vec.push_back(rand() % 1000 + i);

    cout << vec[selectNoK(vec, 0, vec.size() - 1, 10)] << endl;

    return 0;
}

```



**海量数据求查重和top k问题的综合应用**

```cpp
#include <iostream>
#include <unordered_map>
#include <vector>
#include <functional>
#include <queue>
using namespace std;

/*
 * 海量数据求查重和top k问题的综合应用‘
 * 查重：数据是否有重复，以及数据重复的次数
 
 * 数据重复次数最大/最小的前k个/第k个

 * 解法：哈希统计(map) + 堆/快排分割

 */



int main(int args, char* argv[]) 
{
    cout <<" --------------" <<endl;
    vector <int> vec;
    for (int i < 40 < ++i)
        vec.push_back(rand() % 10);
    
    unordered_map<int, int> numMap;
    for (int val : vec)
        numMap[val]++;
    
    using P = pair<int, int>;
    using FUNC = function<bool(P&, P&)>;
    using MinHeap = priority_queue<p, vector<P>, FUNC>;
    MinHeap minheap([](auto &a, auto &b) -> bool
        {
            return a.second > b.second;
        });

    int k = 0;
    auto it = numMap.begin();

    for (; it != numMap.end() && k < 3; ++it, ++k)
        minheap.push(*it);

    for (; it != numMap.end(); ++it)
    {
        if (it->second > minheap.top().second)
        {
            minheap.pop();
            minheap.push(*it);
        }
    }

    while (!minheap.empty())
    {
        auto &pair = minheap.top();
        cout << pair.first << " : " << pair.second << " ";
        minheap.pop();
    }

    return 0;
}
```



**在大文件上进行海量数据求取重复次数最多的top k问题**

```cpp
#include <iostream>
#include <unordered_map>
#include <vector>
#include <functional>
#include <queue>
using namespace std;

/*
 * 海量数据求查重和top k问题的综合应用‘
 * 查重：数据是否有重复，以及数据重复的次数

 * 问题：数据重复次数最大/最小的前k个/第k个
 * 解法：哈希统计(map) + 堆/快排分割


 * 问题：有一个大文件，内存限制200M,求文件中重复次数最多的前十个
 * 把大文件分割成小文件，把大文件的数据通过哈希映射把数据离散的放入小文件中，计算每个小文件的top k

 */

int main(int args, char* argv[])
{

    FILE* fptr = fopen("data.dat", "wb");
    for (int i = 0; i < 20000; ++i)
    {
        int data = rand();
        fwrite(&data, 4, 1, fptr);
    }
    fclose(fptr);

    cout << " --------------" << endl;
    /* 打开存储数据的原始文件 */
    FILE* pf = fopen("data.dat", "rb");
    if (!pf)
        return 0;

    /* 由于原始数据量缩小，所以这里文件划分个数也变小，11个小文件 */
    const int FILE_NO = 11;
    FILE* pfile[FILE_NO] = { nullptr };
    for (int i = 0; i < FILE_NO; ++i)
    {
        char filename[20];
        sprintf(filename, "data%d.dat", i + 1);
        pfile[i] = fopen(filename, "wb+");
    }

    /* 哈希映射，把大文件中的数据，映射到各个小文件中 */
    int data;  // 大文件中的整数
    while (fread(&data, 4, 1, pf) > 0)
    {
        int findex = data % FILE_NO;
        fwrite(&data, 4, 1, pfile[findex]);
    }
    unordered_map<int, int>numMap;  // 定义一个链式哈希表
    using P = pair<int, int>;
    using FUNC = function<bool(P&, P&)>;
    using MinHeap = priority_queue<P, vector<P>, FUNC>;
    MinHeap minheap([](auto& a, auto& b) -> bool
        {
            return a.second > b.second;
        });

    for (int i = 0; i < FILE_NO; ++i)
    {
        /* 恢复小文件的文件指针到起始位置 */
        fseek(pfile[i], 0, SEEK_SET);

        /* 这里直接统计数字的重复次数 */
        while (fread(&data, 4, 1, pfile[i]) > 0)
            numMap[data]++;
        int k = 0;
        auto it = numMap.begin();

        /* 如果堆是空的，先往堆放10个数据 */
        if (minheap.empty())
        {
            /* 先从map表中读取10个数据放到小根堆中，建立top 10的小根堆，最小的元素在堆顶 */
            for (; it != numMap.end() && k < 10; ++it, ++k)
            {
                minheap.push(*it);
            }
        }

        /* 把K+1到末尾的元素进行遍历，和堆顶元素进行比较 */
        for (; it != numMap.end(); ++it)
        {
            /* 如果map表中当前元素重复次数大于堆顶元素的重复次数，则进行替换 */
            if (it->second > minheap.top().second)
            {
                minheap.pop();
                minheap.push(*it);
            }
        }
        numMap.clear();
    }

    while (!minheap.empty())
    {
        auto& pair = minheap.top();
        cout << pair.first << " : " << pair.second << endl;
        minheap.pop();
    }

    return 0;
}

```

