---
title: Java解析与分派
date: 2017-03-06 17:00:00
categories: "Java"
tags: [Java,JVM]
---
> 未完待续
本文论述JAVA方法调用过程中的两个概念：<font style="color:#A52A2A">**解析与分派**</font>。因为java编译器并不进行链接过程（Linking），而是将这一过程放在了类加载阶段。方法调用时，“方法入口”以<font style="color:#A52A2A">**符号引用**</font>的形式保存在编译阶段生成的class文件的常量池中。而如何将这一符号引用转换为直接引用（即<small>实际运行时内存布局中的入口地址</small>）,分为解析和分派两种策略。

### 解析（Resolution）
#### 概念与原理

如前所述，解析过程指的是类加载期间，将编译时class常量池中一些符合要求的方法引用，直接转化为实际的地址。这里的要求指的是，方法满足 <font style="color:#A52A2A">**编译期间可
知，运行期间不可修改** </font> 满足的方法包括：<font style="color:#A52A2A">**静态方法、私有方法、实例构造器和父类方法**</font>
> 解析调用一定是个静态的过程，在编译期间就完全确定，在类装载的解析阶段就会把涉及的符号引用全部转变为可确定的直接引用，不会延迟到运行期再去完成。

### 分派（Dispatch）
#### 概念与原理
分派意味着实际翻译后的方法地址存在多个选择。如何做出这重选择，分为<font style="color:#A52A2A">**静态分派**</font>和<font style="color:#A52A2A">**动态分派**</font>。
**静态类型（Static Type orApparent Type）与实际类型（Actual Type）**
形如 
```java
Animal cat = new Cat();
```
其中，Animal 成为cat的静态类型，Cat是实际类型。
> 依赖静态类型来定位方法执行版本的分派动作，都称为静态分派。静态分派典型应用就是方法重载（Overload）。静态分派发生在编译阶段，因此确定静态分派的动作实际上不是由虚拟机来执行的。

这意味着方法重载时，在编译阶段，Javac编译器就根据方法接收者的<font style="color:#A52A2A">**静态类型**</font>和<font style="color:#A52A2A">**方法参数**</font>两个宗量来决定使用哪个重载版本。因此，<font style="color:#A52A2A">**JAVA的静态分配是多分派的**</font>。
> 根据实际类型来定位方法执行版本的分派动作，都成为动态分派。静态分派的典型应用是方法重写（Override）。动态分派在运行期使用invokevirtual指令进行多态查找。

 **invokevirtual指令多态查找过程**：
>  - 找到操作数栈顶的第一个元素所指向的对象的实际类型，记为C。
 - 如果在类型C中找到与常量中的描述符合简单名称都相符的方法，则进行访问权限校验，如果通过则返回这个方法的直接引用，查找过程结束；如果权限校验不通过，返回java.lang.IllegalAccessError异常。
 - 否则，按照继承关系从下往上一次对C的各个父类进行第2步的搜索和验证过程。
 - 如果始终没有找到合适的方法，则抛出 java.lang.AbstractMethodError异常。

#### 静态分派测试代码
```java
class Human{  
}    
class Man extends Human{  
}  
class Woman extends Human{  
}  
  
public class StaticPai{  
  
    public void say(Human hum){  
        System.out.println("I am human");  
    }  
    public void say(Man hum){  
        System.out.println("I am man");  
    }  
    public void say(Woman hum){  
        System.out.println("I am woman");  
    }  
  
    public static void main(String[] args){  
        Human man = new Man();  
        Human woman = new Woman();  
        StaticPai sp = new StaticPai();  
        sp.say(man);  
        sp.say(woman);  
    }  
}  
```
代码输出：
 I am human
 I am human
实际上动态分派执行之前，在编译期间，编译器已经根据静态类型和方法参数进行过静态分派。此时只需要再根据运行时进一步确定<font style="color:#A52A2A">**方法接收者的实际类型**</font>即可。所以<font style="color:#A52A2A">**JAVA的 动态分派属于单分派类型**</font>。
#### 动态分派测试代码
```java
class Eat{  
}  
class Drink{  
}  

class Father{  
    public void doSomething(Eat arg){  
        System.out.println("爸爸在吃饭");  
    }  
    public void doSomething(Drink arg){  
        System.out.println("爸爸在喝水");  
    }  
}  

class Child extends Father{  
    public void doSomething(Eat arg){  
        System.out.println("儿子在吃饭");  
    }  
    public void doSomething(Drink arg){  
        System.out.println("儿子在喝水");  
    }  
}  

public class SingleDoublePai{  
    public static void main(String[] args){  
        Father father = new Father();  
        Father child = new Child();  
        father.doSomething(new Eat());  
        child.doSomething(new Drink());  
    }  
}    
```
代码输出：
爸爸在吃饭
儿子在喝水
### 综述
可以说，JAVA是一种<font style="color:#A52A2A">**静态多分派，动态单分派**</font>的语言。<font style="color:#A52A2A">**静态方法和私有方法等非虚方法使用解析；重载对应静态分派；重写对应动态分派**</font>。
