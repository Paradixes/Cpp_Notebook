## 条款28：避免返回 $handles$ 指向对象内部成员

### 管理矩形的 $class$

```C++
class Point {					//这个class用来表述“点”
public:
    Point(int x, int y);
    ...
    void setX(int newVal);
    void setY(int newVal);
    ...
};

struct RectData {		//这些“点”数据用来表现一个矩形
    Point ulhc;			//ulhc = "upper left-hand corner"
    Point lrhc;			//lrhc = "lower right-hand corner"
};

class Rectangle {
    ...
private:
    std::tr1::shared_ptr<RectData> pData;	//关于tr1::shared_ptr
};											// 见条款13
```

### 一、 简单实现及问题

### 1. 简单实现

**根据[条款20](F:\滔天\文件\学校\大学\专业\C++\C++笔记\4.设计与声明\条款20：宁以pass-by-reference-to-const替换pass-by-value.md)，以 $by\ reference$ 形式传递用户自定义类型比 $by\ value$ 方式更高效**

```C++
class Rectangle {
public:
    ...
    Point& upperLeft( ) const { return pData->ulhc; }
    Point& lowerRight( ) const { return pData->lrhc; }
    ...
};
```

**然而，此设计却自相矛盾，尽管`upperLeft​`和`lowerRight`被声明为`const`成员函数，但却可以通过 $reference$ 被修改**



### 2. 通过 $referneces$ 更改内部数据

```C++
Point coord1(0, 0);
Point coord2(100, 100);
const Rectangle rec(coord1, coord2);	//rec是个const矩形
										// 从(0, 0)到(100, 100)
rec.upperLeft( ).setX(50);				//现在rec却变成
										// 从(50, 0)到(100, 100)
```

+ **成员变量的封装性等于“返回其 $reference$ ”的函数的访问级别。在本例中，虽然`ulhc`和`lrhc`都被声明为`private`，但实际上却是`public`，因为`public`函数传出其 $reference$ **

+ **如果`const`成员函数传出一个 $reference$，后者所指数据与对象自身有关，却又被存储于对象之外，那么该函数的调用者可以修改这笔数据。这正是 $bitwise\ constness$ 的一个附带结果，[见条款3](F:\滔天\文件\学校\大学\专业\C++\C++笔记\1.习惯C++\条款03：const的使用.md)**

+ **如果返回的是指针或迭代器，相同的情况仍会发生，这三者统称为 $handles$ （号码牌，用来取得某对象）**

+ **此外，访问级别较高的成员函数，不应该返回访问级别低的成员函数，不然，后者的访问级别会与前者相同**



### 二、改进实现及问题

### 1. 改进实现

```C++
class Rectangle {
public:
    ...
    const Point& upperLeft( ) const { return pData->ulhc; }
    const Point& lowerRight( ) const { return pData->lrhc; }
    ...
};
```

**有了这样的改变，客户只能读取，而不能改变 $Points$**

**但即便如此，返回 $handles$ 可能导致 $dangling\ handles$（空悬的号码牌）**



### 2. $dangling\ handles$

#### (1) 内部功能

```C++
class GUIObject { ... };
const Rectangle							//以by value形式返回一个矩形
 boundingBox(const GUIObjects& ogj);	//条款3解释为什么返回const
```



#### (2) 客户调用

```C++
GUIObject* pgo;							//让pgo指向某个GUIObject
...
const Point* pUpperLeft =				//取得一个指针指向外框左上点
 &(boundingBox(*pgo).upperLeft());
```

**对`boundingBox`的调用临时获得一个新的`Rectangle`对象，随后`upperLeft`作用于该对象上，返回一个 $reference$ 指向 $temp$ 的一个内部成分，然而在语句执行结束后，该对象会被销毁，间接导致该对象内`Points`的析构，最终导致`pUpperLeft`变成悬空**

