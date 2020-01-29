## 条款24：若所有参数都需要类型转换，则采用$non-member$函数

### 一、 $non-explicit$构造函数的$member$函数

### 1. 表现有理数的$class$

```C++
class Rational {
public:
	Rational(int numerator = 0,			//构造函数刻意不为explicit
			  int denominator = 1);		//允许int-to-Rational隐式转换
	int numerator() const;				//分子和分母
	int denominator() const;			//的访问函数
private:
	...
};
```


### 2. $member$函数实现

```C++
class Rational {
public:
	...
	const Rational operator* (const Rational& rhs) const;
};
```
**函数写法可参考[条款3](F:\滔天\文件\学校\大学\专业\C++\C++笔记\1.习惯C++\条款03：const的使用.md)、[条款20](条款20：宁以pass-by-reference-to-const替换pass-by-value.md)、[条款21](条款21：必须返回对象时，不要尝试返回reference.md)**



### 3. member函数调用

```C++
Rational oneEighth(1, 8);
Rational oneHalf(1, 2);
Rational result = oneHalf * oneEighth;		//很好
result = result * oneEighth;				//很好
```


### 4. 可能产生的问题（混合式运算）

```C++
result = oneHalf * 2;		//很好
result = 2 * oneHalf;		//错误
```
对应函数形式：
```C++
result = oneHalf.operator*(2);	//很好
result = 2.operator*(oneHalf);	//错误
```


**在第一个表达式中，编译器会尝试用2构造Rational class**

```C++
const Rational temp(2);			//根据2建立暂时性的Rational对象
result = oneHalf * temp;
```
**在第二个表达式中，当编译器找不到2对应的class时，会尝试调用non-member operator***

```C++
result = operator*(2, oneHalf);
```


### 二、 explicit构造函数的member函数

#### 可能产生的问题（混合式运算）
```C++
result = oneHalf * 2;		//错误，在explicit构造函数中
							//无法将2转换为Rational
result = 2 * oneHalf;		//同样错误
```



### 三、 non-member函数

### 1. 函数实现
```C++
class Rational {
	...
};

const Rational operator*(const Rational& lhs,		//现在成为non-member函数
						 const Rational& rhs)
{
	return Rational(lhs.numerator() * rhs.numerator(),
					lhs.denominator() * rhs.denominator());
}
```



### 2. 函数调用

```C++
Rational oneFourth(1, 4);
Rational result;
result = oneFourth * 2;	//没问题
result = 2 * oneFourth;	//没问题
```


### 3. operator*不应该成为friend函数

#### (1) 通过public接口完全可以完成任务
#### (2) 能避免friend函数，尽可能避免friend函数