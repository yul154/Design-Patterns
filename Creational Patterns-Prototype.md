# Prototype
> 将一个对象作为原型，通过对其进行复制而克隆出多个和原型类似的新实例.

## 原型模式概述

将一个原型对象传给那个要发动创建的对象，这个要发动创建的对象通过请求原型对象拷贝自己来实现创建过程.

创建克隆对象的工厂就是原型类自身，工厂方法由克隆方法来实现

**Note**
* 通过克隆方法所创建的对象是全新的对象，它们在内存中拥有新的地址，
* 通常对克隆所产生的对象进行修改对原型对象不会造成任何影响，
* 每一个克隆对象都是相互独立的。通过不同的方式修改可以得到一系列相似但不完全相同的对象

## 原型模式包含如下几个角色

* Prototype（抽象原型类）：它是声明克隆方法的接口，是所有具体原型类的公共父类，可以是抽象类也可以是接口，甚至还可以是具体实现类。
* ConcretePrototype（具体原型类）：它实现在抽象原型类中声明的克隆方法，在克隆方法中返回自己的一个克隆对象。
* Client（客户类）：让一个原型对象克隆自身从而创建一个新的对象，在客户类中只需要直接实例化或通过工厂方法等方式创建一个原型对象，再通过调用该对象的克隆方法即可得到多个相同的对象。由于客户类针对抽象原型类Prototype编程，因此用户可以根据需要选择具体原型类，系统具有较好的可扩展性，增加或更换具体原型类都很方便


## 如何实现克隆

### 通用实现方法
* 通用的克隆实现方法是在具体原型类的克隆方法中实例化一个与自身类型相同的对象并将其返回
* 并将相关的参数传入新创建的对象中，保证它们的成员属性相同

```
class ConcretePrototype implements Prototype
{
  private String  attr; //成员属性
  public void  setAttr(String attr)
  {
      this.attr = attr;
  }
  public String  getAttr()
  {
      return this.attr;
  }
  public Prototype  clone() //克隆方法
  {
      Prototype  prototype = new ConcretePrototype(); //创建新对象
      prototype.setAttr(this.attr);
      return prototype;
  }
}
Prototype obj1  = new ConcretePrototype();
obj1.setAttr("Sunny");
Prototype obj2  = obj1.clone();

```

### Java语言提供的clone()方法

```
public class JavaClone implements Cloneable{
	public Prototype clone() {
		Object object = null;
		try {
			object = super.clone();
		}catch(CloneNotSupportedException exception) {
			　System.err.println("Not support cloneable");
		}
		return (Prototype )object;
	}
}
Prototype obj1  = new JavaClone();
Prototype obj2  = obj1.clone();
```
Java语言中的clone()方法满足：
* 对任何对象x，都有`x.clone() != x`，即克隆对象与原型对象不是同一个对象
* 对任何对象x，都有`x.clone().getClass() == x.getClass()`，即克隆对象与原型对象的类型一样；
* 如果对象x的equals()方法定义恰当，那么`x.clone().equals(x)`应该成立。

直接利用Object类的clone()方法
* 在派生类中覆盖基类的clone()方法，并声明为public；
* 在派生类的clone()方法中，调用super.clone()；
* 派生类需实现Cloneable接口。

## 浅克隆(ShallowClone)和深克隆(DeepClone)
在Java语言中，数据类型分为值类型（基本数据类型）和引用类型，值类型包括int、double、byte、boolean、char等简单数据类型，引用类型包括类、接口、数组等复杂类型。

浅克隆和深克隆的主要区别在于是否支持引用类型的成员变量的复制

### 浅克隆
如果原型对象的成员变量是值类型，将复制一份给克隆对象.

如果原型对象的成员变量是引用类型，则将引用对象的地址复制一份给克隆对象，也就是说原型对象和克隆对象的成员变量指向相同的内存地址。

简单来说，在浅克隆中，当对象被复制时只复制它本身和其中包含的值类型的成员变量，而引用类型的成员对象并没有复制

