# 条款39：谨慎使用 `private` 继承

##  `private` 继承的意义

### 1. `private` 继承与 `public` 继承

```C++
class Person { ... };
class Student: private Person { ... };	//private继承
void eat(const Person& p);				//任何人都会吃
void study(const Student& s);			//只有学生才在校学习

Person p;								//p是人
Student s;								//s是学生

eat(p);									//没问题，p是人，会吃
eat(s);									//错误，学生是人，但却不能吃
```

+ 若 `class` 之间的继承关系是 `public` ，则是 $is$-$a$ 的关系，编译器会将 $derived\ class$ 对象转换为 $base\ class$ 对象（[见条款32](条款32：确定public继承塑模出is-a关系.md)） ，而 `private` 继承则不会如此，因而 `eat(s)` 调用失败

+ 由 `private` $base\ class$ 继承而来的所有成员，在 $derived\ class$ 中都会变成 `private` 属性，无论它们在原来的 $base\ class$ 中原本是 `protected` 或 `public` 属性



### 2. `private` 继承的意义

+ `private` 继承意味着 $implemented$-$in$-$terms$-$of$（根据某物实现出），即如果让 `class D` 以 `private` 形式继承 `class B` ，是为了采用 `class B` 中已经准备妥当的某些特性，`B` 和 `D` 之间不存在任何观念上的关系
+ 简略来说，`private` 继承意味着只有实现部分被继承，接口部分被略去

+ `private` 继承的意义和复合的意义产生重叠（[条款38](条款38：通过复合塑模出has-a或根据某物实现出.md)），在二者的选择间尽可能使用复合，必要时才使用 `private` 继承
+ 主要当 `protected` 成员或 $virtual$ 函数牵扯进来，或是一种激进情况（ $base\ class$ 不带数据）才考虑 `private` 继承 



## `private` 继承的使用以及替代

### 1. $is$-$implemented$-$in$-$terms$-$of$ 场景下的使用

#### (1) `private` 继承

一个定时器 `class` ：

```C++
class Timer {
public:
    explicit Timer(int tickFrequency);
    virtual void onTick() const;		//定时器每滴答一次
    ...									// 此函数就被自动调用
};
```

`class Widget` 需要使用定时器的功能并重定义其虚函数：

```c++
class Widget: private Timer {
private:
    virtual void onTick() const;	//查看Widget的数据...等等
};
```

+ 为了让 `Widget` 重新定义 `Timer` 内的 $virtual$ 函数，但若让其继承接口，会导致接口的错误使用（[条款18](F:\滔天\文件\学校\大学\专业\C++\C++笔记\4.设计与声明\条款18：让接口被正确使用.md)）

+ 因而采用 `private` 继承，将 `onTick` 函数在 `Widget` 内变成 `private`



#### (2) `public` 继承和复合的组合

```C++
class Widget {
private:
    class WidgetTimer: public Timer {
    public:
        virtual void onTick() const;
        ...
    };
    WidgetTimer timer;
    ...
};
```

该设计比只使用 `private` 继承复杂一些，但该设计方法有其好处

+ 若 `Widget` 拥有 $derived\ classes$，但又想阻止 $derived\ classes$ 重新定义 `onTick` 函数，若 `Widget` 用 `private` 继承 `Timer`，上面的想法便不可能实现（[条款35](条款35：考虑·virtual函数以外的选择.md)：$derived\ classes$ 可以重定义 $virtual$ 函数，即便不能调用它们） 

+ 将 `Widget` 编译依存关系降至最低，如果 `Widget` 继承 `Timer`，当 `Widget` 编译时 `Timer` 的定义必须可见，而如果将 `WidgetTimer` 移出 `Widget` 之外而 `Widget` 内含指针指向 `WidgetTimer` ，则可以只带简单的 `WidgetTimer` 声明式（[条款31](F:\滔天\文件\学校\大学\专业\C++\C++笔记\5.实现\条款31：降低文件间的编译依存关系.md)）



### 2. $base\ class$ 不带数据

**不带数据的 `class` ：**

+ **没有 $non$-$static$ 成员变量**
+ **没有 $virtual$ 函数（因为这种函数的存在会给每个对象带来一个 $vptr$，[条款7](F:\滔天\文件\学校\大学\专业\C++\C++笔记\2.构造、析构、赋值运算\条款07：为多态基类声明virtual析构函数.md)）**
+ **没有 $virtual\ base\ classes$（因为这样的 $base\ classes$ 也会招致体积上的额外开销，[条款40]()）**

```C++
class Empty { };

class HoldsAnInt {
private:
    int x;
    Empty e;
};
```

+ 若如上所示，`sizeof(HoldAnInt) > sizeof(int)` ，可见 `Empty` 成员变量占用了内存，若继承 `Empty` 则不会占用内存空间 

    ```C++
    class HoldAnInt: private Empty {
    private:
        int x;
    };
    ```
    
    + 此时，`sizeof(HoldAnInt) == sizeof(int)`，这是所谓的 $EBO(empty\ base\ optimization)$ ，若客户非常在意空间，那么需要注意 $EBO$
    + $EBO$ 一般只在单一继承下有效，无法施行于 “用于多个 $base$ ” 的 $derived\ classes$ 身上
    + 现实中的 $empty\ class$ 并非真的 $empty$ ，往往含有 `typedef`s, `enum`s, `static` 成员变量，或 $non$-$virtual$ 函数
    
+ 大部分 `class` 并非 $empty$，因而 $EBO$ 很少成为 `private` 继承的正当理由

