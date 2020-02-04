# 条款32：确定 `public` 继承塑模出 $is$-$a$ 关系

## `public` 继承意味 "$is$-$a$" （是一种）关系

### 1. `public` 继承的描述

```C++
class Person { ... };
class Student: public Person { ... };	//Student是Person的public继承
```



### 2. `public` 继承的含义

+ 若令 `class D` 以 `public` 形式继承 `class B` ，每一个**类型为 `D` 的对象同时也是类型为 `B` 的对象**，**反之则不成立**

  ```C++
  void eat(const Person& p);			//任何人都会吃
  void study(const Student& s);		//只有学生才到校学习
  Person p;							//p是人
  Student s;							//s是学生
  
  eat(p);								//没问题，p是人
  eat(s);								//没问题，s是学生，而学生也是(is-a)人
  
  study(p);							//没问题，s是个学生
  study(s);							//错误，p不一定是学生
  ```

+ 该论点仅对 `public` 继承成立，`private` 继承和意义可见[条款39]()



## 继承中的冲突

**企鹅和鸟的问题：**

```C++
class Bird {
public:
    virtual void fly();			//鸟可以飞
    ...
};

class Penguin: public Bird {	//企鹅是一种鸟，但它不能飞
    ...
};
```



### 1. 令 `Penguin` 重新定义 `fly` 函数

**令 `fly` 函数产生运行期错误**

```C++
void error(const std::string& msg);	//定义于另外一处
class Penguin: public Bird {
public:
    virtual void fly() { error("Attempt to make a penguin fly!"); }
    ...
};
```

然而，“企鹅不会飞”这一限制可由编译器强制实施，只有在运行期才可以被检测出错误



### 2. 建立分支继承关系

```C++
class Bird {
    ...							//没有声明fly函数
};

class FlyingBird: public Bird {
public:
    virtual void fly();
    ...
};
class Penguin: public Bird {
    ...							//没有声明fly函数
};
```

若系统的重点并不在于区分鸟会不会飞，原先的 “双 `class` 继承体系” 就足够了，没必要刻意区分鸟会不会飞



## `public` 继承主张与现实冲突

+ 矩形：

    ```C++
    class Rectangle {
    public:
        virtual void setHeight(int newHeight);
        virtual void setWidth(int newWidth);
        virtual int height( ) const;		//返回当前值
        virtual int width( ) const;
        ...
    };
    void makeBigger(Rectangle& r)			//这个函数用来增加r的值
    {
        int oldHeight = r.height();
        r.setWidth(r.width() + 10);			//为r的宽度加10
        assert(r.height( ) == oldHeight);	//判断r的高度是否未曾改变
    }
    ```

+ 正方形继承矩形

  ```C++
  class Square: public Rectangle { ... };
  Square s;
  ...
  assert(s.width() == s.height());	//这对所有正方形一定为真
  makeBigger(s);						//由于继承，s是一种(is-a)矩形
  									//所以我们可以增加其面积
  assert(s.width() == s.height());	//对所有正方形仍然为真
  ```
  + 调用 `makeBigger` 之前，`s` 的高度和宽度相同
  + 在 `makeBigger` 函数内，`s` 的宽度改变，但高度不变
  + `makeBigger` 返回之后，`s` 的高度再度和其宽度相同

+ `public` 继承主张，能够施行于 $base$ `class` 对象身上的每件事情，也可以施行于 $derived$ `class` 对象上

+ $is$-$a$ 并非是唯一存在于 `class`es 之间的关系，另两个常见关系是 $has$-$a$ （有一个）和 $is$-$implemented$-$in$-$terms$-$of$（根据某物实现出），这些关系可见[条款38]()和[条款39]()