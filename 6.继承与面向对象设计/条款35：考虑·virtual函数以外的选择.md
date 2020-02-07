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



## $Non$-$Virtual$ $Interface$ 手法

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
  + 可以使用外覆器包装 $virtual$ 函数