# 现实例子

* 毛笔：2型号的毛笔 * 2颜料盒 = 4，颜色和型号实现了分离(将二者解耦)，增加新的颜色或者型号对另一方都没有任何影响
* 蜡笔：4 蜡笔(型号+颜色)，蜡笔中的颜色和型号两个不同的变化维度（即两个不同的变化原因）融合在一起(较强的耦合性)，无论是对颜色进行扩展还是对型号进行扩展都势必会影响另一个维度



# 这个设计模式的意图是什么, 它要解决一个什么问题，

多层继承存在的问题： 某些类具有两个或多个维度的变化,使用了一种多层继承结构,导致系统中类的个数急剧增加,系统扩展麻烦


# 它是如何解决的，

1. 与多层继承方案不同，它将两个独立变化的维度设计为两个独立的继承等级结构
2. 并且在抽象层建立一个抽象关联，该关联关系类似一条连接两个独立继承结构的桥

桥接模式用一种巧妙的方式处理多层继承存在的问题
* 用抽象关联取代了传统的多层继承
* 将类之间的静态继承关系转换为动态的对象组合关系
* 使得系统更加灵活，并易于扩展，同时有效控制了系统中类的个数


## More details
* 应该识别出一个类所具有的两个独立变化的维度，将它们设计为两个独立的继承等级结构
* 为两个维度都提供抽象层，并建立抽象耦合
* 将具有两个独立变化维度的类的一些普通业务方法和与之关系最密切的维度设计为“抽象类”层次结构
* 将另一个维度设计为“实现类”层次结构（实现部分)

# 结构图和实现

![image](https://user-images.githubusercontent.com/27160394/139825203-72aa6dd5-61ad-44b4-9aa6-98c8f75f0b6e.png)

## 桥接模式中包含如下几个角色

* Abstraction：用于定义抽象类的接口，它一般是抽象类而不是接口
  * 其中定义了一个Implementor类型的对象并可以维护该对象，它与Implementor之间具有关联关系
  * 它既可以包含抽象业务方法，也可以包含具体业务方法。
* RefinedAbstraction（扩充抽象类) : 扩充由Abstraction定义的接口，通常情况下它不再是抽象类而是具体类
  * 它实现了在Abstraction中声明的抽象业务方法
  * 在RefinedAbstraction中可以调用在Implementor中定义的业务方法。
* Implementor(实现类接口): 定义实现类的接口，这个接口不一定要与Abstraction的接口完全一致，事实上这两个接口可以完全不同
  * Implementor接口仅提供基本操作，而Abstraction定义的接口可能会做更多更复杂的操作。
  * Implementor接口对这些基本操作进行了声明，而具体实现交给其子类。
  * 通过关联关系，在Abstraction中不仅拥有自己的方法，还可以调用到Implementor中定义的方法，使用关联关系来替代继承关系。
  
* ConcreteImplementor（具体实现类) : 具体实现Implementor接口，在不同的ConcreteImplementor中提供基本操作的不同实现
  * 在程序运行时，ConcreteImplementor对象将替换其父类对象，提供给抽象类具体的业务操作方法。

## 实现代码
### 抽象层
* 实现类接口
```
interface Implementor {  
    public void operationImpl();  
} 
```
* 抽象类
```
abstract class Abstraction {  
    protected Implementor impl; //定义实现类接口对象  
      
    public void setImpl(Implementor impl) {  
        this.impl=impl;  
    }  
      
    public abstract void operation();  //声明抽象业务方法  
}  
```
###  实现层
* 细化抽象类(RefinedAbstraction)
```
class RefinedAbstraction extends Abstraction {  
    public void operation() {  
        //业务代码  
        impl.operationImpl();  //调用实现类的方法  
        //业务代码  
    }  
}  
```

# 适配器模式与桥接模式的联用
Purpose
* 适配器模式通常用于现有系统与第三方产品功能的集成，采用增加适配器的方式将第三方类集成到系统中
* 桥接模式, 用户可以通过接口继承或类继承的方式来对系统进行扩展。

桥接模式和适配器模式用于设计的不同阶段
* 桥接模式用于系统的初步设计，对于存在两个独立变化维度的类可以将其分为抽象化和实现化两个角色，使它们可以分别进行变化
* 而在初步设计完成之后，当发现系统与已有类无法协同工作时，可以采用适配器模式。但有时候在设计初期也需要考虑适配器模式，特别是那些涉及到大量第三方应用接口的情况

# 桥接模式总结
桥接模式是设计Java虚拟机和实现JDBC等驱动程序的核心模式之一，应用较为广泛。在软件开发中如果一个类或一个系统有多个变化维度时(什么时候可以使用它)，都可以尝试使用桥接模式对其进行设计。

## 主要优点

* 分离抽象接口及其实现部分。桥接模式使用“对象间的关联关系”解耦了抽象和实现之间固有的绑定关系，使得抽象和实现可以沿着各自的维度来变化
  * 抽象和实现沿着各自维度的变化 ： 抽象和实现不再在同一个继承层次结构中，而是“子类化”它们，使它们各自都具有自己的子类，以便任何组合子类，从而获得多维度组合对象
* 在很多情况下，桥接模式可以取代多层继承方案，多层继承方案违背了“单一职责原则”，复用性较差，且类的个数非常多，桥接模式是比多层继承方案更好的解决方法，它极大减少了子类的个数
* 桥接模式提高了系统的可扩展性，在两个变化维度中任意扩展一个维度，都不需要修改原有系统，符合“开闭原则”。
* 符合合成复用原则


## 缺点

* 桥接模式的使用会增加系统的理解与设计难度，由于关联关系建立在抽象层，要求开发者一开始就针对抽象层进行设计与编程。
* 桥接模式要求正确识别出系统中两个独立变化的维度，因此其使用范围具有一定的局限性，如何正确识别两个独立维度也需要一定的经验积累。

## 适用场景
* 当一个类内部具备两种或多种变化维度时，使用桥接模式可以解耦这些变化的维度，使高层代码架构稳定
* 当一个系统需要在构件的抽象化角色和具体化角色之间增加更多的灵活性时。
* 一个类存在两个（或多个）独立变化的维度，且这两个（或多个）维度都需要独立进行扩展
* 对于那些不希望使用继承或因为多层继承导致系统类的个数急剧增加的系统，桥接模式尤为适用。
