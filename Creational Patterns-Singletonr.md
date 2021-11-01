# Singleton

## 单例的动机

为什么要确保任务管理器的唯一性
* 如果能弹出多个窗口，且这些窗口的内容完全一致，全部是重复对象，这势必会浪费系统资源
* 如果弹出的多个窗口内容不一致，这意味着在某一瞬间系统资源使用情况和进程、服务等信息存在多个状态，


## 单例模式的构成
1. 构造函数的可见性改为private，我们需要禁止类的外部直接使用new来创建对象.
```
private TaskManager() {……}  
```
2. 在类的内部还是可以创建的，可见性只对类外有效，需要在类中定义一个静态的类型的私有成员变量
```
private static TaskManager tm = null;  
```
3. 增加一个公有的静态方法，让外界该使用该成员变量并何时实例化该成员变量
```
public static TaskManager getInstance()  
{  
    if (tm == null)  
    {  
        tm = new TaskManager();  
    }  
    return tm;  
}  
```

**单例模式有三个要点**
* 某个类只能有一个实例
* 它必须自行创建这个实例
* 提供本类实例的唯一全局访问点，即提供获取唯一实例的方法

## 并发
由于要对实例进行大量初始化工作，需要一段时间来创建对象。而在此时，如果再一次调用getInstance()方法（通常发生在多线程环境中），由于instance尚未创建成功，仍为null值，判断条件(instance== null)为真值，因此代码instance= new 将再次执行，导致最终创建了多个instance对象

解决方法
### 饿汉式单例类 
由于在定义静态变量的时候实例化单例类，因此在类加载的时候就已经创建了单例对象
```
class EagerSingleton {   
    private static final EagerSingleton instance = new EagerSingleton();   
    private EagerSingleton() { }   
  
    public static EagerSingleton getInstance() {  
        return instance;   
    }     
}  
```

### 懒汉式单例类与线程锁定
Lazy Load: 懒汉式单例在第一次调用getInstance()方法时实例化，在类加载时并不自行实例化,即需要的时候再加载实例

```
class LazySingleton {   
    private static LazySingleton instance = null;   
  
    private LazySingleton() { }   
  
    synchronized public static LazySingleton getInstance() {   
        if (instance == null) {  
            instance = new LazySingleton();   
        }  
        return instance;   
    }  
} 
```
`getInstance()` 方法前面增加了关键字`synchronized`进行线程锁，以处理多个线程同时访问的问题,但是每次调用getInstance()时都需要进行线程锁定判断

Improvement: 我们无须对整个getInstance()方法进行锁定，只需对其中的代码“instance = new LazySingleton();”进行锁定即可

```
public static LazySingleton getInstance() {   
    if (instance == null) {  
        synchronized (LazySingleton.class) {  
            instance = new LazySingleton();   
        }  
    }  
    return instance;   
}
```
问题： 在某一瞬间线程A和线程B都在调用getInstance()方法，此时instance对象为null值，均能通过instance == null的判断

解决方案：Double-Check Locking
```
private volatile static LazySingleton instance = null;   
public static LazySingleton getInstance() {   
        //第一重判断  
        if (instance == null) {  
            //锁定代码块  
            synchronized (LazySingleton.class) {  
                //第二重判断  
                if (instance == null) {  
                    instance = new LazySingleton(); //创建单例实例  
                }  
            }  
        }  
        return instance;   
} 
```
### 指令重排问题
> 指令重排序是JVM为了优化指令，提高程序运行效率，在不影响单线程程序执行结果的前提下，尽可能地提高并行度
使用双重检查锁定来实现懒汉式单例类，需要在静态成员变量instance之前增加修饰符volatile，将会保证对所有线程的可见性. 禁止指令重排序优化。

### 饿汉和饱汉的对比
* 饿汉式单例类在类被加载时就将自己实例化，它的优点在于无须考虑多线程访问问题，可以确保实例的唯一性；从调用速度和反应时间角度来讲，由于单例对象一开始就得以创建，因此要优于懒汉式单例
* 但是无论系统在运行时是否需要使用该单例对象，由于在类加载时该对象就需要创建，因此从资源利用效率角度来讲，饿汉式单例不及懒汉式单例，而且在系统加载时由于需要创建饿汉式单例对象，加载时间可能会比较唱
* 懒汉式单例类在第一次使用时创建，无须一直占用系统资源，实现了延迟加载，
* 懒汉式必须处理好多个线程同时访问的问题，特别是当单例类作为资源控制器，在实例化时必然涉及资源初始化，而资源初始化很有可能耗费大量时间，这意味着出现多线程同时首次引用此类的机率变得较大，需要通过双重检查锁定等机制进行控制，这将导致系统性能受到一定影响。

