## 条款23：宁以 $non$-$member$、$non$-$friend$ 替换 $member$ 函数

### 管理网页浏览器的 $class$

```C++
class WebBrowser {
public:
    ...
    void clearCache( );				//清除下载缓存区
    void clearHistory( );			//清除访问历史纪录
    void removeCookies( );			//移除系统中所有cookies
    ...
};
```

### 一、 $non$-$member$ 和 $member$ 函数

### 1. 用 $member$ 函数实现全部删除操作

```C++
class WebBrowser {
public:
    ...
    void clearEverything( );		//调用上述三个函数
    ...
};
```

**函数内调用上述三个成员函数**



### 2. 用 $non$-$member$ 函数实现全部删除操作

```C++
void clearBrowser(WebBrowser& wb) {
    wb.clearCache();
    wb.clearHistory();
    wb.removeCookies();
}
```



### 3. 比较

#### 封装性：

**对象封装性由对象可被多少函数调用来体现，越多函数调用对象，对象的封装性越差**

**对于非`private`对象而言，其可被无数函数调用，因而不具有任何封装性（[条款22](条款22：将成员变量声明为private.md)）**

**而对于成员函数而言，增加了对象被函数掉用的次数**

**非成员函数则只是调用了成员函数，不会增加对象被函数调用的次数，因而封装性更好**

**$friends$ 函数对`private`成员的访问权限和 $member$ 函数一样，因而其封装性也一样不好**



### 二、 使用工具类的函数

**工具类（ $utility\ class$ ）的函数调用不会影响原 $class$ 的封装性**

### 1. 让工具 $class$ 和原 $class$ 在同一个命名空间

```C++
namespace WebBrowserStuff {
    class WebBrowser { ... };
    void clearBrowser(WebBrowser& wb);
    ...
}
```

**$namespace$ 可以跨越多个源码文件，同时对于原 $class$ 没有特殊访问权限**



### 2. 分离便利函数

**客户有不同的需求，但没必要把所有需求都放在一起**

核心头文件：

```C++
//头文件"webbrowser.h"--此头文件针对class WebBrowser自身以及核心机能
namespace WebBrowserStuff {
    class WebBrowser { ... };
    ...							//核心机能（例如客户都需要的non-member函数）
}
```

分支工具 $class1$：

```C++
//头文件"webbrowserbookmarks.h"
namespace WebBrowserStuff {
    ...							//与书签相关的便利函数
}
```

分支工具 $class2$：

```C++
//头文件"webbrowsercookies.h"
namespace WebBrowserStuff {
    ...							//与cookie相关的便利函数
}
```

**这正是 $C++$ 标准库的组织方式，在标准库中含有数十个头文件，每个头文件声明`std`的某些机能**

**客户只需要添加更多 $non$-$member$、$non$-$friend$ 函数到此命名空间**