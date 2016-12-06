# 编程基础 - Ruby示例

这是一个有着程序示例的知识库，且附带描述当下的编程规则与编程模式。

#### 内容:
  - [线程](#线程)
    - [绿色线程](#绿色线程)
    - [全局解释器锁](#全局解释器锁)
    - [互斥锁](#互斥锁)
    - [纤程](#纤程)
    - [Rails的线程安全](#rails的线程安全)
  - [SOLID原则](#solid原则)
    - [单一职责原则](#单一职责原则)
    - [开放封闭原则](#开放封闭原则)
    - [里氏替换原则](#里氏替换原则)
    - [接口分离原则](#接口分离原则)
    - [依赖倒置原则](#依赖倒置原则)
  - [设计模式](#设计模式)
    - [创建型模式](#创建型模式)
      - [抽象工厂模式](#抽象工厂模式)
      - [生成器模式](#生成器模式)
      - [工厂模式](#工厂模式)
      - [原型模式](#原型模式)
      - [单例模式](#单例模式)
    - [结构型模式](#结构型模式)
      - [适配器模式](#适配器模式)
      - [组合模式](#组合模式)
      - [修饰模式](#修饰模式)
      - [外观模式](#外观模式)
      - [享元模式](#享元模式)
      - [代理模式](#代理模式)
    - [行为型模式](#行为型模式)
      - [责任链模式](#责任链模式)
      - [命令模式](#命令模式)
      - [解释器模式](#解释器模式)
      - [迭代器模式](#迭代器模式)
      - [中介者模式](#中介者模式)
      - [备忘录模式](#备忘录模式)
      - [观察者模式](#观察者模式)
      - [状态模式](#状态模式)
      - [策略模式](#策略模式)
      - [模板方法模式](#模板方法模式)
      - [访问者模式](#访问者模式)
  - [数据结构](#数据结构)
    - [数据结构的基本公理](#数据结构的基本公理)
    - [大O记号](#大o记号)  
    - [实现](#实现)
      - [栈](#栈)
      - [队列](#队列)
      - [双端队列](#双端队列)
      - [单向链表](#单向链表)
      - [双向链表](#双向链表)
      - [有序列表](#有序列表)
      - [哈希表](#哈希表)
      - [二叉树](#二叉树)
      - [二叉查找树](#二叉查找树)
      - [B树](#B树)
      - [二叉堆](#二叉堆)
  - [算法](#算法)
    - [排序](#排序)
      - [冒泡排序](#冒泡排序)
      - [插入排序](#插入排序)
      - [选择排序](#选择排序)
      - [希尔排序](#希尔排序)
      - [堆排序](#堆排序)
      - [归并排序](#归并排序)
      - [快速排序](#快速排序)
    - [搜索](#搜索)
      - [二分查找](#二分查找)
      - [KMP查找](#kmp查找)

## 线程

需注意并行性和并发性：两者主要区别在于操作内存是使用进程还是线程。进一步说，进程会复制一份内存，而线程会分享内存。 这使得进程的产生比线程产生得慢，并且导致了进程一旦运行会消耗更多的资源。总的来说，线程相比进程引发的费用少。这个线程API是一个Ruby的API。我已经暗示了不同的Ruby实现有着不同的潜在线程行为。

#### 绿色线程

Ruby 1.9 用本地线程替换掉了绿色线程。然而，全局解释器锁依然妨碍了并行性。话虽如此，并发已通过更好的调度改进来获得了提升。 新的调度器通过将“上下文切换决策”实质上地弄为单独的本地线程（称为定时器线程），使其更加有效率。

#### 全局解释器锁

MRI has a global interpreter lock (GIL). It's a lock around the execution of Ruby code. This means that in a multi-threaded context, only one thread can execute Ruby code at any one time.

So if you have 8 threads busily working on a 8-core machine, only one thread and one core will be busy at any given time. The GIL exists to protect Ruby internals from race conditions that could corrupt data. There are caveats and optimizations, but this is the gist.

This has some very important implications for MRI. The biggest implication is that Ruby code will never run in parallel on MRI. The GIL prevents it.

[示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/threads/gil.rb)

#### 互斥锁

Mutexes provide a mechanism for multiple threads to synchronize access to a critical portion of code. In other words, they help bring some order, and some guarantees, to the world of multi-threaded chaos.

The name 'mutex' is shorthand for 'mutual exclusion.' If you wrap some section of your code with a mutex, you guarantee that no two threads can enter that section at the same time.

[示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/threads/mutex.rb)

#### 纤程

Fibers are primitives for implementing light weight cooperative concurrency in Ruby. Basically they are a means of creating code blocks that can be paused and resumed, much like threads. The main difference is that they are never preempted and that the scheduling must be done by the programmer and not the VM. As opposed to other stackless light weight concurrency models, each fiber comes with a small 4KB stack. This enables the fiber to be paused from deeply nested function calls within the fiber block.

[示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/threads/fibers.rb)

#### Rails的线程安全

The problem with this is that there is no simple way to say with absolute certainty whether an app as a whole is thread-safe.

* Global variables are global. This means that they are shared between threads. If you weren’t convinced about not using global variables by now, here’s another reason to never touch them. If you really want to share something globally across an app, you are more than likely better served by a constant (but see below), anyway.

* Class variables. For the purpose of a discussion about threads, class variables are not much different from global variables. They are shared across threads just the same way.The problem isn’t so much about using class variables, but about mutating them. And if you are not going to mutate a class variable, in many cases a constant is again a better choice.

* Class instance variables. But maybe you’ve read that you should always use class instance variables instead of class variables in Ruby. Well, maybe you should, but they are just as problematic for threaded programs as class variables.

* Memoization by itself is not a thread safety issue. However, it can easily become one for a couple of reasons:

  - It is often used to store data in class variables or class instance variables (see the previous points).
  - The ||= operator is in fact two operations, so there is a potential context switch happening in the middle of it, causing a race condition between threads.
  - It would be easy to dismiss memoization as the cause of the problem, and tell people just to avoid class variables and class instance variables. However, the issue is more complex than that.
  - In this issue, Evan Phoenix squashes a really tricky race condition bug in the Rails codebase caused by calling super in a memoization function. So even though you would only be using instance variables, you might end up with race conditions with memoization.

  Make sure memoization makes sense and a difference in your case. In many cases Rails actually caches the result anyway, so that you are not saving a whole lot if any resources with your memoization method.
  Don’t memoize to class variables or class instance variables. If you need to memoize something on the class level, use thread local variables (Thread.current[:baz]) instead. Be aware, though, that it is still kind of a global variable. So while it’s thread-safe, it still might not be good coding practice.

[示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/threads/rails.rb)

#### 代码及文献资源:

* [http://www.jstorimer.com/blogs/workingwithcode/8085491-nobody-understands-the-gil](http://www.jstorimer.com/blogs/workingwithcode/8085491-nobody-understands-the-gil)

* [https://en.wikipedia.org/wiki/Global_interpreter_lock](https://en.wikipedia.org/wiki/Global_interpreter_lock)

* [http://www.csinaction.com/2014/10/10/multithreading-in-the-mri-ruby-interpreter/](http://www.csinaction.com/2014/10/10/multithreading-in-the-mri-ruby-interpreter/)

## SOLID原则

In computer programming, SOLID (single responsibility, open-closed, Liskov substitution, interface segregation and dependency inversion) is a mnemonic acronym introduced by Michael Feathers for the "first five principles" named by Robert C. Martin in the early 2000s that stands for five basic principles of object-oriented programming and design. The intention is that these principles, when applied together, will make it more likely that a programmer will create a system that is easy to maintain and extend over time. The principles of SOLID are guidelines that can be applied while working on software to remove code smells by causing the programmer to refactor the software's source code until it is both legible and extensible. It is part of an overall strategy of agile and Adaptive Software Development.

#### 单一职责原则

A class should have only a single responsibility.

Every class should have a single responsibility, and that responsibility should be entirely encapsulated.
All its services should be narrowly aligned with that responsibility, this embrace the high cohesion.

[示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/solid/single_responsibility.rb)

#### 开放封闭原则

Software entities should be open for extension, but closed for modification.
That is, such an entity can allow its behaviour to be extended without modifying its source code.

[示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/solid/open_close.rb)

#### 里氏替换原则
Objects in a program should be replaceable with instances of their subtypes without altering the correctness of that program.

It states that, in a computer program, if S is a subtype of T, then objects of type T may be replaced with objects of type S (i.e., objects of type S may substitute objects of type T) without altering any of the desirable properties of that program (correctness, task performed, etc.)

[示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/solid/liskov_substitution.rb)

#### 接口分离原则

Many client-specific interfaces are better than one general-purpose interface.

States that no client should be forced to depend on methods it does not use.
ISP splits interfaces which are very large into smaller and more specific ones so that clients will only have to know about the methods that are of interest to them. Such shrunken interfaces are also called role interfaces. 
ISP is intended to keep a system decoupled and thus easier to refactor, change, and redeploy.

[示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/solid/interface_segregation.rb)

####依赖倒置原则

One should Depend upon Abstractions, Do not depend upon concretions.

Refers to a specific form of decoupling software modules. When following this principle, the conventional dependency relationships established from high-level, policy-setting modules to low-level, dependency modules are inverted (i.e. reversed), thus rendering high-level modules independent of the low-level module implementation details. 

####代码及文献资源:

* [https://gist.github.com/khusnetdinov/9d8f50fdcaab197871b31578f2e14d5d](https://gist.github.com/khusnetdinov/9d8f50fdcaab197871b31578f2e14d5d) 

* [https://robots.thoughtbot.com/back-to-basics-solid](https://robots.thoughtbot.com/back-to-basics-solid) 

* [https://subvisual.co/blog/posts/19-solid-principles-in-ruby](https://subvisual.co/blog/posts/19-solid-principles-in-ruby) 

* [http://blog.siyelo.com/solid-principles-in-ruby/](http://blog.siyelo.com/solid-principles-in-ruby/)

## 设计模式

### 创建型模式

In software engineering, creational design patterns are design patterns that deal with object creation mechanisms, trying to create objects in a manner suitable to the situation. The basic form of object creation could result in design problems or in added complexity to the design. Creational design patterns solve this problem by somehow controlling this object creation. Creational design patterns are composed of two dominant ideas. One is encapsulating knowledge about which concrete classes the system uses. Another is hiding how instances of these concrete classes are created and combined.

[维基百科](https://en.wikipedia.org/wiki/Creational_pattern)

#### 抽象工厂模式

The abstract factory pattern provides a way to encapsulate a group of individual factories that have a common theme without specifying their concrete classes. In normal usage, the client software creates a concrete implementation of the abstract factory and then uses the generic interface of the factory to create the concrete objects that are part of the theme. The client doesn't know (or care) which concrete objects it gets from each of these internal factories, since it uses only the generic interfaces of their products. This pattern separates the details of implementation of a set of objects from their general usage and relies on object composition, as object creation is implemented in methods exposed in the factory interface.

[示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/patterns/creational/abstract_factory.rb) | [维基百科](https://en.wikipedia.org/wiki/Abstract_factory_pattern)

#### 生成器模式

The builder pattern is an object creation software design pattern. Unlike the abstract factory pattern and the factory method pattern whose intention is to enable polymorphism, the intention of the builder pattern is to find a solution to the telescoping constructor anti-pattern[citation needed]. The telescoping constructor anti-pattern occurs when the increase of object constructor parameter combination leads to an exponential list of constructors. Instead of using numerous constructors, the builder pattern uses another object, a builder, that receives each initialization parameter step by step and then returns the resulting constructed object at once.

[示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/patterns/creational/builder.rb) | [维基百科](https://en.wikipedia.org/wiki/Builder_pattern)

#### 工厂模式

In class-based programming, the factory method pattern is a creational pattern that uses factory methods to deal with the problem of creating objects without having to specify the exact class of the object that will be created. This is done by creating objects by calling a factory method—either specified in an interface and implemented by child classes, or implemented in a base class and optionally overridden by derived classes—rather than by calling a constructor.

[示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/patterns/creational/factory.rb) | [维基百科](https://en.wikipedia.org/wiki/Factory_method_pattern)

#### 原型模式

The prototype pattern is a creational design pattern in software development. It is used when the type of objects to create is determined by a prototypical instance, which is cloned to produce new objects. This pattern is used to:

* avoid subclasses of an object creator in the client application, like the abstract factory pattern does.
* avoid the inherent cost of creating a new object in the standard way (e.g., using the 'new' keyword) when it is prohibitively expensive for a given application.

[示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/patterns/creational/prototype.rb) | [维基百科](https://en.wikipedia.org/wiki/Prototype_pattern)

#### 单例模式

Ensure a class only has one instance, and provide a global point of access to it. This is useful when exactly one object is needed to coordinate actions across the system. The concept is sometimes generalized to systems that operate more efficiently when only one object exists, or that restrict the instantiation to a certain number of objects.

[示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/patterns/creational/singleton.rb) | [维基百科](https://en.wikipedia.org/wiki/Singleton_pattern)

#### 其它模式:

* [惰性初始模式](https://en.wikipedia.org/wiki/Lazy_initialization)

* [多例模式](https://en.wikipedia.org/wiki/Multiton_pattern)

* [对象池](https://en.wikipedia.org/wiki/Object_pool_pattern)

* [RAII](https://en.wikipedia.org/wiki/Resource_Acquisition_Is_Initialization)

* [效用模式](https://en.wikipedia.org/wiki/Utility_pattern)

### 结构型模式

In software engineering, structural design patterns are design patterns that ease the design by identifying a simple way to realize relationships between entities.

[维基百科](https://en.wikipedia.org/wiki/Structural_pattern)

#### 适配器模式

In software engineering, the adapter pattern is a software design pattern that allows the interface of an existing class to be used as another interface. It is often used to make existing classes work with others without modifying their source code.

[示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/patterns/structural/adapter.rb) | [维基百科](https://en.wikipedia.org/wiki/Adapter_pattern)

#### 组合模式

The composite design pattern is a structural pattern used to represent objects that have a hierarchical tree structure. It allows for the uniform treatment of both individual leaf nodes and of branches composed of many nodes.

[示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/patterns/structural/composite.rb) | [维基百科](https://en.wikipedia.org/wiki/Composite_pattern)

#### 修饰模式

In object-oriented programming, the decorator pattern (also known as Wrapper, an alternative naming shared with the Adapter pattern) is a design pattern that allows behavior to be added to an individual object, either statically or dynamically, without affecting the behavior of other objects from the same class. The decorator pattern is often useful for adhering to the Single Responsibility Principle, as it allows functionality to be divided between classes with unique areas of concern.

[示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/patterns/structural/decorator.rb) | [维基百科](https://en.wikipedia.org/wiki/Decorator_pattern)

#### 外观模式

The Facade design pattern is often used when a system is very complex or difficult to understand because the system has a large number of interdependent classes or its source code is unavailable. This pattern hides the complexities of the larger system and provides a simpler interface to the client. It typically involves a single wrapper class which contains a set of members required by client. These members access the system on behalf of the facade client and hide the implementation details.

[示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/patterns/structural/facade.rb) | [维基百科](https://en.wikipedia.org/wiki/Facade_pattern)

#### 享元模式

In computer programming, flyweight is a software design pattern. A flyweight is an object that minimizes memory use by sharing as much data as possible with other similar objects; it is a way to use objects in large numbers when a simple repeated representation would use an unacceptable amount of memory. Often some parts of the object state can be shared, and it is common practice to hold them in external data structures and pass them to the flyweight objects temporarily when they are used.

[示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/patterns/structural/flyweight.rb) | [维基百科](https://en.wikipedia.org/wiki/Flyweight_pattern)

#### 代理模式

A proxy, in its most general form, is a class functioning as an interface to something else. The proxy could interface to anything: a network connection, a large object in memory, a file, or some other resource that is expensive or impossible to duplicate. In short, a proxy is a wrapper or agent object that is being called by the client to access the real serving object behind the scenes. Use of the proxy can simply be forwarding to the real object, or can provide additional logic. In the proxy extra functionality can be provided, for example caching when operations on the real object are resource intensive, or checking preconditions before operations on the real object are invoked. For the client, usage of a proxy object is similar to using the real object, because both implement the same interface.

[示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/patterns/structural/proxy.rb) | [维基百科](https://en.wikipedia.org/wiki/Proxy_pattern)

#### 其它模式:

* [注释回调](http://c2.com/cgi/wiki?AnnotatedCallback)

* [桥接模式](https://en.wikipedia.org/wiki/Bridge_pattern)

* [数据总线模式](http://c2.com/cgi/wiki?DataBusPattern)

* [角色对象模式](http://c2.com/cgi/wiki?RoleObjectPattern)

### 行为型模式 

In software engineering, behavioral design patterns are design patterns that identify common communication patterns between objects and realize these patterns. By doing so, these patterns increase flexibility in carrying out this communication.

[维基百科](https://en.wikipedia.org/wiki/Behavioral_pattern)

#### 责任链模式

In object-oriented design, the chain-of-responsibility pattern is a design pattern consisting of a source of command objects and a series of processing objects. Each processing object contains logic that defines the types of command objects that it can handle; the rest are passed to the next processing object in the chain. A mechanism also exists for adding new processing objects to the end of this chain.

[示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/patterns/behavioral/chain_of_responsibility.rb) | [维基百科](https://en.wikipedia.org/wiki/Chain-of-responsibility_pattern)

#### 命令模式

Encapsulate a request as an object, thereby allowing for the parameterization of clients with different requests, and the queuing or logging of requests. It also allows for the support of undoable operations.

[示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/patterns/behavioral/command.rb) | [维基百科](https://en.wikipedia.org/wiki/Command_pattern)

#### 解释器模式

In computer programming, the interpreter pattern is a design pattern that specifies how to evaluate sentences in a language. The basic idea is to have a class for each symbol (terminal or nonterminal) in a specialized computer language. The syntax tree of a sentence in the language is an instance of the composite pattern and is used to evaluate (interpret) the sentence for a client.

[示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/patterns/behavioral/interpreter.rb) | [维基百科](https://en.wikipedia.org/wiki/Interpreter_pattern)

#### 迭代器模式

The iterator design pattern provides sequential access to elements within a container without exposing how the container actually represents the elements. The iterator can be thought of as a moveable pointer that allows access to elements encapsulated within a container.

* External iterator: The iteration logic is contained in a separate class. The iteration class can be generalized to handle multiple object types as long as they allow indexing. It require the additional class to do the actual iterating, but they do allow for greater flexibility because you can control the iteration, which elements are iterated over and in what order.

* Internal iterator: all the iterating logic occurs inside the aggregate object. Use a code block to pass your logic into the aggregate which then calls the block for each of it's elements.

[示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/patterns/behavioral/iterator.rb) | [维基百科](https://en.wikipedia.org/wiki/Iterator_pattern)

#### 中介者模式

Usually a program is made up of a large number of classes. So the logic and computation is distributed among these classes. However, as more classes are developed in a program, especially during maintenance and/or refactoring, the problem of communication between these classes may become more complex. This makes the program harder to read and maintain. Furthermore, it can become difficult to change the program, since any change may affect code in several other classes. With the mediator pattern, communication between objects is encapsulated with a mediator object. Objects no longer communicate directly with each other, but instead communicate through the mediator. This reduces the dependencies between communicating objects, thereby lowering the coupling.

[示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/patterns/behavioral/mediator.rb) | [维基百科](https://en.wikipedia.org/wiki/Mediator_pattern)

#### 备忘录模式

The memento pattern is implemented with three objects: the originator, a caretaker and a memento. The originator is some object that has an internal state. The caretaker is going to do something to the originator, but wants to be able to undo the change. The caretaker first asks the originator for a memento object. Then it does whatever operation (or sequence of operations) it was going to do. To roll back to the state before the operations, it returns the memento object to the originator. The memento object itself is an opaque object (one which the caretaker cannot, or should not, change). When using this pattern, care should be taken if the originator may change other objects or resources - the memento pattern operates on a single object.

[示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/patterns/behavioral/momento.rb) | [维基百科](https://en.wikipedia.org/wiki/Memento_pattern)

#### 观察者模式

The observer pattern is a software design pattern in which an object, called the subject, maintains a list of its dependents, called observers, and notifies them automatically of any state changes, usually by calling one of their methods. It is mainly used to implement distributed event handling systems. The Observer pattern is also a key part in the familiar model–view–controller (MVC) architectural pattern. The observer pattern is implemented in numerous programming libraries and systems, including almost all GUI toolkits.

[示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/patterns/behavioral/observer.rb) | [维基百科](https://en.wikipedia.org/wiki/Observer_pattern)

#### 状态模式

The state pattern is a behavioral software design pattern that implements a state machine in an object-oriented way. With the state pattern, a state machine is implemented by implementing each individual state as a derived class of the state pattern interface, and implementing state transitions by invoking methods defined by the pattern's superclass.

[示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/patterns/behavioral/state.rb) | [维基百科](https://en.wikipedia.org/wiki/State_pattern)

#### 策略模式

In computer programming, the strategy pattern (also known as the policy pattern) is a software design pattern that enables an algorithm's behavior to be selected at runtime. The strategy pattern

* defines a family of algorithms,
* encapsulates each algorithm, and
* makes the algorithms interchangeable within that family.

Strategy lets the algorithm vary independently from clients that use it. Strategy is one of the patterns included in the influential book Design Patterns by Gamma et al. that popularized the concept of using patterns to describe software design.

For instance, a class that performs validation on incoming data may use a strategy pattern to select a validation algorithm based on the type of data, the source of the data, user choice, or other discriminating factors. These factors are not known for each case until run-time, and may require radically different validation to be performed. The validation strategies, encapsulated separately from the validating object, may be used by other validating objects in different areas of the system (or even different systems) without code duplication.

[示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/patterns/behavioral/strategy.rb) | [维基百科](https://en.wikipedia.org/wiki/Strategy_pattern)

#### 模板方法模式

In object-oriented programming, first a class is created that provides the basic steps of an algorithm design. These steps are implemented using abstract methods. Later on, subclasses change the abstract methods to implement real actions. Thus the general algorithm is saved in one place but the concrete steps may be changed by the subclasses.

[示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/patterns/behavioral/template_method.rb) | [维基百科](https://en.wikipedia.org/wiki/Template_method_pattern)

#### 访问者模式

n object-oriented programming and software engineering, the visitor design pattern is a way of separating an algorithm from an object structure on which it operates. A practical result of this separation is the ability to add new operations to existing object structures without modifying those structures. It is one way to follow the open/closed principle.

In essence, the visitor allows one to add new virtual functions to a family of classes without modifying the classes themselves; instead, one creates a visitor class that implements all of the appropriate specializations of the virtual function. The visitor takes the instance reference as input, and implements the goal through double dispatch.

[示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/patterns/behavioral/visitor.rb) | [维基百科](https://en.wikipedia.org/wiki/Visitor_pattern)

#### 其它模式:

* [访问者模式](http://c2.com/cgi/wiki?HierarchicalVisitorPattern)

#### 代码及文献资源:

* [https://ru.wikipedia.org/wiki/Design_Patterns](https://ru.wikipedia.org/wiki/Design_Patterns)

* [https://en.wikipedia.org/wiki/Creational_pattern](https://en.wikipedia.org/wiki/Creational_pattern)

* [https://en.wikipedia.org/wiki/Structural_pattern](https://en.wikipedia.org/wiki/Structural_pattern)

* [https://en.wikipedia.org/wiki/Behavioral_pattern](https://en.wikipedia.org/wiki/Behavioral_pattern)

* [http://c2.com/cgi/wiki?CategoryPattern](http://c2.com/cgi/wiki?CategoryPattern)

* [https://gist.github.com/martindemello/7231bf0f407ca428b509](https://gist.github.com/martindemello/7231bf0f407ca428b509)

## 数据结构 

In computer science, big O notation is used to classify algorithms by how they respond to changes in input size, such as how the processing time of an algorithm changes as the problem size becomes extremely large. In analytic number theory it is used to estimate the "error committed" while replacing the asymptotic size of an arithmetical function by the value it takes at a large finite argument. A famous example is the problem of estimating the remainder term in the prime number theorem.

### 数据结构的基本公理

The running time performance of the common language runtime is given by a set of axioms which we shall now postulate.

[示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/structures/axioms.rb)

#### 代码及文献资源:

* [Axioms](http://www.brpreiss.com/books/opus8/html/page37.html)

### 大O记号

* [Conventions for Writing Big Oh Expressions](http://www.brpreiss.com/books/opus8/html/page66.html)

* [Rules For Big Oh Analysis of Running Time](http://www.brpreiss.com/books/opus8/html/page72.html)

* [Reality Check](http://www.brpreiss.com/books/opus8/html/page76.html)

* [Example: Ruby method with Big O explanation](http://www.brpreiss.com/books/opus8/html/page71.html)

* [Example: Example-Prefix Sums](http://www.brpreiss.com/books/opus8/html/page73.html)

* [Example: Bucket Sort](http://www.brpreiss.com/books/opus8/html/page75.html)

### 实现 

> NOTE: All data structures are given as example for learning purpose. For using in project please refer to other resources. Most examples were taken from [Here](http://www.brpreiss.com/) witch is proper Bruno R. Preiss.

#### 栈

The stack is the sibling of the queue. It mimicks a real-life stack (e.g. of paper). It is FILO (first-in-last-out), so that when items are retrieved from the stack, they are returned in the reverse of the order in which they were added. Again, Ruby Arrays provide a perfect container. As with the Queue, it could also be implemented using a linked list.

| 数据结构 | 访问平均 | 查找平均 | 插入平均 | 删除平均 | 访问最坏 | 查找最坏 | 插入最坏 | 删除最坏 |
|-----------|---------------:|---------------:|------------------:|-----------------:|-------------:|-------------:|----------------:|---------------:|
[栈](http://en.wikipedia.org/wiki/Stack_(abstract_data_type)) | `Θ(n)` | `Θ(n)` | `Θ(1)` | `Θ(1)` | `O(n)` | `O(n)` | `O(1)` | `O(1)` |

[栈(数组) - 示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/structures/stack_as_array.rb) | [实现步骤](http://www.brpreiss.com/books/opus8/html/page131.html)

[栈(链表) - 示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/structures/stack_as_linked_list.rb) | [实现步骤](http://www.brpreiss.com/books/opus8/html/page137.html)

#### 队列

A queue is a simple container-based structure that mimics a real-life queue (e.g. waiting in line at the bank). It is FIFO (first-in-first-out), meaning that when you retrieve items from the queue, they are returned in the order in which they entered. Ruby Arrays provide methods that make Queue implementation trivially easy, but having them named appropriately and contained in a convenience class is worth it to see how they are implemented, and because other structures will inherit from this one. An alternate implementation could be done using a linked list.

| 数据结构 | 访问平均 | 查找平均 | 插入平均 | 删除平均 | 访问最坏 | 查找最坏 | 插入最坏 | 删除最坏 |
|-----------|---------------:|---------------:|------------------:|-----------------:|-------------:|-------------:|----------------:|---------------:|
[队列](http://en.wikipedia.org/wiki/Queue_(abstract_data_type)) | `Θ(n)` | `Θ(n)` | `Θ(1)` | `Θ(1)` | `O(n)` | `O(n)` | `O(1)` | `O(1)` |

[队列(数组) - 示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/structures/queue_as_array.rb) | [实现步骤](http://www.brpreiss.com/books/opus8/html/page147.html)

[队列(链表) - 示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/structures/queue_as_linked_list.rb) | [实现步骤](http://www.brpreiss.com/books/opus8/html/page151.html)

#### 双端队列

A Deque is a queue which allows adding and removing items at both ends.

| 数据结构 | 访问平均 | 查找平均 | 插入平均 | 删除平均 | 访问最坏 | 查找最坏 | 插入最坏 | 删除最坏 |
|-----------|---------------:|---------------:|------------------:|-----------------:|-------------:|-------------:|----------------:|---------------:|
[双端队列](https://en.wikipedia.org/wiki/Double-ended_queue) | `Θ(n)` | `Θ(n)` | `Θ(1)` | `Θ(1)` | `O(n)` | `O(n)` | `O(1)` | `O(1)` |

[双端队列(数组) - 示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/structures/deque_as_array.rb) | [实现步骤](http://www.brpreiss.com/books/opus8/html/page158.html)

[双端队列(链表) - 示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/structures/deque_as_linked_list.rb) | [实现步骤](http://www.brpreiss.com/books/opus8/html/page161.html)

#### 单向链表

Singly linked lists contain nodes which have a data field as well as a 'next' field, which points to the next node in line of nodes. Operations that can be performed on singly linked lists include insertion, deletion and traversal.

| 数据结构 | 访问平均 | 查找平均 | 插入平均 | 删除平均 | 访问最坏 | 查找最坏 | 插入最坏 | 删除最坏 |
|-----------|---------------:|---------------:|------------------:|-----------------:|-------------:|-------------:|----------------:|---------------:|
[单向链表](http://en.wikipedia.org/wiki/Singly_linked_list#Singly_linked_lists) | `Θ(n)` | `Θ(n)` | `Θ(1)` | `Θ(1)` | `O(n)` | `O(n)` | `O(1)` | `O(1)` |

[示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/structures/singly_linked_list.rb) | [实现步骤](http://www.brpreiss.com/books/opus8/html/page96.html#SECTION004300000000000000000) | [维基百科](https://en.wikipedia.org/wiki/Linked_list#Singly_linked_list) 

#### 双向链表

In a doubly-linked list , each list element contains two references--one to its successor and one to its predecessor. 

| 数据结构 | 访问平均 | 查找平均 | 插入平均 | 删除平均 | 访问最坏 | 查找最坏 | 插入最坏 | 删除最坏 |
|-----------|---------------:|---------------:|------------------:|-----------------:|-------------:|-------------:|----------------:|---------------:|
[双向链表](http://en.wikipedia.org/wiki/Doubly_linked_list) | `Θ(n)` | `Θ(n)` | `Θ(1)` | `Θ(1)` | `O(n)` | `O(n)` | `O(1)` | `O(1)` | 

[示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/structures/double_linked_list.rb)

#### 有序列表
 
An ordered list is a list in which the order of the items is significant. However, the items in an ordered lists are not necessarily sorted. Consequently, it is possible to change the order of items and still have a valid ordered list.

| 数据结构 | 访问平均 | 查找平均 | 插入平均 | 删除平均 | 访问最坏 | 查找最坏 | 插入最坏 | 删除最坏 |
|-----------|---------------:|---------------:|------------------:|-----------------:|-------------:|-------------:|----------------:|---------------:|
有序列表(数组) |`Θ(1)` | `Θ(n)` | `Θ(1)` | `Θ(1)` | `O(1)` | `O(n)` | `O(1)` | `O(1)` |
有序列表(链表) |`Θ(n)` | `Θ(n)` | `Θ(1)` | `Θ(1)` | `O(n)` | `O(n)` | `O(1)` | `O(1)` |

[有序列表(数组) - 示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/structures/ordered_list_as_array.rb) | [实现步骤](http://www.brpreiss.com/books/opus8/html/page169.html)

[有序列表(链表) - 示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/structures/ordered_as_linked_list.rb) | [实现步骤](http://www.brpreiss.com/books/opus8/html/page178.html)

#### 哈希表

A hash table  is a searchable container. As such, it provides methods for putting an object into the container, finding an object in the container, and removing an object from the container.

| 数据结构 | 访问平均 | 查找平均 | 插入平均 | 删除平均 | 访问最坏 | 查找最坏 | 插入最坏 | 删除最坏 |
|-----------|---------------:|---------------:|------------------:|-----------------:|-------------:|-------------:|----------------:|---------------:|
[哈希表](http://en.wikipedia.org/wiki/Hash_table) | `Θ(n)` | `Θ(1)` | `Θ(1)` | `Θ(1)` | `Θ(n)` | `O(n)` | `O(n)` | `O(n)` |

[示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/structures/hash_table.rb) | [实现步骤](http://www.brpreiss.com/books/opus8/html/page222.html)

#### 二叉树

A binary tree is a tree in which each node can have a maximum of two children. The children are designated left and right. 

| 数据结构 | 访问平均 | 查找平均 | 插入平均 | 删除平均 | 访问最坏 | 查找最坏 | 插入最坏 | 删除最坏 |
|-----------|---------------:|---------------:|------------------:|-----------------:|-------------:|-------------:|----------------:|---------------:|
[二叉树](https://en.wikipedia.org/wiki/Binary_tree) | `Θ(log(n))` | `Θ(log(n))` | `Θ(log(n))` | `Θ(log(n))` | `O(n)` | `O(n)` | `O(n)` | `O(n)` |

[示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/structures/binary_tree.rb) | [实现步骤](http://www.brpreiss.com/books/opus8/html/page286.html)

#### 二叉查找树

In computer science, binary search trees (BST), sometimes called ordered or sorted binary trees, are a particular type of containers: data structures that store "items" (such as numbers, names etc.) in memory. They allow fast lookup, addition and removal of items, and can be used to implement either dynamic sets of items, or lookup tables that allow finding an item by its key (e.g., finding the phone number of a person by name)

| 数据结构 | 访问平均 | 查找平均 | 插入平均 | 删除平均 | 访问最坏 | 查找最坏 | 插入最坏 | 删除最坏 |
|-----------|---------------:|---------------:|------------------:|-----------------:|-------------:|-------------:|----------------:|---------------:|
[二叉查找树](http://en.wikipedia.org/wiki/Binary_search_tree) | `Θ(log(n))` | `Θ(log(n))` | `Θ(log(n))` | `Θ(log(n))` | `O(n)` | `O(n)` | `O(n)` | `O(n)` |

[示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/structures/binary_search_tree.rb) | [实现步骤](http://www.brpreiss.com/books/opus8/html/page300.html)

#### B树

In computer science, a B-tree is a self-balancing tree data structure that keeps data sorted and allows searches, sequential access, insertions, and deletions in logarithmic time. The B-tree is a generalization of a binary search tree in that a node can have more than two children (Comer . Unlike self-balancing binary search trees, the B-tree is optimized for systems that read and write large blocks of data. B-trees are a good example of a data structure for external memory. It is commonly used in databases and filesystems.

| 数据结构 | 访问平均 | 查找平均 | 插入平均 | 删除平均 | 访问最坏 | 查找最坏 | 插入最坏 | 删除最坏 |
|-----------|---------------:|---------------:|------------------:|-----------------:|-------------:|-------------:|----------------:|---------------:|
[B树](http://en.wikipedia.org/wiki/B_tree) | `Θ(log(n))` | `Θ(log(n))` | `Θ(log(n))` | `O(log(n))` | `O(log(n))` | `O(log(n))` | `O(log(n))` | `O(log(n))` | 

[示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/structures/b_tree.rb) | [实现步骤](http://www.brpreiss.com/books/opus8/html/page339.html)

#### 二叉堆

A binary heap is a heap-ordered complete binary tree which is implemented using an array. In a heap the smallest key is found at the root and since the root is always found in the first position of the array, finding the smallest key is a trivial operation in a binary heap.

| 数据结构 | 访问平均 | 查找平均 | 插入平均 | 删除平均 | 访问最坏 | 查找最坏 | 插入最坏 | 删除最坏 |
|-----------|---------------:|---------------:|------------------:|-----------------:|-------------:|-------------:|----------------:|---------------:|
[二叉堆](https://en.wikipedia.org/wiki/Binary_heap) | `Θ(n)` | `Θ(n)` | `Θ(1)` | `O(log(n))` | `O(n)` | `O(n))` | `O(log(n))` | `O(log(n))` | 

[示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/structures/binary_heap.rb) | [实现步骤] (http://www.brpreiss.com/books/opus8/html/page355.html)

#### 代码及文献资源:

* [https://github.com/blahah/datastructures](https://github.com/blahah/datastructures)

* [http://www.brpreiss.com](http://www.brpreiss.com)

* [https://en.wikipedia.org/wiki/List_of_data_structures](https://en.wikipedia.org/wiki/List_of_data_structures)

* [https://www.cs.cmu.edu/~adamchik/15-121/lectures/Trees/trees.html](https://www.cs.cmu.edu/~adamchik/15-121/lectures/Trees/trees.html)

* [http://bigocheatsheet.com/](http://bigocheatsheet.com/)

* [https://gist.github.com/TSiege/cbb0507082bb18ff7e4b](https://gist.github.com/TSiege/cbb0507082bb18ff7e4b)

## 算法

### 排序

A sorting algorithm is an algorithm that puts elements of a list in a certain order. The most-used orders are numerical order and lexicographical order. Efficient sorting is important for optimizing the use of other algorithms (such as search and merge algorithms) which require input data to be in sorted lists; it is also often useful for canonicalizing data and for producing human-readable output.

#### 冒泡排序

Bubble sort has many of the same properties as insertion sort, but has slightly higher overhead. In the case of nearly sorted data, bubble sort takes O(n) time, but requires at least 2 passes through the data (whereas insertion sort requires something more like 1 pass).

| 最好 | 平均 | 最坏 |
|-----:|--------:|------:|
| `Ω(n)` | `Θ(n^2)` | `O(n^2)` |

[示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/algorithms/sort/bubble.rb) | [维基百科](https://en.wikipedia.org/wiki/Bubble_sort)

#### 插入排序

Although it is one of the elementary sorting algorithms with O(n2) worst-case time, insertion sort is the algorithm of choice either when the data is nearly sorted (because it is adaptive) or when the problem size is small (because it has low overhead).
For these reasons, and because it is also stable, insertion sort is often used as the recursive base case (when the problem size is small) for higher overhead divide-and-conquer sorting algorithms, such as merge sort or quick sort

| 最好 | 平均 | 最坏 |
|-----:|--------:|------:|
| `Ω(n)` | `Θ(n^2)` | `O(n^2)` |

[示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/algorithms/sort/insertion.rb) | [维基百科](https://en.wikipedia.org/wiki/Insertion_sort)

#### 选择排序

From the comparions presented here, one might conclude that selection sort should never be used. It does not adapt to the data in any way (notice that the four animations above run in lock step), so its runtime is always quadratic.
However, selection sort has the property of minimizing the number of swaps. In applications where the cost of swapping items is high, selection sort very well may be the algorithm of choice.

| 最好 | 平均 | 最坏 |
|-----:|--------:|------:|
| `Ω(n^2)` | `Θ(n^2)` | `O(n^2)` |

[示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/algorithms/sort/selection.rb) | [维基百科](https://en.wikipedia.org/wiki/Selection_sort)

#### 希尔排序

The worse-case time complexity of shell sort depends on the increment sequence. For the increments 1 4 13 40 121…, which is what is used here, the time complexity is O(n3/2). For other increments, time complexity is known to be O(n4/3) and even O(n·lg2(n)). Neither tight upper bounds on time complexity nor the best increment sequence are known.
Because shell sort is based on insertion sort, shell sort inherits insertion sort’s adaptive properties. The adapation is not as dramatic because shell sort requires one pass through the data for each increment, but it is significant. For the increment sequence shown above, there are log3(n) increments, so the time complexity for nearly sorted data is O(n·log3(n)).
Because of its low overhead, relatively simple implementation, adaptive properties, and sub-quadratic time complexity, shell sort may be a viable alternative to the O(n·lg(n)) sorting algorithms for some applications when the data to be sorted is not very large.

| 最好 | 平均 | 最坏 |
|-----:|--------:|------:|
| `Ω(n log(n))` | `Θ(n(log(n))^2)` | `O(n(log(n))^2)` |

[示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/algorithms/sort/shell.rb) | [维基百科](https://en.wikipedia.org/wiki/Shellsort)

#### 堆排序

Heap sort is simple to implement, performs an O(n·lg(n)) in-place sort, but is not stable.
The first loop, the Θ(n) “heapify” phase, puts the array into heap order. The second loop, the O(n·lg(n)) “sortdown” phase, repeatedly extracts the maximum and restores heap order.
The sink function is written recursively for clarity. Thus, as shown, the code requires Θ(lg(n)) space for the recursive call stack. However, the tail recursion in sink() is easily converted to iteration, which yields the O(1) space bound.
Both phases are slightly adaptive, though not in any particularly useful manner. In the nearly sorted case, the heapify phase destroys the original order. In the reversed case, the heapify phase is as fast as possible since the array starts in heap order, but then the sortdown phase is typical. In the few unique keys case, there is some speedup but not as much as in shell sort or 3-way quicksort.

| 最好 | 平均 | 最坏 |
|-----:|--------:|------:|
| `Ω(n log(n))` | `Θ(n log(n))` | `O(n log(n))` |

[示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/algorithms/sort/heap.rb) | [维基百科](https://en.wikipedia.org/wiki/Heapsort)

#### 归并排序

Merge sort is very predictable. It makes between 0.5lg(n) and lg(n) comparisons per element, and between lg(n) and 1.5lg(n) swaps per element. The minima are achieved for already sorted data; the maxima are achieved, on average, for random data. If using Θ(n) extra space is of no concern, then merge sort is an excellent choice: It is simple to implement, and it is the only stable O(n·lg(n)) sorting algorithm. Note that when sorting linked lists, merge sort requires only Θ(lg(n)) extra space (for recursion).
Merge sort is the algorithm of choice for a variety of situations: when stability is required, when sorting linked lists, and when random access is much more expensive than sequential access (for example, external sorting on tape).
There do exist linear time in-place merge algorithms for the last step of the algorithm, but they are both expensive and complex. The complexity is justified for applications such as external sorting when Θ(n) extra space is not available.

| 最好 | 平均 | 最坏 |
|-----:|--------:|------:|
| `Ω(n log(n))` | `Θ(n log(n))` | `O(n log(n))` | 

[示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/algorithms/sort/merge.rb) | [维基百科](https://en.wikipedia.org/wiki/Merge_sort)

#### 快速排序

When carefully implemented, quick sort is robust and has low overhead. When a stable sort is not needed, quick sort is an excellent general-purpose sort – although the 3-way partitioning version should always be used instead.
The 2-way partitioning code shown above is written for clarity rather than optimal performance; it exhibits poor locality, and, critically, exhibits O(n2) time when there are few unique keys. A more efficient and robust 2-way partitioning method is given in Quicksort is Optimal by Robert Sedgewick and Jon Bentley. The robust partitioning produces balanced recursion when there are many values equal to the pivot, yielding probabilistic guarantees of O(n·lg(n)) time and O(lg(n)) space for all inputs.
With both sub-sorts performed recursively, quick sort requires O(n) extra space for the recursion stack in the worst case when recursion is not balanced. This is exceedingly unlikely to occur, but it can be avoided by sorting the smaller sub-array recursively first; the second sub-array sort is a tail recursive call, which may be done with iteration instead. With this optimization, the algorithm uses O(lg(n)) extra space in the worst case.

| 最好 | 平均 | 最坏 |
|-----:|--------:|------:|
| `Ω(n log(n))` | `Θ(n log(n))` | `O(n^2)` |

[示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/algorithms/sort/quick.rb) | [维基百科](https://en.wikipedia.org/wiki/Quicksort)

#### 其它排序算法

* [http://rosettacode.org/wiki/Category:Sorting_Algorithms](http://rosettacode.org/wiki/Category:Sorting_Algorithms)

* [https://www.toptal.com/developers/sorting-algorithms](https://www.toptal.com/developers/sorting-algorithms)

#### Additional reading

* [https://www.igvita.com/2009/03/26/ruby-algorithms-sorting-trie-heaps/](https://www.igvita.com/2009/03/26/ruby-algorithms-sorting-trie-heaps/)

### 搜索

#### 二分查找

In computer science, binary search, also known as half-interval search or logarithmic search, is a search algorithm that finds the position of a target value within a sorted array. It compares the target value to the middle element of the array; if they are unequal, the half in which the target cannot lie is eliminated and the search continues on the remaining half until it is successful.

| 最好 | 平均 | 最坏 |
|-----:|--------:|------:|
| `Θ(1)` | `Θ(n log(n))` | `O(log(n))` |

[示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/algorithms/search/binary.rb) | [维基百科](https://en.wikipedia.org/wiki/Knuth%E2%80%93Morris%E2%80%93Pratt_algorithm)

#### KMP查找

In computer science, the Knuth–Morris–Pratt string searching algorithm (or KMP algorithm) searches for occurrences of a "word" W within a main "text string" S by employing the observation that when a mismatch occurs, the word itself embodies sufficient information to determine where the next match could begin, thus bypassing re-examination of previously matched characters.

[示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/algorithms/sort/knuth-morris-pratt.rb) | [维基百科](https://en.wikipedia.org/wiki/Binary_search_algorithm)

#### 其它算法

* [Dijkstra算法](https://en.wikipedia.org/wiki/Dijkstra%27s_algorithm)

* [Kruskal算法](https://en.wikipedia.org/wiki/Kruskal%27s_algorithm)

* [最长上升子序列](https://en.wikipedia.org/wiki/Longest_increasing_subsequence)

* [电话号码对应英语单词](http://www.mobilefish.com/services/phonenumber_words/phonenumber_words.php)

#### 代码及文献资源:

* [https://github.com/sagivo/algorithms](https://github.com/sagivo/algorithms)

* [https://github.com/kanwei/algorithms](https://github.com/kanwei/algorithms)



