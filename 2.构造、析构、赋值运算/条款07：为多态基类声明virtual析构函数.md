## 条款07：为多态基类声明 $virtual$ 析构函数

### 一、 $non-virtual$ 析构函数可能的问题

### 1. 出现的问题

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

**当在派生类中使用`delete`时，会调用基类的析构函数，导致派生类中的数据未被删除**

```C++
TimeKeeper* ptk = getTimeKeeper();			//从TimeKeeper继承体系
											//获得动态分配对象
...											//对其进行一些操作
delete ptk;									//释放它，避免资源泄露
```

**PS: $string$ 、$STL$ 容器等标准库中的 $class$ 一般都不带有 $virtual$ 析构函数**



### 2. 解决方案

```C++
virtual ~TimeKeeper();
```



### 二、 随意使用 $virtual$ 析构函数的问题

**为了实现 $virtual$ 函数，对象必须携带一些信息，用来决定运行期间决定哪个 $virtual$ 函数需要调用，而这会大大增加对象体积**



### 三、$pure\ virtual$ 析构函数的使用

### 1. 目的

**将实体 $class$（不带有 $pure\ virtual$ 的 $class$ ）抽象化**



### 2. 实现

```C++
class AWOV {
public:
    virtual ~AWOV() = 0;		//声明pure virtual析构函数
};
```

**PS: $pure\ virtual$ 析构函数需要一份定义，用来响应每个基类中析构函数的调用**

```C++
AWOV::~AWOV() { }				//定义
```



### 四、 基类的设计目的

#### 1. 多态用途

#### 2. 通过 $base\ classes$ 接口处理 $derived\ class$ 对象（例：`TimeKeeper`）