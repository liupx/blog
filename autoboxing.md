---
title: Java 自动装箱  Auto Boxing
date: 2017-02-14 12:00:00
categories: "Java"       
tags: [Java]              
---
## 自动装箱
### 概念：
> 自动装箱就是Java自动将原始类型值转换成对应的对象，比如将int的变量转换成Integer对象，这个过程叫做装箱，反之将Integer对象转换成int类型值，这个过程叫做拆箱。
因为这里的装箱和拆箱是自动进行的非人为转换，所以就称作为自动装箱和拆箱。

<table>
<tbody>
<tr><th>原始类型</th><th>封装类</th></tr>
<tr><td>byte</td><td>Byte</td></tr>
<tr><td>short</td><td>Short</td></tr>
<tr><td>char</td><td>Character</td></tr>
<tr><td>int</td><td>Integer</td></tr>
<tr><td>long</td><td>Long</td></tr>
<tr><td>float</td><td>Float</td></tr>
<tr><td>double</td><td>Double</td></tr>
<tr><td>boolean</td><td>Boolean</td></tr>
</tbody>
</table>

### 使用时需要注意的问题：
> 循环中效率问题<br>
> 等值比较问题<br>
> 对象类型使用前必须初始化<br>
> 重载时是否自动装箱

### 测试代码
```java
/**
 * Created by liupx on 2017/1/13.
 */
import java.lang.*;

public class AutoboxingTest {

    //测试方法调用与装箱
    public static Integer boxing(Integer n){
        System.out.println("方法调用：\n自动装箱将 int 转化为 Integer："+n);
        return n;
    }

    //测试初始化
    private static Integer init;

    //测试重载与装箱
    public void test(int n){
        System.out.println("原始类型 int n="+n);
    }
    public void test(Integer n){
        System.out.println("对象 Integer n="+n);
    }


    public static void main(String args[]) {
        /**
         * 自动装箱使用情况
         * 1：赋值
         * 2：方法调用
         * */
        Integer n1=7;//自动装箱：原始类型 → 对象
        System.out.println("Integer n1 class="+ n1.getClass());

        int n2=n1;//自动拆箱
        System.out.println("int n2= "+n2);

        boxing(99);//自动装箱
        int n3=boxing(88);//自动拆箱

        /**
         * 自动装箱需注意的几点：
         * 1.循环效率问题
         * 2.等值比较
         * 3.对象类型的未初始化
         * 4.重载时不发生装箱
         * */

        //循环效率
        int count=10000000
                ;
        Integer n4=0;
        long start = System.currentTimeMillis();
        for(int i=0;i<count;i++){
            n4+=1;
            //代码等效于
            //int tem = n4.intValue() + i;
            //Integer n4 = new Integer(tem);
        }

        long end=System.currentTimeMillis();
        System.out.println("自动装箱循环运行用时："+(end-start)+" ms");
        System.out.println("n4="+n4);
        long s1=System.currentTimeMillis();

        int n5=0;
        for(int j=0;j<count;j++){n5+=1;}
        long e1=System.currentTimeMillis();
        System.out.println("不装箱循环运行用时"+(e1-s1)+" ms");
        System.out.println("n5="+n5);

        //等值比较
        // 无装箱
        int n6 = 1;
        int n7 = 1;
        System.out.println("n6==n7 : " + (n6 == n7)); // true

        Integer n8 = 1; //自动装箱
        int n9 = 1;
        System.out.println("n8 == n9 : " + (n8 == n9)); // true

        // 对象类型比较
        /**注：
         * Java会对-128到127的Integer对象进行缓存，
         * 当创建新的Integer对象时，
         * 如果符合这个这个范围，并且已有存在的相同值的对象，
         * 则返回这个对象，否则创建新的Integer对象。*/
        Integer obj1 = 1; // 自动装箱
        Integer obj2 = 1; // 有JVM缓存,所以obj1和obj2指向同一个对象

        Integer obj3=129;//自动装箱
        Integer obj4=129;//无缓存
        //显示各个对象实例
        System.out.println("obj1:"+obj1);
        System.out.println("obj2:"+obj2.hashCode());
        System.out.println("obj3:"+obj3.hashCode());

        System.out.println("obj4:"+obj4.hashCode());

        System.out.println("obj1 == obj2 : " + (obj1 == obj2)); // true
        System.out.println("obj3 == obj4 : " + (obj3 == obj4)); // false

        // Example 4: equality operator - pure object comparison
        Integer obj5 = new Integer(1); // no autoboxing
        Integer obj6 = new Integer(1);
        System.out.println("one == anotherOne : " + (obj5 == obj6)); // false


        //重载时不发生自动装箱
        AutoboxingTest test = new AutoboxingTest();
        int n10 = 666;
        test.test(n10); //不自动装箱
        Integer n11 = n10;
        test.test(n11); //不自动装箱


        //未初始化异常：NullPointerException
        System.out.println("init:"+init);

        if(init<=0){
            System.out.println("Count is not started yet"+init);
        }
    }
}
```
### 说明
> 其中 等值比较时，java 对-128到127的Integer对象进行缓存。从debug 结果可以看到
当Integer 值为1时，符合上述范围有JVM缓存,所以obj1和obj2指向同一个对象


![autoboxing-debug](https://github.com/liupx/img/blob/master/autoboxing-debug.png?raw=true)
### 测试输出
```java
方法调用：自动装箱将 int 转化为 Integer：88
自动装箱循环运行用时：131 ms
n4=10000000
不装箱循环运行用时0 ms
n5=10000000
n6==n7 : true
n8 == n9 : true
obj1:1
obj2:1
obj3:129
obj4:129
obj1 == obj2 : true
obj3 == obj4 : false
one == anotherOne : false
原始类型 int n=666
对象 Integer n=666
init:null
Exception in thread "main" java.lang.NullPointerException
	at AutoboxingTest.main(AutoboxingTest.java:120)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:57)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:601)
	at com.intellij.rt.execution.application.AppMain.main(AppMain.java:144)

Process finished with exit code 1
```
> 参考文章：[Java中的自动装箱与拆箱](http://droidyue.com/blog/2015/04/07/autoboxing-and-autounboxing-in-java/index.html)

