## 条款03：const的使用

### 一、 const的作用

1. 告诉程序员该值应该保持不变，便于程序员理解代码
2. 告诉编译器该值应该保持不变，便于编译器识别错误

### 二、 const的用法

### 1. 修饰指针

```C++
char greeting[] = "Hello";
char* p = greeting;					//non-const指针，non-const对象
const char* p = greeting;			//non-const指针，const对象
char const * p = greeting;			//non-const指针，const对象
char* const p = greeting;			//const指针，non-const对象
const char* const p = greeting;		//const指针，const对象
```

**PS: *const char\* p*与*char const \* p* 含义一致**



### 2. STL迭代器声明

```C++
std::vector<int> vec;
···
const std::vector<int>::iterator iter = vec.begin();	//iter与T* const作用一致
*iter = 10;												//改变指针，可用
++iter;													//值为const，不可修改
std::vector<int>::const_iterator citer = vec.begin();	//citer与const T*作用一致
*citer = 10;											//指针为const，不可修改
++iter;													//改变值，可用
```



### 3. 成员函数

### (1) 具体实现

头文件

```C++
class TextBlock{
public:
    ···
    const char& operator[](std::size_t position) const 	//operator[] for const对象
    { return text[position]; }
    char& operator[](std::size_t position)				//operator[] for non-const对象
    { return text[position]; }
}
```

实现文件

```C++
TextBlock tb("Hello");
cout << tb[0];							//调用non-const TestBlock::operator[]
const TextBlock ctb("Hello");
cout << ctb[0];							//调用const TestBlock::operator[]
tb[0] = 'x';							//没问题
ctb[0] = 'x';							//无法改写const TextBlock
```



### (2) 两种const概念

####  a. bitwise const

**定义：不改变对象内任何一个bit**

```C++
class CTextBlock{
public:
    ···
    char& operator[](std::size_t position) const //bitwise const声明
    { return pText[position]; }
private:
    char* pText;
}
```

**出现的矛盾（此时为logical const）**

```C++
const CTextBlock cctb("Hello");			//声明常量对象
char* pc = &cctb[0];					//调用const operator[]取得指针
*pc = 'J';								//对指针内部数据进行操作
```

---

#### b. logical const

**定义：可以改变内部一些bits，但只有在客户端检测不出的情况下可以允许**

**出现的矛盾（此时为bitwise const）**

```C++
class CTextBlock{
public:
    ···
    std::size_t length() const;
private:
    char* pTest;
    std::size_t textLength;				//文本块长度
    bool lengthIsValid;					//是否有效
};

std::size_t CTextBlock::length() const
{
    if (!lengthIsVaild){
		textLength = std::strlen(pText);	//在const内，无法改写textLength
        lengthIsValid = true;				//也不能对lengthIsValid赋值
    }
    return textLength;
}
```

**解决方法**

```C++
mutable std::size_t textLength;			//使用mutable使这些成员
mutable bool lengthIsValid;				//始终可以被修改
```



### (3)const和non-const的转化（转型casting）

```c++
class TextBlock{
    ···
    const char& operator[](std::size_t position) const
    {
        ···
        return text[position];
    }
    char& operator[](std::size_t position)				//调用const op[]，得到non-const
    {
        return 
            const_char<char&>(							//将op[]返回的const转除
        		static_cast<const TextBlock&>(*this)	//为*this加上const(避免调用自己)
            		[position]							//调用const op[]
        );
    }
};
```

