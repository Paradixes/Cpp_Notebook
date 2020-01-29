## 条款25：写出不抛出异常的`swap`函数

### 一、 简单的`swap`函数（缺省）

### 1. 使用$copying$

```C++
namespace std {
	template<typename T>		//std::swap的典型实现
	void swap(T& a, T& b)		//置换a和b的值
	{
		T temp(a);
		a = b;
		b = temp;
	}
}
```

### 2. 不足之处
#### (1) 涉及三个对象的复制，效率较低
#### (2) 可能会抛出异常



### 二、通过指针置换实现`swap`函数

### 1. $pimpl$ ($pointer\ to\ implementation$)手法

**以指针指向一个对象，内含真正的数据**

```c++
class WidgetImpl {				//针对Widget数据而设计的class
public:							//细节忽视
    ...
private:
    int a, b, c;				//可能有许多数据
    std::vector<double> v;		//意味复制时间长
    ...
};
```

**主$class$：**

```C++
class Widget {								//该class使用pimpl手法
public:
    Widget(const Widget& rhs);
    Widget& operator=(const Widget& rhs)	//复制Widget时，令它复制其
    {										//WidgetImpl对象
        ...									//关于operator=的一般实现
        *pImpl = *(rhs.pImpl);				//细节见条款10，11和12
        ...
    }
    ...
private:
    WidgetImpl* pImpl;						//指针，所指对象内含数据
};
```



### 2. 特化`std::swap`

```C++
namespace std {
    template<>							//这是std::swap针对
    void swap<Widget>( Widget& a,		//“T是Widget”的特化版本
                       Widget& b)		//目前还不能通过编译
    {
        swap(a.pImpl, b.pImpl);			//置换Widgets时只需要置换pImpl指针
    }
}
```

**`template<>`表示它是`std::swap`的一个全特化版本**

**除了为标准$templates$（如`swap`）制造特化版本，`std`命名空间的任何东西不可被改变**

**此函数由于没有访问`private`的权限，因而无法通过编译**



### 3. 成员函数和特化的组合

```C++
class Widget {						//与前相同（无swap函数）
public:
    ...
    void swap(Widget& other)
    {
        using std::swap;			//此声明非常重要，之后解释
        swap(pImpl, other.pImpl);	//若要置换Widgets则置换其pimpl指针
    }
    ...
};

namespace std {
    template<>						//修改后的std::swap版本
    void swap<Widget>( Widget& a,
                       Widget& b )
    {
        a.swap(b);					//调用其swap成员函数
    }
}
```

**此做法可通过编译并与$STL$容器有兼容性，既提供`public swap`也有`std::swap`的特化**

**然而其不适用于$class\ templates$**



### 三、 适用于$class\ templates$的指针置换`swap`函数

### 1. $class\ templates$

```C++
template<typename T>
class WidgetImpl { ... };

template<typename T>
class Widget { ... };
```



### 2. 偏特化（错误示例）

```C++
namespace std {
    template<typename T>
    voud swap< Widget<T> >(Widget<T>& a,	//错误，不合法
                           Widget<T>& b)
    { a.swap(b); }
}
```

**此时我们尝试偏特化一个$function\ template$ (`std::swap`)，而$C++$中只能偏特化$class\ templates$**



### 3. 重载`swap`函数（错误示例）

```C++
namespace std {
    template<typename T>		//std::swap的重载版本
    void swap(Widget<T>& a,
              Widget<T>& b)		//仍错误
    { a.swap(b); }
}
```

**虽然可以编译通过，但在`std`的命名空间添加新的$templates$可能引发未知风险**



### 4. 将`swap`函数置于其他命名空间

```C++
namespace WidgetStuff {
    ...							//模板化的WidgetImpl等
    template<typename T>		//同前，内含swap成员函数
    class Widget { ... };
    ...
    template<typename T>		//non-member swap函数
    void swap(Widget<T>& a,		//这里不属于std命名空间
              Widget<T>& b);
    {
        a.swap(b);
    }
}
```

**$C++$的名称查找法则会找到`WidgetStuff`内的`Widget`专属版本**

**也可以不额外使用命名空间，但是这样的话$global$命名空间会变得极其混乱**

**此方案适用于$classes$和$class\ templates$，但仍需要特化`std::swap`**





### 5. 需要特化`std::swap`的理由

#### (1) 一旦编译器看到对`swap`的调用，便会查找适当的`swap`并调用它

```C++
template<typename T>
void doSomething(T& obj1, T& obj2)
{
    using std::swap;		//令std::swap在此函数内可用
    ...
    swap(obj1, obj2);		//为T型对象调用最佳swap版本
    ...
}
```

#### (2) 错误的修饰符

```C++
std::swap(obj1, obj2);		//错误的swap调用方式
```

**此修饰符强迫编译器只认`std`内的`swap`，因而不再可能调用一个`std`命名空间外的`swap`**

**所以$classes$需要对`std::swap`进行全特化**



### 四、 总结

### 1. 步骤

**判断缺省`swap`是否对调用$class$提供可接受的效率，如果是，则不需要做任何事，直接调用`swap`，如果不是，继续下面步骤**

**(1) 提供`public swap`成员函数，让它高校置换类型的对象值，且绝不能抛出异常**

**(2) 在$class$或$template$所在命名空间内提供一个$non$-$member\ swap$，调用上述`swap`成员函数**

**(3) 如果你编写的是$class$，为你的$class$转化`std::swap`，并令其调用`swap`成员函数**

**最后，如果调用`swap`，确保包含一个`using`声明式，然后不加任何$namspace$修饰符，直接调用`swap`**



### 2. 需要注意的地方

**`swap`成员函数绝不可以抛出异常，因为它的一个应用是帮助$classes$（和$class\ templates$）提供强烈的异常安全性保障（见条款29）**

**缺省版`swap`是以$copy$构造函数和$copy\ assignment$操作符为基础，而两者都允许抛出异常，因而自定版`swap`最主要保障不抛出异常**