#### 深克隆
在深克隆中，无论原型对象的成员变量是值类型还是引用类型，都将复制一份给克隆对象，深克隆将原型对象的所有引用对象也复制一份给克隆对象。

简单来说，在深克隆中，除了对象本身被复制外，对象所包含的所有成员变量也将复制.

#### 深克隆的实现
在Java语言中，如果需要实现深克隆，可以通过序列化(Serialization)等方式来实现
* 序列化就是将对象写到流的过程，写到流中的对象是原有对象的一个拷贝，而原对象仍然存在于内存中。
* 需要注意的是能够实现序列化的对象其类必须实现Serializable接口，否则无法实现序列化操作
* 通过序列化实现的拷贝不仅可以复制对象本身，而且可以复制其引用的成员对象，
* 因此通过序列化将对象写到一个流中，再从流里将其读出来，可以实现深克隆.

## Prototype Manager
> 将多个原型对象存储在一个集合中供客户端使用，它是一个专门负责克隆对象的工厂，其中定义了一个集合用于存储原型对象，如果需要某个原型对象的一个克隆，可以通过复制集合中对应的原型对象来获得
```
class  PrototypeManager
{
       //定义一个Hashtable，用于存储原型对象
       private Hashtable ht=new Hashtable();
       private static PrototypeManager pm =  new PrototypeManager();
      
       //为Hashtable增加公文对象   
     private  PrototypeManager()
     {
              ht.put("far",new  FAR());
              ht.put("srs",new  SRS());               
     }
  
     //增加新的公文对象
       public void addOfficialDocument(String  key,OfficialDocument doc)
       {
              ht.put(key,doc);
       }
 
       //通过浅克隆获取新的公文对象
       public OfficialDocument  getOfficialDocument(String key)
       {
              return  ((OfficialDocument)ht.get(key)).clone();
       }
      
       public static PrototypeManager  getPrototypeManager()
       {
              return pm;
       }
}
```

## Summary
原型模式作为一种快速创建大量相同或相似对象的方式，在软件开发中应用较为广泛，很多软件提供的复制(Ctrl + C)和粘贴(Ctrl + V)操作就是原型模式的典型应用

### 优点
* 当创建新的对象实例较为复杂时，使用原型模式可以简化对象的创建过程，通过复制一个已有实例可以提高新实例的创建效率
* 扩展性较好，由于在原型模式中提供了抽象原型类，在客户端可以针对抽象原型类进行编程，而将具体原型类写在配置文件中，增加或减少产品类对原有系统都没有任何影响
* 原型模式提供了简化的创建结构，原型模式中产品的复制是通过封装在原型类中的克隆方法实现的，无须专门的工厂类来创建产品
* 可以使用深克隆的方式保存对象的状态，使用原型模式将对象复制一份并将其状态保存起来，以便在需要的时候使用（如恢复到某一历史状态），可辅助实现撤销操作

### 缺点
* 需要为每一个类配备一个克隆方法，而且该克隆方法位于一个类的内部，当对已有的类进行改造时，需要修改源代码，违背了“开闭原则”
* 在实现深克隆时需要编写较为复杂的代码，而且当对象之间存在多重的嵌套引用时，为了实现深克隆，每一层对象对应的类都必须支持深克隆，实现起来可能会比较麻烦。

### 适用场景
* 创建新对象成本较大（如初始化需要占用较长的时间，占用太多的CPU资源或网络资源），新的对象可以通过原型模式对已有对象进行复制来获得，如果是相似对象，则可以对其成员变量稍作修改。
* 如果系统要保存对象的状态，而对象的状态变化很小，或者对象本身占用内存较少时，可以使用原型模式配合备忘录模式来实现
* 需要避免使用分层次的工厂类来创建分层次的对象，并且类的实例对象只有一个或很少的几个组合状态，通过复制原型对象得到新实例可能比使用构造函数创建一个新实例更加方便






* 


