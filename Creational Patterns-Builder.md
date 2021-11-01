# Builder Patternns

它将客户端与包含多个组成部分（或部件）的复杂对象的创建过程分离，客户端无须知道复杂对象的内部组成部分与装配方式，只需要知道所需建造者的类型即可. 将一个复杂对象的构建与它的表示分离，使得同样的构建过程可以创建不同的表示

## 在建造者模式中包含如下几个角色：
* Builder（抽象建造者）：它为创建一个产品Product对象的各个部件指定抽象接口，
  * 在该接口中一般声明两类方法，一类方法是buildPartX()，它它们用于创建复杂对象的各个部件；另一类方法是getResult()，
  * 它们用于返回复杂对象。Builder既可以是抽象类，也可以是接口。
* ConcreteBuilder（具体建造者）：它实现了Builder接口，实现各个部件的具体构造和装配方法，定义并明确它所创建的复杂对象，也可以提供一个方法返回创建好的复杂产品对象
* Product（产品角色）：它是被构建的复杂对象，包含多个组成部件，具体建造者创建该产品的内部表示并定义它的装配过程
* Director（指挥者）：指挥者又称为导演类，它负责安排复杂对象的建造次序，
  * 该类主要有两个作用：一方面它隔离了客户与创建过程；另一方面它控制产品的创建过程
  * 指挥者与抽象建造者之间存在关联关系，可以在其construct()建造方法中调用建造者对象的部件构造与装配方法，完成复杂对象的建造。
  * 客户端一般只需要与指挥者进行交互，在客户端确定具体建造者的类型，并实例化具体建造者对象（也可以通过配置文件和反射机制），然后通过指挥者类的构造函数或者Setter方法将该对象传入指挥者类中。

```
class Product  {
       private  String partA; //定义部件，部件可以是任意类型，包括值类型和引用类型
       private  String partB;
       private  String partC;
       //partA的Getter方法和Setter方法省略
       //partB的Getter方法和Setter方法省略
       //partC的Getter方法和Setter方法省略
}

abstract class Builder {
     //创建产品对象
       protected  Product product=new Product();
      
       public  abstract void buildPartA();
       public  abstract void buildPartB();
       public  abstract void buildPartC();
      
     //返回产品对象
       public  Product getResult() {
              return  product;
       }
}

class Director {
       private  Builder builder;
      
       public  Director(Builder builder) {
              this.builder=builder;
       }
      
       public  void setBuilder(Builder builder) {
              this.builder=builer;
       }
      
     //产品构建与组装方法
       public Product construct() {
              builder.buildPartA();
              builder.buildPartB();
              builder.buildPartC();
              return builder.getResult();
       }
}

```

## 建造者模式与抽象工厂模式

* 建造者模式返回一个完整的复杂产品，而抽象工厂模式返回一系列相关的产品
* 在抽象工厂模式中，客户端通过选择具体工厂来生成所需对象，而在建造者模式中，客户端通过指定具体建造者类型并指导Director类如何去生成对象,侧重于一步步构造一个复杂对象，然后将结果返回


## 关于Director的进一步讨论
简单的Director类用于指导具体建造者如何构建产品，它按一定次序调用Builder的buildPartX()方法，控制调用的先后次序，并向客户端返回一个完整的产品对象

### 省略Director
为了简化系统结构，可以将Director和抽象建造者Builder进行合并，在Builder中提供逐步构建复杂产品对象的construct()方法。由于Builder类通常为抽象类，因此可以将construct()方法定义为静态(static)方法

**Summary**
* 同时还简化了系统结构，但加重了抽象建造者类的职责，
* 如果construct()方法较为复杂，待构建产品的组成部分较多，建议还是将construct()方法单独封装在Director中，这样做更符合“单一职责原则”

### 钩子方法的引入
通过Director类来更加精细地控制产品的创建过程，例如增加一类称之为钩子方法(HookMethod)的特殊方法来控制是否对某个buildPartX()的调用

钩子方法的返回类型通常为boolean类型，方法名一般为isXXX()，钩子方法定义在抽象建造者类中,来更加精细地控制产品的创建过程

## Summary

### 优点
* 在建造者模式中，客户端不必知道产品内部组成的细节，将产品本身与产品的创建过程解耦，使得相同的创建过程可以创建不同的产品对象。
* 每一个具体建造者都相对独立，而与其他的具体建造者无关，因此可以很方便地替换具体建造者或增加新的具体建造者，用户使用不同的具体建造者即可得到不同的产品对象。由于指挥者类针对抽象建造者编程，增加新的具体建造者无须修改原有类库的代码，系统扩展方便，符合“开闭原则”
* 可以更加精细地控制产品的创建过程。将复杂产品的创建步骤分解在不同的方法中，使得创建过程更加清晰，也更方便使用程序来控制创建过程。


### 缺点
* 建造者模式所创建的产品一般具有较多的共同点，其组成部分相似，如果产品之间的差异性很大，例如很多组成部分都不相同，不适合使用建造者模式，因此其使用范围受到一定的限制
* 如果产品的内部变化复杂，可能会导致需要定义很多具体建造者类来实现这种变化，导致系统变得很庞大，增加系统的理解难度和运行成本

### 适用场景
* 需要生成的产品对象有复杂的内部结构，这些产品对象通常包含多个成员属性。
* 需要生成的产品对象的属性相互依赖，需要指定其生成顺序
* 对象的创建过程独立于创建该对象的类。在建造者模式中通过引入了指挥者类，将创建过程封装在指挥者类中，而不在建造者类和客户类中
* 隔离复杂对象的创建和使用，并使得相同的创建过程可以创建不同的产品
