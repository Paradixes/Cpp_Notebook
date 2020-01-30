## 条款22：将成员变量声明为`private`

### 一、 使用`private`的原因

### 1. 保持语法一致性

**保证在访问 $class$ 成员时，每一个成员都需要带括号，保证一致性（[同时可见条款18](F:\滔天\文件\学校\大学\专业\C++\C++笔记\4.设计与声明\条款18：让接口被正确使用.md)）**



### 2. 使用函数可以对成员变量有精确的控制

**可以对一个变量进行“禁止访问”、“只读访问”、“读写访问”以及“惟写访问”**

```C++
class AcessLevels {
public:
    ...
    int getReadOnly() const { return readOnly; }
    void setReadWrite(int value) { readWrite = value; }
    int getReadWrite( ) const { return readWrite; }
    void setWriteOnly(int value) { writeOnly = value; }
private:
    int noAcess;			//对此int无任何访问动作
    int readOnly;			//对此int做只读访问
    int readWrite;			//对此int做读写访问
    int WriteOnly;			//对此int做惟写访问
};
```



### 3. 封装

```C++
class SpeedDataCollection {
    ...
public:
    void addValue(int speed);			//添加一笔新数据
    double averageSoFar() const;		//返回平均速度
    ...
};
```

**可以通过封装，可以按照需求替换不同的实现方式（时时计算平均值/被调用时才计算平均值）**

**也可以为“所有可能的实现”提供弹性，在成员变量被读写时轻松通知其他对象，同步验证以及修改控制 $class$**



### 二、 不使用`public`或是`protected`的原因

### 1. 封装的重要性

#### (1) 只有成员函数可以影响到它们，确保 $class$ 的约束条件得到维护

#### (2) 保留日后变更的能力，防止客户码被破坏



### 2. `public`和`private`成员变量

#### (1) 成员变量的封装性与“当其内容改变时可能造成的代码破坏量”成反比（[条款23](条款23：宁以non-member、non-friend替换member函数.md)）

#### (2) 若取消一个`public`成员变量，所有使用它的客户代码都会被破坏

#### (3) 若取消一个 $protected$ 成员变量，所有使用它的 $derived\ classes$ 都会被破坏