## 条款09：不在析构和构造函数中调用virtual函数

### 一、 可能出现的非法调用

base class:

```C++
class Transaction {								//所有交易的base class
public:
    Transaction( );
    virtual void logTransaction() const = 0;	//做出一份因类型不同而不同的日志记录
    ...
};
Transaction::Transaction()						//构造函数的实现
{
    ...
    logTransaction();							//记录这笔交易
}
```

derived class:

```C++
class BuyTransaction: public Transaction {
public:
    virtual void logTransaction() const;		//志记此类型交易
    ...
};
class SellTransaction: public Transaction {
public:
    virtual void logTransaction() const;		//志记此类型交易
    ...    
}
```

**当调用derived class时会先调用base class的构造函数，而此时virtual函数不会下降到derived层级**



### 二、 防止编译器报错，但仍会引发错误的方法

```C++
class Transaction {
public:
    Transaction( )								//调用non-virtual函数
    { init(); }
    virtual void logTransaction() const = 0;
    ...
private:
    void init()
    {
        ...
    	logTransaction();						//这里调用virtual函数
    }
};
```

**此方法仍不能避免构造函数调用virtual函数**



### 三、 通过derived class将必要信息传给base class

base class:

```C++
class Transaction {
public:
    explicit Transaction(const std::string& logInfo);
    void logTransaction(const std::string& logInfo) const;	//如今是non-virtual
    ...
};
Transaction::Transaction(const std::string& logInfo)
{
    ...
    logTransaction(logInfo);								//如今是non-virtual调用
}
```

derived class:

```C++
class BuyTransaction: public Transaction{
public:
    BuyTransaction( parameters )							//将log信息传给
        : Transaction(createLogString( parameters ))		//base class的构造函数
    { ... }
    ...
private:
    static std::string createLogString( parameters );
};
```

