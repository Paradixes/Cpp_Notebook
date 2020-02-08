# 条款35：考虑 `virtual` 函数以外的选择

## `virtual` 函数继承

**设计一个角色相关 `class` ，并用成员函数 `healthValue` 计算健康指数**

```C++
class GameCharacter {
public:
    virtual int healthValue() const;	//返回人物健康指数
    ...									//derived class可重新定义
};
```

**不同的继承角色，对健康指数计算的方法可以不同，也可以采用缺省方法计算（[条款34](条款34：区分接口继承和实现继承.md)）**



## $Non$-$Virtual$ $Interface$ 手法实现 $Template\ Method$ 模式

### 1. 实现

**该流派主张将实际工作的 $virtual$ 函数 `deHealthValue` 放在 `private` 中，并保留 $non$-$virtual$ 的 `healthValue` 为 `public` 成员函数**

```C++
class GameCharacter {
public:
    int healthValue() const				//derived classes不重新定义它
    {									//条款36
        ...								//做一些事前工作
        int retVal = doHealthValue();	//做真正的工作
        ...								//做一些事后工作
        return retVal;
    }
    ...
private:
    virtual int doHealthValue() const	//derived classes可重新定义它
    {				
        ...								//缺省算法，计算健康指数
    }
};
```

+ 在此段代码中直接在 `class` 定义式内呈现成员函数本体，则全部把他们在暗中变成 `inline` （[见条款30](F:\滔天\文件\学校\大学\专业\C++\C++笔记\5.实现\条款30：彻底了解inline.md)）

+ 基本设计是 “令客户通过 `public` $non$-$virtual$ 成员函数间接调用 `private` $virtual$ 函数”，是 $Template\ Method$ 设计模式的一个独特表现形式，这个 $non$-$virtual$ 函数是 $virtual$ 函数的外覆器



### 2. 优点及注意事项

+ 优点：
  + 上述代码中 “做一些事前工作” 可包括锁定互斥器、制造运转记录项、验证class约束条件、验证函数先决条件，在 $virtual$ 函数被调用前设定好适当场景
  +  “事后工作” 可包括互斥器解锁、验证函数的事后条件、再次验证 `class` 的约束条件，在 $virtual$ 函数调用结束后，可用于清理场景
  + 可以使用外覆器包装 $virtual$ 函数，并处理一些执行 $virtual$ 函数所有必备的工作，防止客户自行构建 $virtual$ 函数时难以做到位 

+ 注意事项
  + $NVI$ 手法涉及在 $derived\ classes$ 内重新定义 $private\ virtual$ 函数，在此处并不矛盾， “重新定义 $virtual$ 函数” 表示某些事 “如何” 完成， “调用 $virtual$ 函数” 则表明它 “何时” 被完成
  + $virtual$ 函数没有必要一定是 `private` ，某些 `class` 继承体系要求 $derived\ classes$ 在 $virtual$ 函数的实现内必须调用其继承的函数（[条款27](F:\滔天\文件\学校\大学\专业\C++\C++笔记\5.实现\条款27：尽量少做转型动作.md)），这种情况下 $virtual$ 必须是 `protected` 
  + 若 $virtual$ 函数一定得是 `public` （[条款7](F:\滔天\文件\学校\大学\专业\C++\C++笔记\2.构造、析构、赋值运算\条款07：为多态基类声明virtual析构函数.md)），此时就无法实施 $NVI$ 手法



## $Function\ Pointers$ 实现 $Strategy$ 模式

### 1. 实现

**要求构造函数接收健康计算函数指针，便可以调用它进行实际计算**

```C++
class GameCharacter;				//前置声明
//以下是计算健康指数的缺省算法
int defaultHealthCalc(const GameCharacter& gc);

class GameCharacter {
public:
    typedef int (*HealthCalcFunc) (const GameCharacter&);	//简化变量声明
    explicit GameCharacter(HealthCalcFunc hcf = defaultHealthCalc)	//设定初始函数
     : healthFunc(hcf)				//初始化函数指针
     {}
    int healthValue() const
    { return healthFunc(*this); }	//返回函数指针
    ...
private:
    HealthCalcFunc healthFunc;		//声明函数指针变量
};
```



### 2. 应用

