# 条款10：令赋值操作返回 $reference\ to\ *this$

## 赋值的连锁形式

```C++
int x, y, z;
x = y = z = 15;			//赋值连锁形式
```

**和下面等价**

```C++
x = (y = (z = 15));
```



## 赋值操作需遵循的协议（返回 $reference$ ）

### 1. 标准赋值操作

**为了实现 “连锁赋值”，赋值操作必须要返回 $reference$ 指向操作符左侧实参**

```C++
class Widget {
public:
    ...
	Widget& operator=(const Widget& rhs);	//返回类型是reference
    {										//指向当前对象
        ...
        return* this;						//返回左侧对象
    }
    ...
};
```



### 2. 赋值相关运算

**该协议也适用于赋值相关操作（`+=`, `-=`, `*=` 或是不同类型参数）**

```C++
class Widget {
public:
    ...
	Widget& operator+=(const Widget& rhs);	//该协议适用于
    {										//+=, -=, *=等等
        ...
        return *this;
    }
    Widget& operator=(int rhs)				//此函数也适用
    {										// 即使参数类型不符协定
        ...
        return *this;
    }
    ...
};
```

**该协议被标准库类型 `string`, `vector`, `complex`, `tr1::shared_ptr` 或即将提供的类型（[见条款54]()）共同遵守**