---
title: 软件设计模式之代理（proxy）、委派（delegate）
tag: [软件设计, Java]
key: 2022-03-22-软件设计模式之代理（proxy）、委派（delegate）
---

Refer:

[委托模式 wikipedia](https://zh.wikipedia.org/wiki/%25E5%25A7%2594%25E6%2589%2598%25E6%25A8%25A1%25E5%25BC%258F)

[代理模式  菜鸟教程](https://www.runoob.com/design-pattern/proxy-pattern.html)

[JAVA 动态代理](https://www.jianshu.com/p/9bcac608c714)

[动态代理  廖雪峰](https://www.liaoxuefeng.com/wiki/1252599548343744/1264804593397984)

- [Java 设计模式——Delegate 委派模式](https://www.jianshu.com/p/192e4e6ba648)
- [mixin  wikipedia](https://zh.m.wikipedia.org/zh-hans/Mixin)

# 代理 proxy

[代理模式  菜鸟教程](https://www.runoob.com/design-pattern/proxy-pattern.html)代码例子

## 使用场景

- **远程代理**，这种方式通常是为了隐藏目标对象存在于不同地址空间的事实，方便客户端访问。例如，用户申请某些网盘空间时，会在用户的文件系统中建立一个虚拟的硬盘，用户访问虚拟硬盘时实际访问的是网盘空间。
- **虚拟代理**，这种方式通常用于要创建的目标对象开销很大时。例如，下载一幅很大的图像需要很长时间，因某种计算比较复杂而短时间无法完成，这时可以先用小比例的虚拟代理替换真实的对象，消除用户对服务器慢的感觉。
- **安全代理**，这种方式通常用于控制不同种类客户对真实对象的访问权限。
- **智能指引**，主要用于调用目标对象时，代理附加一些额外的处理功能。例如，增加计算真实对象的引用次数的功能，这样当该对象没有被引用时，就可以自动释放它。
- **延迟加载**，指为了提高系统的性能，延迟对目标的加载。例如，Hibernate 中就存在属性的延迟加载和关联表的延时加载

## Java 动态代理

[动态代理  廖雪峰](https://www.liaoxuefeng.com/wiki/1252599548343744/1264804593397984)代码例子

Java 标准库提供了一种动态代理（Dynamic Proxy）的机制：可以在运行期动态创建某个 `interface` 的实例，无需实现类。

动态代理实际上是 JVM 在运行期动态创建 class 字节码并加载的过程

# 委派 delegate

```java
 class RealPrinter { // the "delegate"
     void print() { 
       System.out.print("something"); 
     }
 }
 
 class Printer { // the "delegator"
     RealPrinter p = new RealPrinter(); // create the delegate 
     void print() { 
       p.print(); // delegation
     } 
 }
 
 public class Main {
     // to the outside world it looks like Printer actually prints.
     public static void main(String[] args) {
         Printer printer = new Printer();
         printer.print();
     }
 }

```

## 使用场景

- 类 B 和类 A 是两个互相没有任何关系的类，但是 B 具有和 A 一模一样的方法和属性；并且调用 B 中的方法/属性就是调用 A 中同名的方法和属性。
- B 好像就是一个受 A 授权委托的中介，第三方的代码不需要知道 A 的存在，也不需要和 A 发生直接的联系，通过 B 就可以直接使用 A 的功能，这样既能够使用到 A 的各种功能，又能够很好的将 A 保护起来。
- 一件事情（或一个请求）对象 A 本身不知道怎样处理，对象 A 把请求交给其它对象 B,C 来做。外界看起来是对象 A 做的一样。
- 委托模式使得我们可以用聚合来替代继承，它还使我们可以模拟 mixin。

> mixin，是面向对象程序设计语言中的类，提供了方法的实现。其他类可以访问 mixin 类的方法而不必成为其子类。Mixin 有时被称作"included"而不是"inherited"。mixin 为使用它的 class 提供额外的功能，但自身却不单独使用（不能单独生成实例对象，属于抽象类）。因为有以上限制，Mixin 类通常作为功能模块使用，在需要该功能时“混入”，而且不会使类的关系变得复杂。使用者与 Mixin 不是“is-a”的关系，而是“-able”关系

- 缺点：代码量大
- 如果可以使用接口，那委派可以做到类型更安全并且更加灵活。

> 类型安全，旨在避免必然的错误形式，和不良的程序行为（称为类型错误）
>
> 类型错误（type error）是错误或不期望的程序行为，由不同数据类型的差别所引起，适用于程序的常量、变量、方法（函数），如把整型（int）当作了浮点型（float）。
>
> 类型安全近似于所谓的存储器安全（就是限制从存储器的某处，将任意的字节合复制到另一处的能力）。例如，如果初始化一个指向整数区域数据结构的指针，但新对象的指针区域却分配在整数的地方，然后指针区域可借由改变整数区域的值简单改变成任何东西（经由间接引用悬置指针）。




<details markdown="1"><summary>在这个例子里，类C可以委托类A或类B，类C拥有方法使自己可以在类A或类B间选择。因为类A或类B必须实现接口I规定的方法，所以在这里委托是类型安全的。</summary>
```java
package Paint;

interface I {
    void f();
    void g();
}

class A implements I {
    public void f() {
        System.out.println("A: doing f()");
    }
    public void g() {
        System.out.println("A: doing g()");
    }
}

class B implements I {
    public void f() {
        System.out.println("B: doing f()");
    }
    public void g() {
        System.out.println("B: doing g()");
    }
}

class C implements I {
    I i = new A();
    public void f() {
        i.f();
    }
    public void g() {
        i.g();
    }
    public void toA() {
        i = new A();
    }
    public void toB() {
        i = new B();
    }
}

public class Main {
    public static void main(String[] args) {
        C c = new C();
        c.f();      // output: A: doing f()
        c.g();      // output: A: doing g()
        c.toB();    // 更换委托对象
        c.f();      // output: B: doing f()
        c.g();      // output: B: doing g()
    }
}
```
</details>



**个人理解：**

- delegation 可以提供给其他类使用，委托模式可以自由切换被委托者
- proxy 是防止除了代理方以外的类使用，目标对象从头到尾不会有任何的改变

比如在一场考试中，监考老师 C，考生 A

1. 如果考生 A 手骨折了，无法完成考试，需要另一个学生 B 听 A 口述写下答案，那就 B 是 proxy
2. 如果 A 请了枪手 B 或枪手 D 等等来教室考试，那就是 delegation 模式
