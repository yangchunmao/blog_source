---
title: Java ClassLoader
tags:
---

>在我们的应用中会编写很多`*.java`文件,`java`的虚拟机`jvm`是如何加载并关联相关依赖?这里就需要`ClassLoader`类的作用.
### Class文件
`java`是强类型的静态语言,`jvm`并不能直接解析`.java`文件,所以需要通过`javac`命令编译成`.class`文件,然后才可通过`jvm`进行解析,并运行,举例:

    package com.haier.loader;
    public class HelloWorld {
        public static void main(String[] args) {
            System.out.println("Hello world!");
        }
    }

通过`javac -d ./out com/haier/loader/HelloWorld.java`编译成`.class`字节码文件;
然后找到相应目录加载`jvm`中执行, 命令`java -cp ./out com.haier.loader.HelloWorld`, 结果打印`Hello world!`.另外用`C`或`Python`编写的程序正确转换为`.class`文件后,同样可装载到`jvm`中执行.

### Class文件初始化顺序
对于普通的`java`程序,一般都不需要显式的声明来动态加载`java`类,只需要用`import`关键字将相关联的类引入,类被第一次调用的时候,就会被加载初始化.那对于一个类对象,其内部各组成部分的初始化顺序又是如何的呢?
一个`java`类对象在初始化的时候必定是按照一定顺序初始化其静态快、静态属性、类内部属性、构造方法.这里我们讨论的初始化分别针对两个对象,一个是类本身还有一个类实例化的对象.举例:

    package com.haier.loader;
    public class HelloWorld {
        public static class Inner{
            /**
             * 静态内部类里初始化实例
             */
            public final static HelloWorld HELLO_WORLD = new HelloWorld(3);
            static {
                System.out.println("HelloWorld Inner Static!");
            }
        }
        public static HelloWorld getHello(){
            return Inner.HELLO_WORLD;
        }
        static {
            System.out.println("HelloWorld Static !");
        }
        /**
         * 静态属性
         */
        public static HelloWorld hello = new HelloWorld(1);
        public HelloWorld(int i){
            System.out.println("HelloWorld " + i +" Construct!");
        }
        public static void main(String[] args) {
            
            HelloWorld helloWorld = new HelloWorld(2);
            HelloWorld.getHello();

        }
    }

执行后的结果如下

    HelloWorld Static !
    HelloWorld 1 Construct!
    HelloWorld 2 Construct!
    HelloWorld 3 Construct!
    HelloWorld Inner Static!

从结果可以得出如下结论, **类内部静态块 > 类静态属性 > 类内部属性 > 类构造函数**;还可以通过类**内部静态类**的加载顺序可以看出,只有在调用静态类的方法的时候才初始化,这样的特性可以实现优雅的单例模式.