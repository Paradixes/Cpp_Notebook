## 条款30：彻底了解`inline`

### 一、 `inline`函数的“调用方式”及优劣

### 1. `inline`函数的优势

+ 看起来像函数，但却节省了函数调用的额外开销（可见[条款2](F:\滔天\文件\学校\大学\专业\C++\C++笔记\1.习惯C++\条款02：以编辑器替换预处理器.md)）
+ 编译器最优化机制通常被设计用来浓缩这些 “不含函数调用” 的代码



### 2. “调用方式”及劣势

+ `inline`将对 “对此函数的每一个调用” 都用**函数本体**替换

+ 将函数`inline`会造成程序体积过大，降低效率
+ 然而如果`inline`本体很小，编译器对**函数本体**产生的代码可能比**函数调用**产生的代码更少

+ `inline`只是对编译器的申请，不是强制命令

  + 隐喻提出`inline`申请（将函数定义于 $class$ 定义式内）

  ```C++
  class Person {
  public:
      ...
      int age() const { return theAge; }		//隐喻的inline申请
      ...										//age被定义于class定义式内
  private:
      int theAge;
  }
  ```

  + 明确申请 `inline` 函数（在函数定义式前加上`inline`）

  ```C++
  template<typename T>
  inline const T& std::max(const T& a, const T& b)	//std::max前有
  { return a < b ? b : a; }							//关键字"inline"
  ```



### 二、 编译器对`inline`函数的做法

### 1. `inline`函数和`template`s

+ `inline`函数需要置于头文件内，因为大多数建置环境**在编译过程中** $inlining$，而为了将 “函数调用” 替换为 “被调用函数本体”，编译器需要知道函数的样子
+ `template`s通常也置于头文件内，因为它一旦被使用，编译器为了将它具现化，需要知道它的样子
+ `template`的具现化和 $inlining$ 无关，而`inline`函数需要成本，会导致代码膨胀（这对`template`作者很重要，[条款44]()）



### 2. `inline`是个申请，编译器可以拒绝

+ 大部分编译器会拒绝太过复杂（带有循环或递归）的函数 $inlining$
+ 对于所有`virtual`函数的调用也会使 $inlining$ 落空，因为`virtual`函数需要等到运行期才知道该调用哪个函数
+ 如果无法将申请的函数`inline`化，编译器会给出一个警告信息（[见条款53]()）



### 3. 有时候编译器也会为`inline`函数生成函数本体

**如果程序要取得某个`inline`函数的地址，编译器便会生成一个 $outlined$ 函数本体**

```C++
inline void f() { ... }	//假设这个编译器有意愿inline“对f的调用”
void ( * pf )( ) = f;	//pf指向f
...
f();					//这个调用被inlined
pf();					//这个调用或许不被inlined，因为它通过函数指针达成
```

**即使你从未使用函数指针，编译器可能会生成构造函数和析构函数的 $outline$ 副本，为了获得指向那些函数的指针**



### 4. 构造函数和析构函数是`inline`的糟糕选择

+ 一个 “空” 的构造函数：

    ```C++
    class Base {
    public:
        ...
    private:
        std::string bm1, bm2;		//base成员
    };

    class Derived: public Base {
    public:
        Derived() { }				//derived的构造函数为空
        ...
    private:
        std::string dm1, dm2, dm3;	//derived成员
    }
    ```

+ 编译器会为这个空的构造函数生成许多代码

  ```C++
  Derived::Derived()				//“空白Derived构造函数”观念性实现
  {
      Base::Base();						//初始化“Base成分”
      try { dm1.std::string::string(); }	//试图构造dm1
      catch (...) {						//如果抛出异常就
          Base::~Base();					//销毁base class成分
          throw;							//传播该异常
      }
      try { dm2.std::string::string(); }	//试图构造dm2
      catch (...) {
          dm1.std::string::~string();
          Base::~Base();
          throw;
      }
      ...
  }
  ```

  **编译器真正制造的代码要更复杂，同理也适用于Base构造函数，编译器生成的代码也会影响编译器是否选择`inline`**



### 三、 `inline`的合理选择

### 1. 程序库设计者谨慎选择

+ `inline`函数无法随着程序库的升级而升级
+ 一旦程序库设计者打算修改`inline`函数，客户必须要重新编译，而 $non$-`inline`函数只需要重新连接



### 2. 程序开发在`inline`函数上遇到的难题

+ 大部分调试器都对`inline`函数束手无策
+ 由于`inline`函数本质上是不存在的函数，因而在函数内设置不了断点



### 3. 掌握合乎逻辑的策略

+ 将`inline`函数局限在两个方面
  + 一定成为`inline`（[条款46]()）
  + 十分平淡无奇（例如前例中的`Person::age`）函数

+ 80-20经验法则
  + 平均一个程序80%的执行时间花在20%的代码上
  + 找出那20%的程序，将它`inline`或瘦身