# Java 反射机制
### 概述
通俗易懂的来说，反射是可以理解为 Java 运行时的一本说明书。这本说明书里面，记录着被加载到 Java 虚拟机中 Class 对象的名字、域、方法信息，无论这些信息是私有的还是公有的。我们可以通过这本说明书，来在运行时了解一个类的信息，然后根据这些信息，再进行动态的调用。
### 作用
- 可以在运行时**探明、加载和使用**编译期间完全未知的类
- 可以根据类的名称，动态加载一个类，而不需要在编码期间就确定调用哪个类
- 如果一个类已经加载，可以利用反射知道这个类的所有属性和方法
- 对于任意一个对象，都能调用它的方法和属性，甚至可以动态修改它的访问权限
### 作用机制
Java虚拟机加载完类之后，会在堆内存中产生一个 ==Class== 类型的对象（每个类只有一个Class对象），这个对象包含了完整的类的结构信息。这也是 Java 反射得名的原因——这个Class包含了完整的类的结构信息，就像一面镜子。

p.s. Java 的八种原始类型以及关键字 void 也会被视为 Class 对象。
### 获取 Class 对象
假设有一个类 Car，一个类型为 Car 的对象 car，我们有以下三种方法获取 Class 对象：
- 通过调用对象的 ==getClass()== 方法 ``` car.getClass() ```
- 通过类的 ==.class==属性 (最安全且性能最佳) ``` Car.class ```
- 通过 ==Class.forName(String className)== 动态加载 ``` Class.forName("Car") ```
### 怎么利用 Class 类
==java.lang.reflect== 包中有三个类 Field 、 Method 和 Constructor，分别用于描述类的域、方法和构造器。

我们可以从==Class==类中获取包括构造函数、方法、属性、修饰符等信息，下面是常用的方法列表：

方法的作用 | 方法的签名 | 方法的包
---|---|---
返回一个类的新实例 | Object newInstance() | java.lang.Class 1.0
构造一个这个构造器所属类的新实例 | Object newInstance(Object[] args) | java.lang.reflect.Constructor 1.1
返回这个类和其超类的公有域 | Field[] getFields() | java.lang.Class 1.0
返回这个类的全部域 | Field[] getDeclaredFields() | java.lang.Class 1.0
返回这个类和其超类的公有方法 | Method[] getMethods() | java.lang.Class 1.0
返回这个类的全部方法 | Method[] getDeclaredMethods() | java.lang.Class 1.0
返回这个类的公有构造器 | Constructor[] getConstructors() | java.lang.Class 1.0
返回这个类的所有构造器 | Constructor[] getDeclaredConstructors() | java.lang.Class 1.0
调用一个对象所描述的方法 | Object invoke(Object implicitParameter, Object[] explicitParameters) | java.lang.reflect.Method 1.1
... | ... | ...
更多方法请查阅 java.lang.reflect 文档
### 创建对象
通过反射来生成对象的方式有两种:
- 使用 ==Class== 对象的 ==newInstance()== 方法来创建该 ==Class== 对象对应类的实例(这种方式要求该Class对象的对应类有默认构造器).
- 先使用 ==Class== 对象获取指定的 ==Constructor== 对象, 再调用Constructor对象的 ==newInstance()== 方法来创建该 Class 对象对应类的实例(通过这种方式可以选择指定的构造器来创建实例).
### 调用方法
当获取到某个类对应的 Class 对象之后, 就可以通过该 Class 对象的 ==getMethod== 来获取一个 Method 数组或 Method 对象。每个 Method 对象对应一个方法，在获得 Method 对象之后，就可以通过调用 ==invoke== 方法来调用该 Method 对象对应的方法.
### 访问成员变量
通过 ==Class== 对象的的 ==getField()== 方法可以获取该类所包含的全部或指定的成员变量 Field ，Field 提供了如下两组方法来读取和设置成员变量值.
- ==getXxx(Object obj)==: 获取obj对象的该成员变量的值, 此处的Xxx对应8中基本类型,如果该成员变量的类型是引用类型, 则取消get后面的Xxx;
- ==setXxx(Object obj, Xxx val)==: 将obj对象的该成员变量值设置成val值.此处的Xxx对应8种基本类型, 如果该成员类型是引用类型, 则取消set后面的Xxx;
### 具体应用场景
利用反射实现工厂模式，从此，我们将对象的实例化操作，分离到了工厂类中去实现。我们可以利用反射实现的工厂模式，对编程进行解耦操作。如下，将 Car 对象的实例化操作，交给了 CarFactory 实现，我们只需要将所需要实例化的对象的类名，以 String 形式传入工厂即可：

```
package Reflect;

interface Car {
    public void accelerate();
    public void brake();
}

class Cadillac implements Car {
    public void accelerate() {
        System.out.println("凯迪拉克加速");
    }
    public void brake() {
        System.out.println("凯迪拉克刹车");
    }
}

class Buick implements Car {
    public void accelerate() {
        System.out.println("别克加速");
    }
    public void brake() {
        System.out.println("别克刹车");
    }
}

class CarFactory {
    public static Car getInstance(String CarClassName) {
        Car car = null;
        try {
            car = (Car) Class.forName(CarClassName).newInstance();
        }
        catch(Exception e) {
            e.printStackTrace();
        }
        return car;
    }
}

class Test {
    public static void main(String[] args) {
        Car car = CarFactory.getInstance("Reflect.Cadillac");
        if(car != null){
            car.accelerate();
            car.brake();
        }
        car = CarFactory.getInstance("Reflect.Buick");
        if(car != null){
            car.accelerate();
            car.brake();
        }
    }
}
```
刘丰璨 写于 2018.04.06 21:00

参考目录:
1. 《Java 核心技术 卷 I 》
2. [《Java 反射》](https://blog.csdn.net/zjf280441589/article/details/50453776)
3. [《利用反射机制实现工厂模式》](https://blog.csdn.net/qq_27093465/article/details/52256224)