+ **同一人物类型的不同实体，可以有不同的健康计算函数**

    ```C++
    class EvilBadGuy: public GameCharacter {
    public:
        explicit EvilBadGuy(HealthCalcFunc hcf = defaultCalc)
         : GameCharacter(hcf)
         { ... }
        ...
    };

    int loseHealthQuickly(const GameCharacter&);	//健康指数计算函数1
    int loseHealthSlowly(const GameCharacter&);		//健康指数计算函数2

    EvilBadGuy ebg1(loseHealthQuickly);				//相同类型的人物
    EvilBadGuy ebg2(loseHealthSlowly);				// 搭配不同类型的计算方式
    ```

+ `HealthCalcFunc` 是个 `typedef` ，用来表现函数指针的某个具现体
+ **健康指数计算函数可在运行期变更**

+ 如果需要 $non$-$public$ 信息进行精确运算，就会产生问题，可能需要通过 $friends$ 或是将其实现的一部分提供 `public` 访问函数，而这做法将会弱化 `class` 的封装性，需要根据不同的情况抉择



## `tr1::function` 实现 $Strategy$ 模式

### 1. 实现

**不使用函数指针，而是使用一个类型为 `tr1::function` 的对象（[条款54]()）**

```C++
class GameCharacter;
int defaultHealthCalc(const GameCharacter& gc);	//同前

class GameCharacter {
public:
    //HealthCalcFunc可以是任何“可调用物”
   	//可被调用并接受任何兼容于GameCharacter之物，返回任何兼容于int的东西
    typedef std::tr1::function<int (const GameChrarcter&)> HealthCalcFunc;
    explicit GameCharacter(HealthCalcFunc hcf = defaultHealthCalc)
     : healthFunc(hcf)
     { }
    int healthValue() const
    { return healthFunc( * this); }
    ...
private:
    HealthCalcFunc healthFunc;
};
```

+ `std::tr1::function< int (const GameCharacter&)>` 代表 “接受一个 $reference$ 指向 `const GameCharacter` ，并返回 `int` ”



### 2. 应用

```C++
short calcHealth(const GameCharacter&);		//健康计算函数
											//返回short类型
struct HealthCalculator {					//为健康计算而设计的函数对象
    int operator() (const GameCharacter&) const
    { ... }
};
class GameLevel {
public:
    float health(const GameCharacter&) const;	//成员函数，用以计算健康
    ...											//返回float类型
};
class EvilBadGuy: public GameCharacter {		//同前
    ...
};
class EyeCandyCharacter: public GameCharacter {	//另一个人物类型
    ...											//假设其构造函数与
};												//EvilBadGuy同

EvilBadGuy ebg1(calcHealth);					//人物1，使用某个
												// 函数计算健康指数
EyeCandyCharacter ecc1(HealthCalculator());		//人物2，使用某个
												// 函数对象计算健康指数
GameLevel currentLevel;
...
EvilBadGuy ebg2{								//人物3，使用某个
    std::tr1::bind(&GameLevel::health,			//成员函数计算健康指数
                   currentLevel,
                   _1)
};
```

+ `tr1::function` 可以允许客户在计算人物健康指数时使用任何兼容的可调用物
+ 在人物3中，`GameLevel::health` 实际上接受两个参数（`GameLevel` 和 `GameCharacter`），然而 `GameCharacters` 的健康计算函数只接受一个参数
+ 因而，在此例中，我们使用 `currentLevel` 作为 `GameLevel` 对象，使用 `tr1::bind` 指明 `ebg2` 的健康计算函数总是以 `currentLevel` 作为 `GameLevel` 对象



## 古典的 $Strategy$ 模式

**将继承体系内的 $virtual$ 函数替换为另一个继承体系内的 $virtual$ 函数**

```C++
class GameCharacter;
class HealthCalcFunc {
public:
    ...
    virtual int calc(const GameCharacter& gc) const
    { ... }
    ...
};
HealthCalcFunc defaultHealthCalc;
class GameCharacter {
public:
    explicit GameCharacter(HealthCalcFunc* phcf = &defaultHealthCalc)
     : pHealthCalc(phcf)
     {}
    int healthValue() const
    { return pHealthCalc->calc(*this); }
    ...
private:
    HealthCalcFunc* pHealthCalc;
};
```

**只需要为 `HealthCalcFunc` 继承体系添加一个 $derived\ class$ 便可将一个既有的健康算法纳入使用**