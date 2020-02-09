# 条款09：不在析构和构造函数中调用 $virtual$ 函数

## 可能出现的非法调用

+ $base\ class$:

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

+ $derived\ class$:

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

+ 调用 $derived\ class$

  ```C++
  BuyTransaction b;
  ```

  + 在 `Transaction` 构造函数被调用前，一定会先调用 $base\ class$ 的构造函数，此时会调用 $base\ class$ 的 `logTransaction` ，而 `logTransaction` 在 $base\ class$ 内是个 $pure\ virtual$ 函数，编译器找不到 `Transaction::logTransaction` 的实现码，会引发错误
  + 析构函数也同样适用于此条款



## 解决方法

### 1. 防止编译器报错，但仍会引发错误

**将 $base\ class$ 的 $pure\ virtual$ 函数隐藏在初始化函数中**

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

+ 此方法仍不能避免构造函数调用 $pure\ virtual$ 函数，一旦编译器发现调用 $pure\ virtual$ 函数，则会中止程序

+ 若调用的是 $virtual$ 函数，编译器会正常执行，但是 $derived\ class$ 会因此调用一个错误的 `logTransaction`
+ 唯一能够避免该问题的做法就是：确定你的构造函数和析构函数没有调用 $virtual$ 函数



### 2. 通过 $derived\ class$ 将必要信息传给 $base\ class$

**在 `class Transaction` 内将 `logTransaction` 函数改为 $non$-$virtual$ ，然后让 $derived\ class$ 传递必要信息给 `Transaction` 构造函数**

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

**`BuyTransaction` 内的 `private static` 函数 `createLogStatic` 比起直接在成员初始列中堆 `Transaction` 赋值要更方便，防止意外指向 “初期未成熟的 `BuyTransaction` 对象内尚未初始化的成员变量”**