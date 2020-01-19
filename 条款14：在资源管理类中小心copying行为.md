## 条款14：在资源管理类中小心copying行为

### 一、 资源取得的时候便应该被初始化（RAII）

### 1. 资源在构造期间获得，在析构期间释放

```C++
void lock(Mutex* pm);		//锁定pm所指的互斥器
void unlock(Mutex* pm);		//将互斥器解除锁定
```

#### (1) 管理class

```C++
class Lock {
public:
    explicit Lock(Mutex* pm);
     : mutexPtr(pm)
     { lock(mutexPtr); }			//获得资源
    ~Lock() {unlock(mutexPtr); }	//释放资源
private:
    Mutex *mutexPtr;
};
```



#### (2) 客户的合理调用

```C++
Mutex m;			//定义你所需要的互斥器
...
{					//建立区块定义critical section
Lock m1(&m);		//锁定互斥器
	...				//执行critical section操作
}					//自动解除互斥器锁定
```



#### (3) 被复制时遇到的难题

```C++
Lock ml1(&m);		//锁定m
Lock ml2(ml1);		//将ml1复制到ml2上，会如何？
```



### 二、 做出选择

### 1. 禁止复制

**可参考[条款6](条款06：拒绝编译器自动生成的函数.md)**

```C++
class Lock: private Uncopyable {	//禁止复制
public:
    ...
};
```



### 2. 引用计数法

**希望保有资源直到它最后一个使用者消失**

```C++
class Lock {
public:
    explicit Lock(Mutex* pm)		//以某个Mutex初始化shared_ptr
     : mutexPtr(pm, unlock)			// 并以unlock函数作为删除器
     {
         lock(mutexPtr.get());		//参见条款15
     }
private:
    std::tr1::shared_ptr<Mutex> mutexPtr;	//使用shared_ptr替换raw ptr
};
```

**自定义`shared_ptr`的删除器**



### 3. 复制底部资源

**可随意复制资源，在不需要运用某副本时，需要资源管理类释放它，此时进行的是“深度拷贝”**

**某些标准字符类型由指针构成，此时深度复制便需要复制其指针和所指内存**



### 4. 转移底部资源拥有权

**确保永远只有一个RAII对象指向一个资源，如被复制，则会将资源拥有权转移到目标物（同`auto_ptr`）**