---
layout:     post

title:      CPP--Linux--MultiThread

date:       2018-12-12

author:     Augustine Tong

header-img: img/steve.jpg

catalog: true

tags:
    - C++
---

# CPP--Linux--MultiThread
**代码均为自己手打过一次, 输出结果均为自己的输出结果**.  

## 一个线程安全的Counter示例


```c++
#include <cstdint>  
  
// A thread-safe counter.  
class Counter : boost::noncopyable {  
    // copy-ctor and assignment should be private by default for a class.
public:
    Counter() : value_(0) {}
    int64_t value() const;
    int64_t getAndIncrease();

private:
    int64_t value_;
    mutable MutexLock mutex_;
};

int64_t Counter::value() const {
    MutexLockGuard lock(mutex_);
    return value_;
}

int64_t Counter::getAndIncrease() {
    MutexLockGuard lock(mutex_);
    int64_t ret = value_++;
    return ret;
}
// In a real world, atomic operations are preferred.
// 实际项目中，这个class用原子操作更合理，这里用锁仅仅是为了举例

```

## 对象的创建
对象构造要做到线程安全，唯一的要求是在构造期间不要泄露this指针，即    
- 不要在构造函数中注册任何回调  
- 也不要在构造函数中把this传递给跨线程的对象  
- 即便在构造函数的最后一行也不行  

之所以这么规定，是因为在构造函数执行期间对象还没有完成初始化，如果this被泄露(escape)给了其他对象，(其自身创建的子对象除外)，则别的线程有可能访问这个半成品对象，这会造成难以预料的后果。  

```cpp
// Don't do this
class Foo : public Observer
{
public:
    Foo(Observable* s)
    {
        s->register_(this); // wrong, un safe thread
    }

    virtual void update();
};

// Do this
class Foo : public Observer
{
public:
    Foo();
    virtual void update();

    // 另外定义一个函数，在构造之前执行回调函数的注册工作
    void observe(Observable* s)
    {
        s->register_(this);
    }
};

Foo* pFoo = new Foo;
Observable* s = getSubject();
pFoo->observe(s);  // 二段式构造，或者直接写 s->register_(pFoo);
```

这里也说明，二段式构造————即构造函数 + initialize()————有时候会是好办法，这虽然不符合C++教条，但是多线程下别无选择。另外，既然允许二段式构造，那么构造函数不必主动抛出异常，调用方法靠initialize()的返回值来判断对象是否构造成功，这能简化错误处理。   

即是是构造函数的最后一行也不要泄露this，因为Foo有可能是个基类，基类先于派生类构造，执行完Foo:Foo()的最后一行代码还会继续执行派生类的构造函数，这时most-derived class的对象还处于构造中，仍然不安全。  
相对而言，对象的构建做到线程安全还是比较容易的，毕竟曝光少，回头率为0，而析构的线程安全却不那么简单。

## 销毁太难
对象析构，这在单线程里不构成问题，最多需要注意避免空悬指针和野指针。  
而在多线程程序中，存在了太多的竞态条件。  
对一般的成员函数而言，做到线程安全的办法是让他们顺次执行，而不要并发执行(关键是不要同时读写共享状态)，也就是让每个成员函数的临界区不重叠。  
**成员函数用来保护临界区的互斥器本身必须是有效的**   
而析构函数破坏了这一假设，它会把mutex成员变量销毁掉。  

### mutex不是办法
mutex只能保证函数一个接一个的执行，考虑下面的代码，它试图用互斥锁来保护析构函数：
```cpp
Foo::~Foo()
{
    MutexLockGuard lock(mutex_);  
    // free internal state (1)  
}

void Foo::update() 
{
    MutexLockGuard lock(mutex_);  // (2)  
    // make use of internal state  
}

```

此时，有A、B两个线程都能看到Foo对象x，线程A即将销毁x，而线程B正准备调用x->update()
```cpp
extern Foo* x;

// thread A
delete x;
x = NULL; // helpless

// thread B
if (x)
{
    x -> update();
}
```  

1. 线程A执行到了析构函数的(1)处，已经持有了互斥锁，即将继续往下执行。  
2. 线程B通过了if(2)检测，阻塞在了(2)处  

