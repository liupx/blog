---
title: Java引用说明
date: 2017-03-06 13:14:00
tags: [Java]
categories: "Java"
---
---
## Java 中为何全是引用
### 以方法传参为例说明
1. 原理说明
  方法参数对应的实例对象，存储在JVM 内存分区的堆中；方法传递的是该实例的引用。实际上，此引用是存储在JVM 内存分区的栈区中，具体在方法对应的栈帧的**局部变量表（Local Variable Table）**中。
所以参数通俗地讲，java 方法中所能获取的是对象引用的一个本地备份。
也就是说 ，以方法1调用方法2，传递 CustomeClass s为例


----------

2. 测试代码
``` java
/**
 * Created by liupx on 2017/03/06.
 */
public class ReferenceParamsInMethod {

    private class CustomeClass{
        private String value="hello";

        public String getValue() {
            return value;
        }

        public void setValue(String value) {
            this.value = value;
        }
    }
    public void father(){
        CustomeClass c=new CustomeClass();
        System.out.println("Origin value of c in father:\t"+c.getValue());
        son(c);
        System.out.println("Final  value of c in father:\t"+c.getValue());
    }
    public void son(CustomeClass c){

        System.out.println("Origin value of c in son:   \t"+c.getValue());
        c=new CustomeClass();
        c.setValue("hello world");
        System.out.println("Final  value of c in son:   \t"+c.getValue());

    }
    
    public static void main(String args[]) {
        ReferenceParamsInMethod test=new ReferenceParamsInMethod();
        test.father();
    }
    
}

```
考虑下图中发几个断点处，c所指向的对象及其成员变量值。
<br>
![ReferenceParamsInMethodCode][1]
<br>
调用方法 son(c) 传递参数 c,此时son中参数对应的实例和father中**实例是同一个**。如下图：
<br>
![ReferenceParamsInMethodDebug1Line19][2]
<br>
![ReferenceParamsInMethodDebug2Line25][3]
<br>
当son修改了引用关系，比如**c=newCustomeClass();**  此时，son中的c对象引用指向了新创建的对象实例。如下图：
<br>
![ReferenceParamsInMethodDebug3Line28][4]
<br>
当son方法调用完毕，其对应的**栈帧出栈**，son中创建的CustomeClass 对象**无指向其的引用，等待GC回收**。与此同时，father方法重新成为**栈顶**，此时上下文（Code Line：21）中，c对象引用c以及其所指向的对象与原来的上下文（Code Line：19）并没有发生任何变化。如下图：
<br>
![ReferenceParamsInMethodDebug4Line21][5]


  [1]: https://raw.githubusercontent.com/liupx/img/master/ReferenceParamsInMethodCode.png
  [2]: https://raw.githubusercontent.com/liupx/img/master/ReferenceParamsInMethodDebug1Line19.png
  [3]: https://raw.githubusercontent.com/liupx/img/master/ReferenceParamsInMethodDebug2Line25.png
  [4]: https://raw.githubusercontent.com/liupx/img/master/ReferenceParamsInMethodDebug3Line28.png
  [5]: https://raw.githubusercontent.com/liupx/img/master/ReferenceParamsInMethodDebug4Line21.png
