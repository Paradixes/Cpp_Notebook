## 条款21：必须返回对象时，不要尝试返回reference

### 一、 by value形式传递数据

**用以表现有理数的class**

```C++
class Rational {
public:
    Rational(int numerator = 0,				//条款24说明构造函数
             int denominator = 1);			//不声明为explicit的原因
    ...
private:
    int n, d;								//分子(numerator)和分母(denominator)
 friend
     const Rational							//条款3说明返回类型是const的原因
      operator* (const Rational& lhs,
                 const Rational& rhs);
};
```

**该class中通过by value的形式返回operator*的计算结果**

```C++
Rational a(1, 2);			//a = 1/2
Rational b(3, 5);			//b = 3/5
Rational c = a * b;			//c = 3/10
```

**此式operator*内需要自己构造一个Rational对象，计算完毕后再将其复制到c中**



### 二、 定义local变量，并在stack空间创建对象

### 1. 代码实现

```C++
const Rational& operator* (const Rational& lhs,
                           const Rational& rhs)
{
    Rational result(lhs.n * rhs.n, lhs.d * rhs.d);	//错误的代码
    return result;
}
```

### 2. 存在的问题

**result是个local对象，而local对象在函数退出前会被销毁，导致reference指向一个被删除的空间**



### 三、 在heap空间构造对象，并返回reference

### 1. 代码实现

```C++
const Rational& operator* (const Rational& lhs,
                           const Rational& rhs)
{													//错误代码 + 1
    Rational* result = new Rational(lhs.n * rhs.n, lhs.d * rhs.d);
    return *result;
}
```

### 2. 存在问题

**仍需要付出一个“构造函数调用”的代价，同时可能会导致没有能够对new的对象实施delete**

```C++
Rational w, x, y, z;
w = x * y * z;				//与operator*(operator*(x, y), z)相同
```

**调用两次operator\*，使用了两次new，但operator\*背后隐藏的指针不得而知，也就难以进行delete**



### 四、 让返回的reference指向在函数内部定义的static Rational对象

### 1. 代码实现

```C++
const Rational& operator* (const Rational& lhs,
                           const Rational& rhs)
{
    static Rational result;		//static对象，此函数将返回其reference
    
    result = ...;				//将乘积结果置于result内
    
    return result;
}
```

### 2. 存在问题

```C++
bool operator==(const Rational& lhs,
                const Rational& rhs);
Rational a, b, c, d;
...
if ((a * b) == (c * d)) {
    //当乘积相等时...
} else {
    //当乘积不相等时...
}
```

**此时，表达式`((a * b) == (c * d))`等价于`operator==(operator*(a, b), operator*(c, d))`，而由于`static`对象值是同一个，左右两式永远相等**

### 3. 奇特的想法（创建$array/vector$）

**此想法忽略了空间复杂度反而增加了成本，得不偿失**



### 五、 合理的想法

### 1. 通过$by-value$的方式传递数据

```C++
inline const Rational operator * (const Rational& lhs, const Rational& rhs)
{
    return Rational(lhs.n * rhs.n, lhs.d * rhs.d);
}
```

### 2. 不调用操作符重载，函数传参（此方法来自笔者自己的想法）

```C++
const Rational& Product(const Rational& lhs, const Rational& rhs
                 Rational& result)
{
	result.n = lhs.n * rhs.n;
    result.d = lhs.d * rhs.d;
    return *result;
}
```

