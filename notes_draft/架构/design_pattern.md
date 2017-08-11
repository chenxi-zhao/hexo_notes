>![史上最全设计模式导学目录（完整版）][http://blog.csdn.net/lovelion/article/details/17517213]

### 创建型模式
#### 简单工厂模式
> 简单工厂模式(Simple Factory Pattern)：定义一个工厂类，它可以根据参数的不同返回不同类的实例，被创建的实例通常都具有共同的父类。因为在简单工厂模式中用于创建实例的方法是静态(static)方法，因此简单工厂模式又被称为静态工厂方法(Static Factory Method)模式，它属于类创建型模式。

![](http://static.tmaczhao.cn/images/daac881bebe605d8b22324957cf269aa.jpg)
>将抽象产品类和工厂类合并，将静态工厂方法移至抽象产品类中

#### 工厂方法模式
> 工厂方法模式(Factory Method Pattern)：定义一个用于创建对象的接口，让子类决定将哪一个类实例化。工厂方法模式让一个类的实例化延迟到其子类。工厂方法模式又简称为工厂模式(Factory Pattern)，又可称作虚拟构造器模式(Virtual Constructor Pattern)或多态工厂模式(Polymorphic Factory Pattern)。工厂方法模式是一种类创建型模式。

![](http://static.tmaczhao.cn/images/2c3d34220d8274ccb8c66bb42f57dcf1.jpg)

#### 抽象工厂
> 抽象工厂模式(Abstract Factory Pattern)：提供一个创建一系列相关或相互依赖对象的接口，而无须指定它们具体的类。抽象工厂模式又称为Kit模式，它是一种对象创建型模式。

![](http://static.tmaczhao.cn/images/11096cffb61581e1ed6e2f138736421f.jpg)

#### 单例模式
1. 饿汉模式
![](http://static.tmaczhao.cn/images/6dcf948cbe3f161a0896e2b3f9a0ea96.jpg)

2. 懒汉模式
![](http://static.tmaczhao.cn/images/2fbcd6a2619040be276f3d9f8ea2ffad.jpg)

```java
class Singleton {
    private Singleton() {
    }

    private static class HolderClass {
            private final static Singleton instance = new Singleton();
    }

    public static Singleton getInstance() {
        return HolderClass.instance;
    }

    public static void main(String args[]) {
        Singleton s1, s2;
            s1 = Singleton.getInstance();
        s2 = Singleton.getInstance();
        System.out.println(s1==s2);
    }
}
```
#### 原型模式
![](http://static.tmaczhao.cn/images/7f54e69dac428ec0d8f132c88c25ac30.jpg)
>java中的拷贝（深拷贝、浅拷贝）

#### 建造者模式
![](http://static.tmaczhao.cn/images/4895dab08652f696bd16f0d035eaea2d.jpg)
>Director（指挥者）：指挥者又称为导演类，它负责安排复杂对象的建造次序，指挥者与抽象建造者之间存在关联关系，可以在其construct()建造方法中调用建造者对象的部件构造与装配方法，完成复杂对象的建造。




### 结构性模式

#### 适配器模式
>适配器模式(Adapter Pattern)：将一个接口转换成客户希望的另一个接口，使接口不兼容的那些类可以一起工作，其别名为包装器(Wrapper)。适配器模式既可以作为类结构型模式，也可以作为对象结构型模式。

**Jdk中的应用：**`java.io.InputStreamReader(InputStream)`

![](http://static.tmaczhao.cn/images/3f4a45daf2334c9a97645979b79acff1.jpg)
>- Target（目标抽象类）：目标抽象类定义客户所需接口，可以是一个抽象类或接口，也可以是具体类。
- Adapter（适配器类）：适配器可以调用另一个接口，作为一个转换器，对Adaptee和Target进行适配，适配器类是适配器模式的核心，在对象适配器中，它通过继承Target并关联一个Adaptee对象使二者产生联系。
- Adaptee（适配者类）：适配者即被适配的角色，它定义了一个已经存在的接口，这个接口需要适配，适配者类一般是一个具体类，包含了客户希望使用的业务方法，在某些情况下可能没有适配者类的源代码。

![](http://static.tmaczhao.cn/images/ed51a9e295eefa2d0d989b0752c8b7de.jpg)


#### 桥接模式
>桥接模式(Bridge Pattern)：将抽象部分与它的实现部分分离，使它们都可以独立地变化。它是一种对象结构型模式，又称为柄体(Handle and Body)模式或接口(Interface)模式。

**Jdk中的应用：**`JDBC`

![](http://static.tmaczhao.cn/images/403b6614476ba347b55b49f791a21c1a.jpg!md)
##### 桥接模式和适配器模式的联用
![](http://static.tmaczhao.cn/images/0a1fff6673ad01fc38fd7aa0007bc7e7.jpg)


#### 组合模式
>组合模式(Composite Pattern)：组合多个对象形成树形结构以表示具有“整体—部分”关系的层次结构。组合模式对单个对象（即叶子对象）和组合对象（即容器对象）的使用具有一致性，组合模式又可以称为“整体—部分”(Part-Whole)模式，它是一种对象结构型模式。

![](http://static.tmaczhao.cn/images/c2150510be59b32a77e49fb36362969b.jpg)
>- Component（抽象构件）：它可以是接口或抽象类，为叶子构件和容器构件对象声明接口，在该角色中可以包含所有子类共有行为的声明和实现。在抽象构件中定义了访问及管理它的子构件的方法，如增加子构件、删除子构件、获取子构件等。
- Leaf（叶子构件）：它在组合结构中表示叶子节点对象，叶子节点没有子节点，它实现了在抽象构件中定义的行为。对于那些访问及管理子构件的方法，可以通过异常等方式进行处理。
- Composite（容器构件）：它在组合结构中表示容器节点对象，容器节点包含子节点，其子节点可以是叶子节点，也可以是容器节点，它提供一个集合用于存储子节点，实现了在抽象构件中定义的行为，包括那些访问及管理子构件的方法，在其业务方法中可以递归调用其子节点的业务方法。

**`组合模式的关键是定义了一个抽象构件类，它既可以代表叶子，又可以代表容器，而客户端针对该抽象构件类进行编程，无须知道它到底表示的是叶子还是容器，可以对其进行统一处理。`**

**实例**
![](http://static.tmaczhao.cn/images/b67fed9501620cb9c2f1030cb07af0b3.jpg)


#### 装饰模式
>装饰模式(Decorator Pattern)：动态地给一个对象增加一些额外的职责，就增加对象功能来说，装饰模式比生成子类实现更为灵活。装饰模式是一种对象结构型模式。

![](http://static.tmaczhao.cn/images/26b2e746aa669ebeb4b9b1781d2b5583.jpg)
>- Component（抽象构件）：它是具体构件和抽象装饰类的共同父类，声明了在具体构件中实现的业务方法，它的引入可以使客户端以一致的方式处理未被装饰的对象以及装饰之后的对象，实现客户端的透明操作。
- ConcreteComponent（具体构件）：它是抽象构件类的子类，用于定义具体的构件对象，实现了在抽象构件中声明的方法，装饰器可以给它增加额外的职责（方法）。
- Decorator（抽象装饰类）：它也是抽象构件类的子类，用于给具体构件增加职责，但是具体职责在其子类中实现。它维护一个指向抽象构件对象的引用，通过该引用可以调用装饰之前构件对象的方法，并通过其子类扩展该方法，以达到装饰的目的。
- ConcreteDecorator（具体装饰类）：它是抽象装饰类的子类，负责向构件添加新的职责。每一个具体装饰类都定义了一些新的行为，它可以调用在抽象装饰类中定义的方法，并可以增加新的方法用以扩充对象的行为。

**实例**
![](http://static.tmaczhao.cn/images/713d4012857d8cda05bb9ec31bc5b686.jpg)
```java
// ...
Component component,componentSB,componentBB;  //使用抽象构件定义
component = new Window(); //定义具体构件
componentSB = new  ScrollBarDecorator(component); //定义装饰后的构件
componentBB = new  BlackBorderDecorator(componentSB); //将装饰了一次之后的对象继续注入到另一个装饰类中，进行第二次装饰
componentBB.display();
// ...
```
#### 外观模式
>外观模式：为子系统中的一组接口提供一个统一的入口。外观模式定义了一个高层接口，这个接口使得这一子系统更加容易使用。
外观模式中，一个子系统的外部与其内部的通信通过一个统一的外观类进行，外观类将客户类与子系统的内部复杂性分隔开，使得客户类只需要与外观角色打交道，而不需要与子系统内部的很多对象打交道。

![](http://static.tmaczhao.cn/images/7bc01ca6c16f2386341f39b407a488ed.jpg)

**实例**
![](http://static.tmaczhao.cn/images/dd4e72962bf360e7a955410a360f6c88.jpg)

##### 抽象外观类
![](http://static.tmaczhao.cn/images/4b1dea9e915308a36f7daba8496a802b.jpg)


#### 享元模式
享元模式通过共享技术实现相同或相似对象的重用，`在逻辑上每一个出现的字符都有一个对象与之对应，然而在物理上它们却共享同一个享元对象`，这个对象可以出现在一个字符串的不同地方，相同的字符对象都指向同一个实例，在享元模式中，存储这些共享实例对象的地方称为`享元池(Flyweight Pool)`。

享元对象能做到共享的关键是区分了内部状态(Intrinsic State)和外部状态(Extrinsic State):
1. 内部状态是存储在享元对象内部并且不会随环境改变而改变的状态，内部状态可以共享。如字符a内部状态下始终是a不会改变
2. 外部状态是随环境改变而改变的、不可以共享的状态。字符颜色字体大小改变等

>享元模式(Flyweight Pattern)：运用共享技术有效地支持大量细粒度对象的复用。系统只使用少量的对象，而这些对象都很相似，状态变化很小，可以实现对象的多次复用。由于享元模式要求能够共享的对象必须是细粒度对象，因此它又称为轻量级模式，它是一种对象结构型模式。

![](http://static.tmaczhao.cn/images/06be93b24f65e840ff67351e5aa19b23.jpg)
>- Flyweight（抽象享元类）：通常是一个接口或抽象类，在抽象享元类中声明了具体享元类公共的方法，这些方法可以向外界提供享元对象的内部数据（内部状态），同时也可以通过这些方法来设置外部数据（外部状态）。
- ConcreteFlyweight（具体享元类）：它实现了抽象享元类，其实例称为享元对象；在具体享元类中为内部状态提供了存储空间。通常我们可以结合单例模式来设计具体享元类，为每一个具体享元类提供唯一的享元对象。
- UnsharedConcreteFlyweight（非共享具体享元类）：并不是所有的抽象享元类的子类都需要被共享，不能被共享的子类可设计为非共享具体享元类；当需要一个非共享具体享元类的对象时可以直接通过实例化创建。
- FlyweightFactory（享元工厂类）：享元工厂类用于创建并管理享元对象。

#### 代理模式
>代理模式：给某一个对象提供一个代理或占位符，并由代理对象来控制对原对象的访问。

![](http://static.tmaczhao.cn/images/18d453f190db452a6d4eab30877da063.jpg)

##### 代理模式和装饰模式的区别
代理模式中，代理类对被代理的对象有控制权，决定其执行或者不执行。而装饰模式中，装饰类对代理对象没有控制权，只能为其增加一层装饰，以加强被装饰对象的功能。



### 行为型模式
#### 责任链模式
>职责链模式(Chain of Responsibility  Pattern)：避免请求发送者与接收者耦合在一起，让多个对象都有可能接收请求，将这些对象连接成一条链，并且沿着这条链传递请求，直到有对象处理它为止。职责链模式是一种对象行为型模式。

![](http://static.tmaczhao.cn/images/f2373a850771089a225d118a03f7b7bd.jpg)
>- Handler（抽象处理者）：它定义了一个处理请求的接口，一般设计为抽象类，由于不同的具体处理者处理请求的方式不同，因此在其中定义了抽象请求处理方法。因为每一个处理者的下家还是一个处理者，因此在抽象处理者中定义了一个抽象处理者类型的对象（如结构图中的successor），作为其对下家的引用。通过该引用，处理者可以连成一条链。
- ConcreteHandler（具体处理者）：它是抽象处理者的子类，可以处理用户请求，在具体处理者类中实现了抽象处理者中定义的抽象请求处理方法，在处理请求之前需要进行判断，看是否有相应的处理权限，如果可以处理请求就处理它，否则将请求转发给后继者；在具体处理者中可以访问链中下一个对象，以便请求的转发。

`职责链模式并不创建职责链，职责链的创建工作必须由系统的其他部分来完成，一般是在使用该职责链的客户端中创建职责链。`
>ConcreteHandler.setSuccessor(NextConcretehandler)


#### 命令模式
>命令模式(Command Pattern)：将一个请求封装为一个对象，从而让我们可用不同的请求对客户进行参数化；对请求排队或者记录请求日志，以及支持可撤销的操作。命令模式是一种对象行为型模式，其别名为动作(Action)模式或事务(Transaction)模式。

![](http://static.tmaczhao.cn/images/39fccc71b4d3685c5c8cd1c76876efb1.jpg)



