接下来不知道会发生什么   
因为析构函数会把mutex_销毁，那么(2)处可能会永远阻塞下去，有可能进入“临界区”，然后core dump，或者发生其他更糟糕的情况。  

这个例子至少说明delete对象之后把指针置为NULL根本没用，如果一个程序要靠这个来防止二次释放，说明代码逻辑出了问题。  

### 作为数据成员的mutex不能保护析构
前面例子说明，作为class数据成员的MutexLock只能用于同步本class的其他数据成员的读和写，它不能保护安全地析构。因为MutexLock成员的生命周期最多和对象一样长，而析构动作可以说是发生在对象身故之后(或者身亡之时)。  
另外，对于基类对象，那么调用到基类析构函数的时候，派生类对象的那部分已经析构了，那么基类对象拥有的MutexLock不能保护整个析构过程。  
再说，析构过程本来也不需要保护，因为只有别的线程都访问不到这个对象时，析构才是安全的，否则就会有竞态条件发生。  

```cpp

void swap(Counter& a, Counter& b)
{
    MutexLockGuard aLock(a.mutex_);  // potential dead lock
    MutexLockGuard bLock(b.mutex_);
    int64_t value = a.value_;
    a.value_ = b.value_;
    b.value_ = value;
}
```

如果线程A执行swap(a, b); 而同时线程B执行swap(b, a); 就有可能死锁。

```cpp

Counter& Counter::operator=(const Counter& rhs)
{
    if (this == &rhs)
        return *this;

    MutexLockGuard myLock(mutex_);      // potential dead lock
    MutexLockGuard itsLock(rhs.mutex_);
    value_ = rhs.value_;        // 改成 value_ = rhs.value()会死锁
    return *this;
}
```

一个函数如果要锁住相同类型的多个对象，为了保证始终按照相同的顺序加锁，我们可以比较mutex对象的地址，始终先加锁地址较小的mutex。

## 线程安全的Observer
一个动态创建的对象是否还活着，光看指针是看不出来的(引用也一样看不出来)。指针就是指向了一块内存，这块内存上的对象如果已经销毁，那么就根本不能访问，既然不能访问又如何知道对象的状态呢？  
换句话说，判断一个指针是不是合法指针没有高效的办法，这是C/C++指针问题的根源。  
在面向对象的程序设计中，对象的关系主要有三种: composition、aggregation、association。
composition(组合/复合)关系在多线程里不会遇到什么麻烦，因为对象x的生命周期由其唯一的拥有者owner控制，owner析构时会把x也析构掉。  
从形式上看，x是owner的直接数据成员，或者scoped_ptr成员，抑或owner持有的容器的元素。  
后两种关系在C++里比较难办，处理不好就会造成内存泄露或者重复释放。association(关联/联系)是一种很宽泛的关系，它表示一个对象a用到了另一个对象b，调用了后者的成员函数。从代码的形式上看，a持有b的指针(或引用)，但是b的生命周期不由a单独控制。  
aggregation(聚合)关系从形式上看于association相同，除了a和b有逻辑上的整体与部分关系。如果b是动态创建的并在整个程序结束前有可能被释放，那么就会出现竞态条件。  
似乎一个简单的解决办法是: 只创建不销毁。程序使用一个对象池来暂存用过的对象，下次申请新对象时，如果对象池里有存货，就重复利用现有的对象，否则就新建一个。对象用完了，不是直接释放对象，而是放回池子里。这个办法至少避免访问失效对象的情况发生。  
这种山寨方法的问题有：  
- 对象池的线程安全，如何安全地、完整地把对象放回池子里，防止出现“部分放回”的竞态？(线程A认为对象x已经放回了，线程B认为对象x还活着。)   
- 全局共享数据引发的lock contention，这个集中化的对象池会不会把多线程并发的操作串行化？  
- 如果共享对象的类型不止一种，那么是重复实现对象池还是使用类模板？  
- 会不会造成内存泄露与分片？因为对象池占用的内存只增不减，而且多个对象池不能共享内存(想想为何)?  
回到正题上来，如果对象x注册了任何非静态成员函数回调，那么必然在某处持有了指向x的指针，这就暴露在race condition之下。  

 













