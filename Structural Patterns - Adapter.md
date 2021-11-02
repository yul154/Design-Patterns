# 这个设计模式的意图是什么,它要解决一个什么问题，

两个对象因接口不兼容而不能在一起工作的实例，这时需要第三者进行适配。
* 讲中文的人同讲英文的人对话时需要一个翻译
* 用直流电的笔记本电脑接交流电源时需要一个电源适配器
* 用计算机访问照相机的 SD 内存卡时需要一个读卡器等。

# 它是如何解决的，

## 结构和组成
在适配器模式中
* 引入了一个被称为适配器(Adapter)的包装类
* 而它所包装的对象称为适配者(Adaptee)，即被适配的类。
 
适配器的实现就是把客户类的请求转化为对适配者的相应接口的调用


## 解决过程
客户类调用适配器的方法时
* 我们通过增加一个新的适配器类来解决接口不兼容的问题
* 在适配器类的内部将调用适配者类的方法，而这个过程对客户类是透明的
* 客户类并不直接访问适配者类
* 适配器模式可以将一个类的接口和另一个类的接口匹配起来，而无须修改原来的适配者接口和抽象目标类接口。
* 适配器让那些由于接口不兼容而不能交互的类可以一起工作

## 适配器类别

适配器模式
* 可以作为类结构型模式： 适配器与适配者之间是继承（或实现）关系
* 可以作为对象结构型模式：在对象适配器模式中，适配器与适配者之间是关联关系


# 角色和结构图
* Target（目标抽象类）：目标抽象类定义客户所需接口，可以是一个抽象类或接口，也可以是具体类。
* Adapter（适配器类）：适配器可以调用另一个接口，作为一个转换器，对Adaptee和Target进行适配，适配器类是适配器模式的核心，在对象适配器中，它通过继承Target并关联一个Adaptee对象使二者产生联系。
* Adaptee（适配者类）：适配者即被适配的角色，它定义了一个已经存在的接口，这个接口需要适配，适配者类一般是一个具体类，包含了客户希望使用的业务方法，在某些情况下可能没有适配者类的源代码

![image](https://user-images.githubusercontent.com/27160394/139809723-c9104f94-390e-4bb6-801e-3ac2393cf937.png)

# 代码实现

## 对象适配器模式
*  Adapter `extends`  目标接口
*  维持一个对适配者对象的引用 
*  在目标需求方法中年调用适配者的方法

```
class Adapter extends Target {  
    private Adaptee adaptee; //维持一个对适配者对象的引用  
      
    public Adapter(Adaptee adaptee) {  
        this.adaptee=adaptee;  
    }  
      
    public void request() {  
        adaptee.specificRequest(); //转发调用  
    }  
} 

class OperationAdapter implements ScoreOperation {  
    private QuickSort sortObj; //定义适配者QuickSort对象  
    private BinarySearch searchObj; //定义适配者BinarySearch对象  
  
    public OperationAdapter() {  
        sortObj = new QuickSort();  
        searchObj = new BinarySearch();  
    }  
  
    public int[] sort(int array[]) {    
            return sortObj.quickSort(array); //调用适配者类QuickSort的排序方法  
}  
  
    public int search(int array[],int key) {    
          return searchObj.binarySearch(array,key); //调用适配者类BinarySearch的查找方法  
}  
```
## 类适配器
类适配器模式中适配器和适配者是继承关系
* 适配器类实现了抽象目标类接口Target
* 并继承了适配者类
* 在适配器类的方法中调用所继承的适配者类的方法，实现了适配

目标抽象类Target不是接口，而是一个类，就无法使用类适配器；此外，如果适配者Adapter为最终(Final)类，也无法使用类适配器

```
class Adapter extends Adaptee implements Target {  
    public void request() {  
        specificRequest();  
    }  
}  
```
## 双向适配器
在对象适配器的使用过程中
* 如果在适配器中同时包含对目标类和适配者类的引用
* 适配者可以通过它调用目标类中的方法
* 目标类也可以通过它调用适配者类中的方法

```
class Adapter implements Target,Adaptee {  
    //同时维持对抽象目标类和适配者的引用  
    private Target target;  
    private Adaptee adaptee;  
      
    public Adapter(Target target) {  
        this.target = target;  
    }  
      
    public Adapter(Adaptee adaptee) {  
        this.adaptee = adaptee;  
    }  
      
    public void request() {  
        adaptee.specificRequest();  
    }  
      
    public void specificRequest() {  
        target.request();  
    }  
}  
```
## 缺省适配器(Default Adapter Pattern)
当不需要实现一个接口所提供的所有方法时
* 可先设计一个抽象类实现该接口
* 并为接口中每个方法提供一个默认实现（空方法）
* 那么该抽象类的子类可以选择性地覆盖父类的某些方法来实现需求
* 它适用于不想使用一个接口中的所有方法的情况，又称为单接口适配器模式

### 角色

![image](https://user-images.githubusercontent.com/27160394/139811404-a7efe4e0-b8bc-4b4d-b21c-586414322bfa.png)

* ServiceInterface（适配者接口）：它是一个接口，通常在该接口中声明了大量的方法。

* AbstractServiceClass（缺省适配器类）：它是缺省适配器模式的核心类，使用空方法的形式实现了在ServiceInterface接口中声明的方法。通常将它定义为抽象类，因为对它进行实例化没有任何意义。

* ConcreteServiceClass（具体业务类）：它是缺省适配器类的子类，在没有引入适配器之前，它需要实现适配者接口，因此需要实现在适配者接口中定义的所有方法，而对于一些无须使用的方法也不得不提供空实现。在有了缺省适配器之后，可以直接继承该适配器类，根据需要有选择性地覆盖在适配器类中定义的方法




# Summary

## 这个模式的优缺点是什么，
**Pro**
* 将目标类和适配者类解耦，通过引入一个适配器类来重用现有的适配者类，无须修改原有结构
* 增加了类的透明性和复用性，将具体的业务实现过程封装在适配者类中，对于客户端类而言是透明的，而且提高了适配者的复用性，同一个适配者类可以在多个不同的系统中复用。
* 灵活性和扩展性都非常好，通过使用配置文件，可以很方便地更换适配器，也可以在不修改原有代码的基础上增加新的适配器类，完全符合“开闭原则”

类适配模式
* 由于适配器类是适配者类的子类，因此可以在适配器类中置换一些适配者的方法，使得适配器的灵活性更强

对象适配模式
* 一个对象适配器可以把多个不同的适配者适配到同一个目标；
* 可以适配一个适配者的子类，由于适配器和适配者之间是关联关系，根据“里氏代换原则”，适配者的子类也可通过该适配器进行适配


**Con**
类适配器模式的缺点
* 一次最多只能适配一个适配者类，不能同时适配多个适配者
* 适配者类不能为最终类
* 类适配器模式中的目标抽象类只能为接口，不能为类

对象适配器模式
* 要在适配器中置换适配者类的某些方法比较麻烦。如果一定要置换掉适配者类的一个或多个方法，可以先做一个适配者类的子类，将适配者类的方法置换掉，然后再把适配者类的子类当做真正的适配者进行适配

## 适用场景
* 以前开发的系统存在满足新系统功能需求的类，但其接口同新系统的接口不一致。
* 使用第三方提供的组件，但组件接口定义和自己要求的接口定义不同

## 在使用时要注意什么
* 类适配器: 以类给到.在Adapter里,就是将src当做类,继承
* 对象适配器: 以对象给到,在Adapter里,将src作为一个对象,持有
* 接口适配器: 以接口给到,在Adapter里,将src作为一个接口,实现