## Better implemnetation
饿汉式单例类不能实现延迟加载，不管将来用不用始终占据内存；懒汉式单例类线程安全控制烦琐，而且性能受影响

Initialization Demand Holder (IoDH)的技术: 能够将两种单例的缺点都克服，而将两者的优点合二为一
* 在单例类中增加一个静态(static)内部类
* 在该内部类中创建单例对象
* 再将该单例对象通过getInstance()方法返回给外部

```
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
优点
* InnerClass 是一个内部类，其在初始化时是不会被加载的，当用户执行了 getInstance()方法才会加载，同时创建单例对象，所以他也是懒汉式的方法
* 因为 InnerClass 是一个静态内部类，所以只会被实例化一次，从而达到线程安全，
* 因为并没有加锁，所以性能上也会很快，所以一般是推荐
* 
### 总结
* 一次调用getInstance()时将加载内部类HolderClass，在该内部类中定义了一个static类型的变量instance，此时会首先初始化这个成员变量,由Java虚拟机来保证其线程安全性，确保该成员变量只能初始化一次。由于getInstance()方法没有任何线程锁定，因此其性能不会造成任何影响
* 这种方式的实现依赖于JVM对类加载过程中初始化阶段的执行，任何初始化失败都会导致单例类不可用。也就是说，IoDH这种实现方式只能用于能保证初始化不会失败的情况
* 当SingleTon类被JVM加载时，由于这个类没有其他静态属性，其初始化过程会顺利完成。但是内部静态类Holder直到调用getInstance()时才会被初始化
* 当Holder第一次被执行时，JVM会加载并初始化该类。由于Holder含有静态方法INSTANCE，因此会一并初始化INSTANCE
* 所有后序的并发调用getInstance()都会返回一个正确初始化的INSTANCE而不会有额外同步开销

## 反射问题
如果客户端使用反射机制，借助`AccessibleObject.setAccessible(true)`方法，那么就可以用反射的方式调用private修饰的私有方法

方法1: 因为我们反射走的其无参构造，所以在无参构造中再次进行非null判断，加上原来的双重锁定，现在也就有三次判断了
```
private Lazy1(){
    synchronized (Lazy1.class){
        if(lazy1 != null) {
            throw new RuntimeException("反射破坏单例异常");
        } 
    }
}
```
如果两个都是反射实例化出来的，也就是说，根本就不去调用 getInstance() 方法，那可怎么办？
* 解决方案：增加一个标识位，例如下文通过增加一个布尔类型的 ideal 标识，保证只会执行一次，更安全的做法，可以进行加密处理，保证其安全性

 
方法2: 枚举类型
 它不仅能避免多线程同步问题，而且还自动支持序列化机制，防止反序列化重新创建新的对象，绝对防止多次实例化
 ```
 public enum Singleton{
	INSTANCE;
}
 ```
 
 
 


## Summary
### 优点
* 单例模式提供了对唯一实例的受控访问。因为单例类封装了它的唯一实例，所以它可以严格控制客户怎样以及何时访问它
* 由于在系统内存中只存在一个对象，因此可以节约系统资源
* 允许可变数目的实例。基于单例模式我们可以进行扩展，使用与单例控制相似的方法来获得指定个数的对象实例，既节省系统资源，又解决了单例单例对象共享过多有损性能的问题

### 缺点
* 由于单例模式中没有抽象层，因此单例类的扩展有很大的困难。
* 单例类的职责过重，在一定程度上违背了“单一职责原则”。因为单例类既充当了工厂角色，提供了工厂方法，同时又充当了产品角色，包含一些业务方法，将产品的创建和产品的本身的功能融合到一起
* 运行环境都提供了自动垃圾回收的技术，因此，如果实例化的共享对象长时间不被利用，会自动销毁并回收资源，下次利用时又将重新实例化，这将导致共享的单例对象状态的丢失

### 适用场景
* 系统只需要一个实例对象，如系统要求提供一个唯一的序列号生成器或资源管理器，或者需要考虑资源消耗太大而只允许创建一个对象。
* 客户调用类的单个实例只允许使用一个公共访问点，除了该公共访问点，不能通过其他途径访问该实例






