[TOC]

## 一.概述

个人理解，设计模式就是对因需求而产生变化过程中，基于==设计原则==以及某些特定语言的局限性，对变化而进行有规律的封装的一种范式化总结归纳。

而基于所谓的封装变化又可以大体分为三大类。

|                   | 设计模式                                 | 变化点                                        |
| ----------------- | ---------------------------------------- | --------------------------------------------- |
| 创建型            | *Abstract Factory*                       | 产品对象家族                                  |
| *Builder*         | 组合创建组合 *(* 复杂 *)* 对象           |                                               |
| *Factory Method*  | 被实例化的子类 *(* 相关类 *)*            |                                               |
| *Prototype*       | 被实例化的类                             |                                               |
| *Singleton*       | 一个类的唯一实例                         |                                               |
| *Object Factory*  | 被实例化的类， *Factory Method* 的变种   |                                               |
| *Object Pool*     | 对象管理和重用， *Factory Method* 的变种 |                                               |
| *Creation Method* | 类的构造函数， *Factory Method* 的变种   |                                               |
| 结构型            | *Adapter*                                | 对象的接口 *(* 可变 *API)*                    |
| *Bridge*          | 对象实现 *(* 实现逻辑 *)*                |                                               |
| *Composite*       | 对象的结构和组成                         |                                               |
| *Decorator*       | 对象的职责 *(* 职责组合 *)*              |                                               |
| *Façade*          | 子系统的接口 *(* 子系统的变化 *)*        |                                               |
| *Flyweight*       | 对象的存储                               |                                               |
| *Proxy*           | 对象的访问和对象的位置                   |                                               |
| 行为型            | *Chain of Responsibility*                | 满足某个请求的对象 *(* 请求的实际处理对象 *)* |
| *Command*         | 何时、如何实现某个请求                   |                                               |
| *Interpreter*     | 一个语言的文法和解释                     |                                               |
| *Iterator*        | 如何遍历、访问一个聚合的各元素           |                                               |
| *Mediator*        | 对象间的交互逻辑                         |                                               |
| *Memento*         | 对象信息的存储管理                       |                                               |
| *Observer*        | 对象间的依赖和通讯                       |                                               |
| *State*           | 对象状态                                 |                                               |
| *Strategy*        | 算法                                     |                                               |
| *Template Method* | 算法某些步骤                             |                                               |
| *Visitor*         | 作用于对象上的操作                       |                                               |



## 二.设计原则

### 1. 开闭原则

​    对扩展开放，对修改关闭

### 2. 里氏转换原则

子类可以实现父类的抽象方法，但是不能覆盖父类的非抽象方法 。子类中可以增加自己特有的方法

### 3.  依赖倒转原则

引用一个对象，如果这个对象有底层类型，直接引用底层类型

### 4.  接口隔离原则

每一个接口应该是一种角色。如果有一些特有的方法需要多种接口的

### 5.  合成/聚合复用原则

要尽量使用合成和聚合，尽量不要使用继承

> has - a（组合，包含) 和 is a(属于)进行是否要合成聚合还是继承的推断，合成聚合一般的理解新的对象应使用一些已有的对象，使之成为新对象的一部分  

### 6. 迪米特原则

 [最少知识原则详解](ref/最少知识原则.md) 

一个对象应对其他对象有尽可能少的了解
> 优点：遵守 Law of Demeter 将降低模块间的耦合，提升了软件的可维护性和可重用性。
> 缺点：应用 Law of Demeter 可能会导致不得不在类中设计出很多用于中转的包装方法（Wrapper Method），这会提升类设计的复杂度。

## 三.设计模式

### 1．创建型模式

创建型模式，就是==创建对象的模式==，抽象了实例化的过程。它帮助一个系统独立于如何创建、组合和表示它的那些对象。关注的是对象的创建，创建型模式将创建对象的过程进行了抽象，也可以理解为将创建对象的过程进行了封装，作为客户程序仅仅需要去使用对象，而不再关系创建对象过程中的逻辑。

社会化的分工越来越细，自然在软件设计方面也是如此，因此对象的创建和对象的使用分开也就成为了必然趋势。因为对象的创建会消耗掉系统的很多资源，所以单独对对象的创建进行研究，从而能够高效地创建对象就是创建型模式要探讨的问题。这里有6个具体的创建型模式可供研究，它们分别是：

- 简单工厂模式（Simple Factory）
- 工厂方法模式（Factory Method）
- 抽象工厂模式（Abstract Factory）
- 创建者模式（Builder）
- 原型模式（Prototype）解决接口参数的对象再次实例化问题
- 单例模式（Singleton）

> 简单工厂模式不是GoF总结出来的23种设计模式之一

### 2．结构型模式

结构型模式是为解决怎样组装现有的类，设计它们的交互方式，从而达到实现一定的功能目的。结构型模式包容了对很多问题的解决。例如：扩展性（外观、组成、代理、装饰）、封装（适配器、桥接）。

- 外观模式/门面模式（Facade门面模式）

- 适配器模式（Adapter）

  > 这种模式遵循了ISP的接口隔离策略，其中的对象适配器就是实现了委托分离的机制，而类适配器就是一种多继承分离模式
  >
  > 此外还有一种比较特殊的类适配器交缺省适配器

- 代理模式（Proxy）

- 装饰模式（Decorator）

- 桥梁模式/桥接模式（Bridge）

- 组合模式（Composite）

- 享元模式（Flyweight）

### 3．行为型模式

行为型模式涉及到算法和对象间职责的分配，行为模式描述了对象和类的模式，以及它们之间的通信模式，行为模式刻划了在程序运行时难以跟踪的复杂的控制流可分为行为类模式和行为对象模式。

#### 3.1   行为类模式

使用继承机制在类间分派行为。

- 模板方法模式（Template Method）

- 解释器模式（Interpreter）

#### 3.2  行为对象模式

使用对象聚合来分配行为。一些行为对象模式描述了一组对等的对象怎样相互协作以完成其中任何一个对象都无法单独完成的任务。

观察者模式

- 观察者模式（Observer）
- 状态模式（State）
- 策略模式（Strategy）
- 职责链模式（Chain of Responsibility）
- 命令模式（Command）
- 访问者模式（Visitor）
- 调停者模式（Mediator）
- 备忘录模式（[Memento](ref/备忘录模式.md)）
- 迭代器模式（Iterator）




