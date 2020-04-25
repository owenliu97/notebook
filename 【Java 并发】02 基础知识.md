>本文主要覆盖《Java 并发编程实战》的第一部分（2-5章）内容，属于较为基础的知识，要求是掌握熟练，掌握基本使用方法。

## 线程安全性

### 什么是线程安全性

>多线程访问一个类时，无论采用何种调度方式，线程如何交替执行，主调代码中不需要额外的同步或协同，该类都能表现出正确的行为，则该类线程安全。

从定义中可以看出，线程安全第一强调正确性，第二强调同步机制加在线程安全类中，客户端无需采取同步措施。

最简单的线程安全性为无状态对象，即不包含任何域或者对其他类的域引用。

### 原子性

> 多个对象同时调用某个对象，提取域的值并运算，会出现最终数据与想象中不符。抽象的说，当某个计算的正确性取决于多个线程的交替时序，就会发生竞态条件 Race Condition。`先检查后执行`就是一种竞态条件。实际应用中可以使用原子类实现原子性。
>
> 原子类（例如 AtomicLong），通过调用 UnSafe 类中的 CAS 操作实现底层的原子性。CAS操作是现代处理器的原子操作，在 JDK 9 以后对用户开放使用，JDK 8 中只能使用原子类等方式间接调用。

### 加锁机制

Java 通过加锁实现原子性，内部的机制为同步代码块 synchronized。synchronized 是可重入的，即封锁的粒度是线程而不是调用。

这段代码是可重入的例子，因为父类方法和子类方法实际上对一个实例来说使用的是一个锁。（原因是 synchronized 修饰方法时默认对 this 加锁。）

``` java
public class Widget {  
    public synchronized void doSomething() {  
        ...  
    }  
}  
  
public class LoggingWidget extends Widget {  
    public synchronized void doSomething() {  
        System.out.println(toString() + ": calling doSomething");  
        super.doSomething();  
    }  
}
```

加锁必然导致性能下降。

## 对象的共享

同步的另一个方面是可见性，可见性是指写线程完成对数据的修改后应及时使读线程可见。

### 可见性与 volatile 关键字

实现可见性，禁止指令重排序。volatile 不能保证同步性原子性，只能保证可见性有序性。使用 volatile 需要同时满足以下条件：

- 对变量的写入操作不依赖变量的当前值，或保证只有单个线程更新变量的值（否则多个线程更新同一个 volatile 变量仍然线程不安全）
- 该变量不能和其他变量一起纳入不变性
- 访问变量不需要加锁

### 对象的发布和逸出

把一个对象开放给作用于之外的代码使用，就叫做对象的发布。发布必然会带来使用它的线程不可控制，出现线程安全问题。如果是不该发布的对象被发布，就叫做逸出（escape）。

以下这些操作都可以导致对象被发布：

- 将对象保存在公有静态变量中，发布该对象以及其中引用的对象
- 用非私有方法返回一个对象的引用
- 将对象传递给外部方法（即非 private final 方法）
- 对象发布一个内部类实例（使内部类中隐含对该对象本身的引用）

```java
public class ThisEscape {
    public ThisEscape(EventSource source) {
        source.registerListener(new EventListener() { 
         		// 建立新内部匿名类对象时，重写了 onEvent 方法，隐含对 this 的引用
            @Override
            public void onEvent(Event e) {
                doSomething(e);
            }
        });
    }

    void doSomething(Event e) {}
  
    interface EventSource { void registerListener(EventListener e); }

    interface EventListener { void onEvent(Event e); }

    interface Event {}
}
```

避免逸出就是安全对象的构造，保证安全构造的方法主要有：

- 不要使 this 引用逸出
- 不要在构造函数中启动一个线程并立即启动
- 使用私有构造函数加工厂方法

``` java
public class SafeListener {
    private final EventListener listener;

    private SafeListener() {
        listener = new EventListener() {
            @Override
            public void onEvent(Event e) {
                doSomething(e);
            }
        };
    }

    public static SafeListener newInstance(EventSource source) {
      	// 该工厂方法有效防止 this 逸出
        SafeListener safe = new SafeListener();
        source.registerListener(safe.listener);
        return safe;
    }

    void doSomething(Event e) {}

    interface EventSource { void registerListener(EventListener e); }

    interface EventListener { void onEvent(Event e); }

    interface Event {}
}
```

### 线程封闭

线程封闭使对象封闭在线程中，通过不共享数据自动解决了线程安全问题。Java 中提供局部变量和 ThreadLocal 类来维护线程封闭性，但是仍需要通过编程确保对象不从线程中逸出。Ad-hoc 线程封闭是维护线程封闭完全由程序本身承担，语言十分脆弱，不如使用单线程子系统。

#### 栈封闭

只能通过局部变量访问数据。保证方法中的局部变量引用不外泄。

``` java
public int loadTheArk (Collection<Animal> candidates) {
            SortSet<Animal> animals;
            int numPairs = 0;
            Animal candidate = null;
            //animals被封装在方法中，不能使它们溢出
            animals = new TreeSet<Animals>(new SpeciesGenderComparator());
            animals.addAll(candidates);
            for (Animal a : animals){
                if (candidate == null || !candidate.isPotentialMate(a)) {
                    candidate = a;
                }else {
                    ark.load(new AnimalPair(candidate , a));
                    ++numPairs;
                    candidate = null;
                }
            }
        }
```

#### ThreadLocal 类

ThreadLocal 类使共享变量在不同的线程中的值保存起来，提供 get 和 set 接口调用。通过该类保存进程的上下文信息，为编程提供便利。但是滥用 ThreadLocal 会导致代码耦合性高。

### 不变性 Immutable

使用不可变对象是一定线程安全的。不可变对象的满足条件包括：

1. 创建后状态不可修改
2. 所有域都是 final 类型
3. 类的 this 引用不会在创建时逸出

使用 final 声明的域是不可变的，<u>但是其引用的对象仍然是可变的</u>。在 Java 内存模型中，final 还可以保证初始化过程的安全性，因为 JVM 指令重排序使对象引用可能早于初始化发布，加入 final 声明后则在最后才发布，*因此可以不受限制的访问不可变对象，并在共享这些对象时无需同步*。

## 对象的组合

### 设计线程安全的类

分为以下三步：找到状态变量，找到约束变量的不变性条件，设计同步策略。

#### 找出构成对象状态的变量

对象的状态变量是对象内部的域，包括所有基本数据类型域的可能取值和引用对象的状态的总和。

#### 找出约束状态变量的不变性条件



#### 建立对象状态并发访问管理策略



















