## 条款10：令operator= 返回一个reference to *this

### 一、赋值的连锁形式

```C++
int x, y, z;
x = y = z = 15;			//赋值连锁形式
```

**和下面等价**

```C++
x = (y = (z = 15));
```



### 二、 赋值操作需遵循的协议（返回reference）

### 1. 标准赋值操作

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

```C++
class Widget {
public:
    ...
	Widget& operator+=(const Widget& rhs);	//该协议适用于
    {										//+=, -=, *=等等
        ...
        return* this;
    }
    Widget& operator=(int rhs)				//此函数也适用
    {										//即使参数类型不符协定
        ...
        return *this;
    }
    ...
};
```

