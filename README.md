# Cpp_Notebook

## [1. 习惯 C++](1.习惯C++)

### [条款01：语言联邦](1.习惯C++/条款01：语言联邦.md)

### [条款02：以编辑器替换预处理器](1.习惯C++/条款02：以编辑器替换预处理器.md)

### [条款03：const 的使用](1.习惯C++/条款03：const的使用.md)

### [条款04：初始化对象](1.习惯C++/条款04：初始化对象.md)



## [2. 构造/析构/赋值运算](2.构造、析构、赋值运算)

### [条款05：C++ 默默编写调用的函数](2.构造、析构、赋值运算/条款05：C++默默编写调用的函数.md)

### [条款06：拒绝编译器自动生成的函数](2.构造、析构、赋值运算/条款06：拒绝编译器自动生成的函数.md)

### [条款07：为多态基类声明 virtual 析构函数](2.构造、析构、赋值运算/条款07：为多态基类声明virtual析构函数.md)

### [条款08：避免析构函数抛出异常](2.构造、析构、赋值运算/条款08：避免析构函数抛出异常.md)

### [条款09：不在析构和构造函数中调用 virtual 函数](2.构造、析构、赋值运算/条款09：不在析构和构造函数中调用virtual函数.md)

### [条款10：令赋值操作返回一个 reference to *this](2.构造、析构、赋值运算/条款10：令赋值操作返回一个reference_to_this.md)

### [条款11：处理自我赋值](2.构造、析构、赋值运算/条款11：处理自我赋值.md)

### [条款12：复制对象的每一个成分](2.构造、析构、赋值运算/条款12：复制对象的每一个成分.md)



## [3. 资源管理](3.资源管理)

### [条款13：以对象管理资源](3.资源管理/条款13：以对象管理资源.md)

### [条款14：在资源管理类中小心 copying 行为](3.资源管理/条款14：在资源管理类中小心copying行为.md)

### [条款15：在资源管理类中提供对原始资源的访问](3.资源管理/条款15：在资源管理类中提供对原始资源的访问.md)

### [条款16：成对使用 new 和 delete 时要采用相同形式](3.资源管理/条款16：成对使用new和delete时要采用相同形式.md)

### [条款17：以独立语句将对象置入智能指针](3.资源管理/条款17：以独立语句将对象置入智能指针.md)



## [4. 设计与声明](4.设计与声明)

### [条款18：让接口被正确使用](4.设计与声明/条款18：让接口被正确使用.md)

### [条款19：像设计 type 一样设计 class](4.设计与声明/条款19：像设计type一样设计class.md)

### [条款20：宁以 pass-by-reference-to-const 替换 pass-by-value](4.设计与声明/条款20：宁以pass-by-reference-to-const替换pass-by-value.md)

### [条款21：必须返回对象时，不要尝试返回 reference](4.设计与声明/条款21：必须返回对象时，不要尝试返回reference.md)

### [条款22：将成员变量声明为 private](4.设计与声明/条款22：将成员变量声明为private.md)

### [条款23：宁以 non-member、non-friend 替换 member 函数](4.设计与声明/条款23：宁以non-member、non-friend替换member函数.md)

### [条款24：若所有参数都需要类型转换，则采用 non-member 函数](4.设计与声明/条款24：若所有参数都需要类型转换，则采用non-member函数.md)

### [条款25：写出不抛出异常的 swap 函数](4.设计与声明/条款25：写出不抛出异常的swap函数.md)



## [5. 实现](5.实现)

### [条款26：尽可能延后变量定义式出现的时间](5.实现/条款26：尽可能延后变量定义式出现的时间.md)

### [条款27：尽量少做转型动作](5.实现/条款27：尽量少做转型动作.md)

### [条款28：避免返回 handles 指向对象内部成员](5.实现/条款28：避免返回handles指向对象内部成员.md)

### [条款29：为“异常安全”而努力](5.实现/条款29：为异常安全而努力.md)

### [条款30：彻底了解 inline](5.实现/条款30：彻底了解inline.md)

### [条款31：降低文件间的编译依存关系](5.实现/条款31：降低文件间的编译依存关系.md)



## [6. 继承与面向对象设计](6.继承与面向对象设计)

### [条款32：确定 public 继承塑模出 is-a 关系](6.继承与面向对象设计/条款32：确定public继承塑模出is-a关系.md)

### [条款33：避免遮掩继承而来的名称](6.继承与面向对象设计/条款33：避免遮掩继承而来的名称.md)

### [条款34：区分接口继承和实现继承](6.继承与面向对象设计/条款34：区分接口继承和实现继承.md)

### [条款35：考虑 virtual 函数以外的选择](6.继承与面向对象设计/条款35：考虑·virtual函数以外的选择.md)

### [条款36：绝不重新定义继承而来的 non-virtual 函数](6.继承与面向对象设计/条款36：绝不重新定义继承而来的non-virtual函数.md)

### [条款37：绝不重新定义继承而来的缺省参数值](6.继承与面向对象设计/条款37：绝不重新定义缺省而来的参数值.md)

### [条款38：通过复合塑模出 has-a 或 ”根据某物实现出“](6.继承与面向对象设计/条款38：通过复合塑模出has-a或根据某物实现出.md)

### [条款39：谨慎使用 private 继承](6.继承与面向对象设计/条款39：谨慎使用private继承.md)

