# 条款38：通过复合塑模 $has$-$a$ 或 “根据某物实现”

**复合 ($composition$) 等同分层 ($layering$)、内含 ($containment$)、聚合 ($aggregation$)、内嵌 ($embedding$)**

**当复合发生于应用域内的对象之间，表现出 $has$-$a$ 的关系；当它发生于实现域内则是表现 “根据某物实现” 关系**



## 通过复合塑模 $has$-$a$

```C++
class Address { ... };			//住址
class PhoneNumber { ... };		//电话号

class Person {
public:
    ...
private:
    std::string name;			//合成成分物
    Address address;			//同上
    PhoneNumber voiceNumber;	//同上
    PhoneNumber faxNumber;		//同上
};
```

+ `Person` 有一个名称，一个地址，以及语音和传真两个电话号码，在上例中表明 “人有一个电话号码” 或 “人有一个地址”，因而是 $has$-$a$ 关系



## 通过复合塑模 $is$-$implemented$-$in$-$terms$-$of$

### 1. $is$-$implemented$-$in$-$terms$-$of$ 与 $is$-$a$ 的区别

+ 标准库中提供 `set` $template$ ，但该 `set` 牺牲了空间成本来提高时间效率，若需求不一致则需要自己构建
+ 表现不重复对象组成的 `set`s ，可以调用标准库中的 `std::list` ，然而采用 `public` 继承是错误的做法

    ```C++
template<typename T>					//将list应用于Set，错误的做法
    class Set: public std::list<T> { ... };
    ```
    
+ 由于 `list` 对象可以含重复对象，而 `Set` 不行，因而不符合 $is$-$a$ 的标准，不能用 `public` 继承（[条款32](条款32：确定public继承塑模出is-a关系.md)）

+ 因而应该使用复合形式实现

  ```C++
  template<class T>					//将list用于Set，正确的做法
  class Set {
  public:
      bool member(const T& item) const;
      void insert(const T& item);
      void remove(const T& item);
      std::size_t size() const;
  private:
      std::list<T> rep;				//用来表述Set的数据
  };
  ```



### 2. `Set` 的具体实现

```C++
template<typename T>
bool Set<T>::member(const T& item) const
{
    return std::find(rep.begin(), rep.end(), item) != rep.end();
}

template<typename T>
void Set<T>::insert(const T& item)
{
    if(!member(item)) rep.push_back(item);
}

template<typename T>
void Set<T>::remove(const T& item)
{
    typename std::list<T>::iterator it =			//见条款42
        std::find(rep.begin(), rep.end(), item);	//对"typename"的讨论
    if (it != rep.end()) rep.erase(it);
}

template<typename T>
std::size_t Set<T>::size() const
{
    return rep.size();
}
```

+ 这些函数都很简单，因而都很适合作为 `inline` 函数，但在做出 `inline` 决定前应该先了解[条款30](F:\滔天\文件\学校\大学\专业\C++\C++笔记\5.实现\条款30：彻底了解inline.md)