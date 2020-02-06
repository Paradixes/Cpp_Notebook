# 条款07：为多态基类声明 $virtual$ 析构函数

## $non-virtual$ 析构函数可能的问题

### 1. $factory$ 函数

$base\ class$ 和 $derived\ class$ 的声明：

```C++
class TimeKeeper {
public:
    TimeKeeper();
    ~TimeKeeper();
    ...
};
class AtomicClock: public TimeKeeper { ... };		//继承TimeKeeper
class WaterClock: public TimeKeeper { ... };
class WristWatch: public TimeKeeper { ... };
```

+ 此时可以设计一个 $factory$ 函数， $factory$ 函数会返回 $base\ class$ 指针，指向新生成的 $derived\ class$ 对象



### 2. 产生的问题

+ 为遵守 $factory$ 函数规定，需要将 $factory$ 函数返回的每一个对象 `delete` 掉

    ```C++
    TimeKeeper* ptk = getTimeKeeper();			//从TimeKeeper继承体系
                                                //获得动态分配对象
    ...											//对其进行一些操作
    delete ptk;									//释放它，避免资源泄露
    ```

+ 然而这样会产生一系列问题
  + 因为依赖客户执行 `delete` 动作，会产生错误倾向（[条款13](F:\滔天\文件\学校\大学\专业\C++\C++笔记\3.资源管理\条款13：以对象管理资源.md)、[条款18](F:\滔天\文件\学校\大学\专业\C++\C++笔记\4.设计与声明\条款18：让接口被正确使用.md)）
  + `getTimeKeeper` 返回指针指向一个 $derived\ class$ 对象，此时使用 `delete` ，只会调用基类的析构函数，导致 $derived\ class$ 中的数据未被删除



## $virtual$ 析构函数

### 1. 防止 $derived\ class$ 内存泄露

**给 $base\ class$ 一个 $virtual$ 析构函数，然后 `delete` 便会销毁整个对象**

```C++
virtual ~TimeKeeper();
```
+ $string$ 、$STL$ 容器等标准库中的 `class` 一般都不带有 $virtual$ 析构函数，因而不能用于继承

  ```C++
  class SpecialString: public std::string {	//馊主意，std::string
      ...										// 有个non-virtual析构函数
  };
  
  SpecialString* pss = new SpecialString("Impending Doom");
  std::string* ps;
  ...
  ps = pss;				//SpecialString* => std::string*
  ...
  delete ps;				//string没有virtual析构函数，
  						// 无法调用SpecialString析构，
  						// 会导致SpecialString资源泄露
  ```




### 2. $virtual$ 析构函数的不足

+ 为了实现 $virtual$ 函数，对象必须携带一些信息，用来决定运行期间决定哪个 $virtual$ 函数需要调用，而这会大大增加对象体积

+ 因而不能无端将所有 `class`es 的析构函数，都声明为 $virtual$ ，只有当 `class` $ 至少内含一个 $virtual$ 函数的时候才为它声明

+ $string$ 、$STL$ 容器等标准库中的 `class` 一般都不带有 $virtual$ 析构函数，因而不能用于继承



## $pure\ virtual$ 析构函数

**将实体 $class$ (不带有 $pure\ virtual$ 的 $class$ ) 抽象化**

```C++
class AWOV {
public:
    virtual ~AWOV() = 0;		//声明pure virtual析构函数
};
```

+ $pure\ virtual$ 析构函数需要一份定义，用来响应每个基类中析构函数的调用

    ```C++
    AWOV::~AWOV() { }			//定义
    ```



## $virtual$ 析构函数应用的 $base\ class$

+ 仅有多态用途的 $base\ class$ 才需要 $virtual$ 析构函数，这些 $base\ classes$ 通过自身接口处理 $derived\ class$ 对象（例：`TimeKeeper` ）

+ 某些 `class` 的设计目的是作为 $base\ classes$ 的使用，但不是为了多态用途（如[条款6](条款06：拒绝编译器自动生成的函数.md)中的 `Uncopyable` 以及[条款47]()中的 `input_iterator_tag`），此时便不需要 $virtual$ 析构函数

