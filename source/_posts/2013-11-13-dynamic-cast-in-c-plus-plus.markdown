---
layout: post
title: "Understanding dynamic_cast in c++"
date: 2013-11-13 22:59
comments: true
categories: C++ dev
---
在面向对象程序设计中，有时我们需要在运行时查询一个对象是否能作为某种多态类型使用。与Java的instanceof，以及C#的as、is运算符类似，C++提供了`dynamic_cast`函数用于动态转型。相比C风格的强制类型转换和C++ `reinterpret_cast`，`dynamic_cast`提供了类型安全检查，是一种基于能力查询(Capability Query)的转换，所以在多态类型间进行转换更提倡采用`dynamic_cast`。本文主要介绍`dynamic_cast`的意义，用法和注意事项。
<!--more--> 
基本用法

dynamic_cast可以获取目标对象的引用或指针：

	T1 obj;
	T2* pObj = dynamic_cast<T2*>(&obj);		//转换为T2指针，失败返回NULL
	T2& refObj = dynamic_cast<T2&>(obj);	//转换为T2引用，失败抛出bad_cast异常

 

多态类型

在使用时需要注意：被转换对象`obj`的类型`T1`必须是多态类型，即`T1`必须公有继承自其它类，或者`T1`拥有虚函数（继承或自定义）。若`T1`为非多态类型，使用`dynamic_cast`会报编译错误。下面的例子说明了哪些类属于多态类型，哪些类不是：

	//A为非多态类型 
	class A{
	};

	//B为多态类型
	class B{ 
    	public: virtual ~B(){}
	};

	//D为多态类型
	class D: public A{
	};

	//E为非多态类型
	class E : private A{
	};

	//F为多态类型
	class F : private B{
	};

 

横向转型

在多态类型间转换，分为3种类型：

1.子类向基类的向上转型(Up Cast)

2.基类向子类的向下转型(Down Cast)

3.横向转型(Cross Cast)

向上转型是多态的基础，需不要借助任何特殊的方法，只需用将子类的指针或引用赋给基类的指针或引用即可，当然`dynamic_cast`也支持向上转型，而其总是肯定成功的。而对于向下转型和横向转型来讲，其实对于`dynamic_cast`并没有任何区别，它们都属于能力查询。为了理解方便，我们不妨把`dynamic_cast`视为cross cast：



	class Shape {
    	public: virtual ~Shape();
    	virtual void draw() const = 0;
	};

	class Rollable {
    	public: virtual ~Rollable();
    	virtual void roll() = 0;
	};

	class Circle : public Shape, public Rollable {
    	void draw() const;
    	void roll();
	};

	class Square : public Shape {
    	void draw() const;
	};

	//横向转型失败
	Shape *pShape1 = new Square();
	Rollable *pRollable1 = dynamic_cast<Rollable*>(pShape2);//pRollable为NULL

	//横向转型成功
	Shape *pShape2 = new Circle();
	Rollable *pRollable2 = dynamic_cast<Rollable*>(pShape2);//pRollable不为NULL

指针比较

接上面的例子，在我的机器上`pShape2`和`pRollable2`的值（所指向的地址）分别为：
`pShape2: 0x0039A294`, `pRollable2:0x0039A290`

说明`dynamic_cast`在进行转型的时候对不同多态类型设置了不同的偏移量。接下来的问题是
`pRollable2 == pShape2`