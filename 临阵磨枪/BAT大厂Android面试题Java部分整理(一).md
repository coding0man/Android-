#Java基础知识整理    

- [x] java中==和equals和hashCode的区别
- [x] int、char、long各占多少字节数
- [x] int与integer的区别
- [x] 谈谈对java多态的理解
- [x] String、StringBuffer、StringBuilder区别？  
- [x] 什么是内部类？内部类的作用？成员内部类、静态内部类、局部内部类和匿名内部类的理解，以及项目中的应用？  
- [x] 静态内部类的设计意图？  
- [x] 抽象类和接口区别
- [x] 抽象类的意义
- [x] 抽象类与接口的应用场景
- [x] 抽象类是否可以没有方法和属性？
- [x] 接口的意义
- [x] 泛型中extends和super的区别
- [x] 父类的静态方法能否被子类重写
- [x] 进程和线程的区别
- [x] final，finally，finalize的区别
- [x] 序列化的方式
- [x] Serializable 和Parcelable 的区别
- [x] 静态属性和静态方法是否可以被继承？是否可以被重写？以及原因？
- [x] 谈谈对kotlin的理解
- [x] 闭包和局部内部类的区别
- [x] string 转换成 integer的方式及原理

#Java深入点的知识点
- [ ]哪些情况下的对象会被垃圾回收机制处理掉？
- [ ]讲一下常见编码方式？
- [ ]utf-8编码中的中文占几个字节；int型几个字节？
- [ ]静态代理和动态代理的区别，什么场景使用？
- [ ]Java的异常体系
- [ ]谈谈你对解析与分派的认识。
- [ ]修改对象A的equals方法的签名，那么使用HashMap存放这个对象实例的时候，会调用哪个equals方法？
- [ ]Java中实现多态的机制是什么？
- [ ]如何将一个Java对象序列化到文件里？
- [ ]说说你对Java反射的理解
- [ ]说说你对Java注解的理解
- [ ]说说你对依赖注入的理解
- [ ]说一下泛型原理，并举例说明
- [ ]Java中String的了解
- [ ]String为什么要设计成不可变的？
- [ ]Object类的equal和hashCode方法重写，为什么？

#问题解答部分
##1. java中==和equals和hashCode的区别？
  
> - == 是判断相等运算符，用于比较基本数据类型的值是否相等，或者引用类型对象的内存地址是否相等。  
> - equals 是Object类的一个方法，子类可以重写这个方法，实现特定类的equals方法。常用于判断两个对象的内容是否一致(实际上如果你关注的仅仅是人的年龄，你可以让一个男人equals一个女人。是否equal的规则是由自己定义的，但是一般情况下重写equals方法还是有一些要求的。)。Object类的equals方法的默认实现只是比较了对象的内存地址是否一样（obj1==obj2），所以我们在需要的时候需要自己重写equals方法来判断对象内容是否一致。equals方法的重写原则是: 
> 
> > 
1. **自反性** x.equals(x) = true
2. **对称性** x.eqlas(y) = y.equals(x)
3. **传递性** x.eqlas(y) = true && y.equals(z)=true  则 x.equals(z)=true
4. **一致性** 每次调用返回的结果应该一致（如果参与equals判断的字段没有改变）
5. x.equals(null) = false
6. x==y  则x.equals(y)=true
7. equal objects must have equal hash codes
8. 如果x.equals(y),如果x和y的类型不一致，应该返回false  

> - hashCode 是对象的一个标示值,是根据对象内的值进行运算得到的结果。把一个需要占据更多内存的对象用一个占用更少内存的HashCode值来表示，因此肯定会有信息损失，导致的结果是不同的对象的HashCode值可能会相等。常常用在HashMap或者HashTable中，hashCode方法的规则如下:

>>
1. 一个应用程序一个执行片段（线程）中HashCode值应该不变，但是并不需要保证在两个不同的执行片段中HashCode值也要相等
2. equal的两个对象的HashCode值需要保证相等
3. not equal的两个对象的HashCode值不需要保证不相等，但是如果可以不一致能够提高HashMap等的效率（防止碰撞的发生）
4. Object类的hashCode方法是根据对象在内存中的地址转换出的  

> [我的另外一篇博客有专门写到这个](http://blog.csdn.net/code__man/article/details/79474457)
> 

##2. int、char、long各占多少字节数?
> Java中有八种基本数据类型：  
>> 1. byte 1个字节
>> 2. char 2个字节
>> 3. boolean java没有说明一定是几个字节，可认为是一个字节，毕竟只要一个bit就可以表示true和false两种情况了
>> 4. short 短整型 2个字节
>> 5. int 整型 4个字节
>> 6. long 长整型 8个字节
>> 7. float 浮点数类型 4个字节
>> 8. double 双精度浮点数类型 8个字节
>> 

##3. int与integer的区别？
> - Java中的数据类型分为基本数据类型和引用数据类型，所以有人说Java不是一门纯粹的面向对象语言。int是基本数据类型，Integer是int的包装类。讲到这个不得不说一说自动装箱和自动拆箱。自动装箱和自动拆箱是Java5新增的概念，Java5以前需要使用手动装箱和拆箱。  
> - 自动装箱指的是基本数据类型自动转换成想对应的包装类型，如int转换成Integer会自动调用Integer.valueOf(int a)方法将int值转换成对应的Integer包装类。（**这里有一点值得注意的是：-128~127之间的int值装箱时指向的是缓存对象，不在这里范围内则会新new一个Integer对象**）
>> ```java
>>      public static Integer valueOf(int i) {
>>        if (i >= IntegerCache.low && i <= IntegerCache.high)
>>            return IntegerCache.cache[i + (-IntegerCache.low)];
>>        return new Integer(i);
>>    }
>> ```

> - 自动拆箱指的是包装类型自动转换成对应的基本数据类型的过程。如Integer自动转换成int、Character自动转换成char,分别调用integer.intValue()、character.charValue()。
> - 自动装箱和自动拆箱的发生时机：当需要int值，但是传入的参数是Integer类型时会发生自动拆箱，相反则会发生自动装箱
> - 其他需要额外注意的点：
>> 1. integer += 2,会进行一次自动拆箱和一次自动装箱，使用时需要注意。
>> 2. 发生重载时不会发生自动装箱和拆箱操作
>> 3. int == int ,int == Integer 比较的是值。Integer == Integer比较的是对象地址.
>> 4. 前面说的那个-128~127的问题。贴一段源码自己看哦，小伙伴们
  
> ```java
     /**
     * Returns an {@code Integer} instance representing the specified
     * {@code int} value.  If a new {@code Integer} instance is not
     * required, this method should generally be used in preference to
     * the constructor {@link #Integer(int)}, as this method is likely
     * to yield significantly better space and time performance by
     * caching frequently requested values.
     *
     * This method will always cache values in the range -128 to 127,
     * inclusive, and may cache other values outside of this range.
     *
     * @param  i an {@code int} value.
     * @return an {@code Integer} instance representing {@code i}.
     * @since  1.5
     */
     public static Integer valueOf(int i) {
        if (i >= IntegerCache.low && i <= IntegerCache.high)
            return IntegerCache.cache[i + (-IntegerCache.low)];
        return new Integer(i);
    }
>```    
    
>```java
>    /**
     * Cache to support the object identity semantics of autoboxing for values between
     * -128 and 127 (inclusive) as required by JLS.
     *
     * The cache is initialized on first usage.  The size of the cache
     * may be controlled by the {@code -XX:AutoBoxCacheMax=<size>} option.
     * During VM initialization, java.lang.Integer.IntegerCache.high property
     * may be set and saved in the private system properties in the
     * sun.misc.VM class.
     */
    private static class IntegerCache {
        static final int low = -128;
        static final int high;
        static final Integer cache[];
        static {
            // high value may be configured by property
            int h = 127;
            String integerCacheHighPropValue =
                sun.misc.VM.getSavedProperty("java.lang.Integer.IntegerCache.high");
            if (integerCacheHighPropValue != null) {
                try {
                    int i = parseInt(integerCacheHighPropValue);
                    i = Math.max(i, 127);
                    // Maximum array size is Integer.MAX_VALUE
                    h = Math.min(i, Integer.MAX_VALUE - (-low) -1);
                } catch( NumberFormatException nfe) {
                    // If the property cannot be parsed into an int, ignore it.
                }
            }
            high = h;
            cache = new Integer[(high - low) + 1];
            int j = low;
            for(int k = 0; k < cache.length; k++)
                cache[k] = new Integer(j++);
            // range [-128, 127] must be interned (JLS7 5.1.7)
            assert IntegerCache.high >= 127;
        }
        private IntegerCache() {}
    }
> ``` 

 
##4.谈谈对java多态的理解。
> - 是什么。多态是面向对象的三大特性之一，面向对象的三大特性是：继承、封装和多态。多态指的是：**父类的引用变量 指向子类对象，当调用父类中存在的方法时，实际上会调用子类重写之后的方法（如果有重写。否则调用父类的方法。）**
> - 为什么。在开发中会经常遇到设计一定的准则A，大家都来遵循这个准则A来办事。准则A1、A2...的具体实行留给实施该准则的地方来实现，但是怎么调用到某一个特定的准则A1呢，有两个方法，一个是强制类型转换之后将A转换成A1，然后调用A1重写的规则，再有就是创造一个方法可以使用A直接调用A1代表的准则，就是多态了。
> - 怎么做。多态的实现需要三个条件：  
>
>> - 继承+重写：子类A1继承自父类A，并且重写父类A的方法function1();
>> - 向上转型：使用父类A的引用变量指向A1对象，如<code>A a = new A1();</code>
>> - 使用a.function1()；

>> 当使用a调用function1的时候，实际上调用的是A1重写的function1方法。
 
> - 来个实例：例子是网上流传比较广的例子，找不到出处了，谁知道出处告诉我一下，我把原文链接贴出来。

```java

 public class Main {
    public static void main(String[] args) {
        A a1 = new A();
        A a2 = new B();
        B b = new B();
        C c = new C();
        D d = new D();
        System.out.println("1--" + a1.show(b));
        System.out.println("2--" + a1.show(c));
        System.out.println("3--" + a1.show(d));
        System.out.println("4--" + a2.show(b));
        System.out.println("5--" + a2.show(c));
        System.out.println("6--" + a2.show(d));
        System.out.println("7--" + b.show(b));
        System.out.println("8--" + b.show(c));
        System.out.println("9--" + b.show(d));
    }
    
    public static class A {
        public String show(D obj) {
            return ("A and D");
        }
        public String show(A obj) {
            return ("A and A");
        }
    }
    
    public static class B extends A {
        public String show(B obj) {
            return ("B and B");
        }
        public String show(A obj) {
            return ("B and A");
        }
    }
    
    public static class C extends B {}
    public static class D extends B {}
      
    /**
     * 
     * 下面这是正确答案.你对了几个？
     * 1--A and A
     * 2--A and A
     * 3--A and D
     * 4--B and A
     * 5--B and A
     * 6--A and D
     * 7--B and B
     * 8--B and B
     * 9--A and D
     * 
     */   
 }
```
     
     
     

>下来做下简单说明：   
根据继承关系我们知道: C/D->B->A
根据继承关系关系，我们把类A和类B的方法做一下简化。  

```java
class A{
	代号F1:show(A) 输出 A and A
	代号F2:show(D) 输出 A and D
}

class B extends A{
	代号F3:show(A) 输出 B and A   **重写的方法，覆盖父类的show(A)方法
	代号F4:show(B) 输出 B and B   **新增加的方法
	代号F5:show(D) 输出 A and D   **继承自A类的方法，没有重写。
}

再看一下引用对象：
A a1->A 引用变量和实际对象一致，可以调用类A中的方法(F1,F2)
A a2->B 引用变量和实际对象不一致，做了向上转型，此时a2只能调用继承自A类的方法或者重写了A类的方法，不能调用B类自己新增的方法。(F3,F5)
B b ->B 引用变量和实际对象一致，可以调用类B中的所有方法，包括继承来的、重写的、新增的。(F3,F4,F5)

在进行方法的选择时，首先看能调用到哪些方法，然后在可以调用的方法里寻找是否能完美匹配实参真实类型的，如果没有，向上转型找到最近的一个父类。

1-3都是a1调用方法，a1的变量类型和实际类型一致，可以调用F1,F2
System.out.println("1--" + a1.show(b)); //b是B的实例，但是A类没有show(B)方法，所以需要对b向上转型，然后b是A的子类的实例，所以调用F1,输出-> A and A
System.out.println("2--" + a1.show(c));//c跟B一样需要做向上转型，调用F1,输出-> A and A
System.out.println("3--" + a1.show(d));//d是D的实例，可以直接调用F2,输出A and D
4-6 都是a2调用方法，可以调用F3,F5
System.out.println("4--" + a2.show(b));//b是A的子类的实例，调用F3,输出B and A
System.out.println("5--" + a2.show(c));//c是A的子类的实例，调用F3,输出B and A 
System.out.println("6--" + a2.show(d));//d是D的实例，调用F5,输出A and D
7-9 都是a2调用方法，可以调用F3,F4,F5
System.out.println("7--" + b.show(b));//b是A的子类的实例，实际调用时调用F4,输出B and B
System.out.println("8--" + b.show(c));//c是B的子类的实例，调用F4,输出B and B
System.out.println("9--" + b.show(d));//d是D的实例，调用F5,输出A and D

这样一加分析，是不是就很清楚明了了呢。

```
      
##5.String、StringBuffer、StringBuilder区别?

> **相同点：**String、StringBuilder、StringBuffer都是使用一个char[]保存字符串数据。  
> **不同点：**String 是不可变的，StringBuilder是可变的线程不安全的，StringBuffer是在StringBuilder的基础上加上了线程安全控制，所以是可变的线程安全的。  
> 面试官可能会深入到下面的问题
>> - 字符串池：String是不可变的，共享使用可以提高效率。不可变，所以所有改变字符串的方法都是new了一个新String（可能会借助SB）。contact用的是把原来的数据和新增的数据copy到另一个char[]中，
>> - 内存模型：常量池放在栈内存区，new对象放在堆内存区。<code>String s0 = "abc";String s1 = "abc";String s2 = new String("abc");</code>s0和s1指向的是同一个对象（常量池中的对象）；s1和s2指向的不是同一个对象。
>> - 线程安全：这是另一个话题
>> - new String，String+String，String.replace,String.concat等方法的内部实现
>> - 或者其他的

##6.什么是内部类，内部类的作用？成员内部类、静态内部类、局部内部类和匿名内部类的理解，以及项目中的应用?
**定义：**把一个类定义在另外一个类中，这个类就是内部类。  
**作用(意义)：**内部类可以实现更好的封装；内部类的使用可以实现多重继承；如果一个类需要实现的两个接口中有同名同参数列表的方法（或者父类和接口），也需要借助内部类来实现；内部是面向对象中的闭包。
内部类可以分为下面几种：    
*成员内部类包含_非静态内部类_和_静态内部类_*  
*局部内部类包含_一般局部内部类_和_匿名内部类_* 
 
下面的很多内容都涉及了成员间的相互访问能力，这里有一条准则是一定要牢记的：   
###静态成员不能访问非静态成员！
 
> - 非静态内部类：
> 
>> - **定义：**类定义在外部类的大括号里，作为外部类的成员。非静态内部类不能拥有静态成员（静态方法、静态成员、静态初始化块）。 
>> - **访问外部类：**内部类可以直接访问外部类的所有静态成员和非静态成员（内部类作为外部类的成员，成员间的相互访问是不受访问控制符的限制的）。
>> - **被外部类访问：**外部类需要新建一个内部类的实例才能访问内部类的非静态成员（根据第二条规则，事实上也不会有静态成员）,不受访问控制符的限制。
>> - **被其他类访问：**在外部类以外访问内部类首先需要确定是否有内部类的访问权限，如果可以访问内部类，则需要新建一个外部类的实例，然后用外部类的实例创建内部类对象进行访问。
>> - **作用与应用：**因为非静态内部类不能脱离离开外部类实例单独存在，所以特别适用于内部类是外部类不可分割的一部分，内部类实例离开外部类实例就没有意义的那种情况（如响尾蛇是一个实例，响尾蛇的尾巴是一个实例，响尾蛇的尾巴离开响尾蛇就不会响了，这个尾巴的成员变量和成员方法就失去了意义）。或者添加上访问控制符，不想让内部类脱离外部类使用的情况，如一个RecyclerView的Adapter是一个外部类，Adapter中的ViewHolder是一个内部类，这个ViewHolder脱离里这个Adapter就没有了意义，所以可以给这个ViewHolder添加private访问控制符不让外部类以外的类访问。
>
> - 静态内部类：
> 
>> - **定义：**类定义在外部类的大括号里，作为外部类的成员，并且增加Static关键字修饰。
>> - **访问外部类：**静态内部类可以访问外部类的静态成员变量和静态成员方法，不能访问外部类的非静态成员。
>> - **被外部类访问：**外部类需要新建一个内部类的实例才能访问内部类的非静态成员，或者直接使用内部类类名访问内部类的静态成员。
>> - **被其他类访问：**在外部类以外可以直接使用*外部类名.内部类名*访问内部类的静态成员，或者需要新建一个内部类的实例来访问内部类的非静态成员。
>> - **作用与应用：**<code>//todo 这个我不知道诶</code>我也还不是很明白静态内部类的应用场景呢。
> 
> 
>
> - 局部内部类：
> 
>> - **定义：**类定义在外部类的方法中，并且有自己的类名。局部内部类相当于一个局部变量，所以不能有访问控制符以及static等修饰符。局部内部类的访问（包括对象的创建、方法的调用）局限在外部类的方法内部。
>> - **访问限制：**局部内部类可以访问方法内的局部变量，但是局部变量需要是final类型的（Java8以前）
>> - **作用与应用：** 一般的局部内部类确实用的比较少诶
> 
>> 
> 
> - 匿名内部类
> 
>> - **定义：**匿名内部类继承自一个类或者实现一个接口，使用方法是直接使用new 父类名称或者接口名称就可以创建自己匿名内部类的对象。 
>> - **内部类的构造器：**匿名内部类没有类名，所以无法定义构造器。如果是继承父类的匿名内部类，新建对象是直接使用父类的构造器。
>> - **作用与应用：** 最常见的创建匿名内部类的方式是在回调时需要某个接口类型的对象。
> 
> 



##7.静态内部类的设计意图？ 
如果有一个内部类并不需要保存一个外部类的引用时，可以设计成静态类；  
如果外部类有一个静态方法需要使用到内部类中的某个成员（成员变量或者成员方法），那么也需要把内部类设置成静态的。



##8.抽象类和接口区别?
> - **设计目的：**接口体现的是一种规范和实现相分离的设计哲学，接口定义了一系列的准则供实现类来实现。实现此接口的所有类都遵循了接口所定义的一系列准则，他们内部实现不一样，但是对于调用者而言他们可以提供相同的对外服务。面向接口的耦合是一种低层次的耦合，可以提供更好的拓展性和可维护性。  
> 抽象类则提现的是一种模板模式的设计。抽象类定义了一种模板，完成模板的一些基础实现，其他的细节交给子类去延迟实现。
> - **用法(包含的内容)：**
> 
>>> 1. *变量：*抽象类可以包含任何类型的成员变量；接口只能包含public static final类型的变量。  
>>> 2. *方法：*抽象类可以包含抽象方法或者非抽象方法（可以有方法的实现）；接口只能包含抽象实例方法（java8以后可以包含类方法和默认方法）。
>>> 3. *初始化块和构造器：*抽象类可以包含，接口则不能包含。
>
> - **其他：**抽象类的关键字abstract class,接口的关键字interface。
> 

##9.抽象类的意义?
> 抽象类是值用abstract关键字修饰的类，抽象类中可以包含抽象方法并且不能被实例化，除此之外，抽象类和其他的类并没有本质性的区别（甚至可以把抽象类看成普通类和接口的复合体，或者说是把抽象类看成接口和具体实现类之间的一个过渡，比如说List接口--->AbstractList抽象类--->ArrayList具体实现类）。  
> 
> - 抽象类提取了子类中的公有方法，可以提供方法级别的代码复用；
> - 同时抽象类的实现类必须实现抽象类中定义的抽象方法，向外提供一致的调用方法。
> - 抽象类将子类的公共部分进行提取，将可变部分留给子类去实现，子类可同时拥有父类的能力，也拥有自己新的能力。
> - 抽象类是用于继承的，可以实现多态。
> 

##10.抽象类与接口的应用场景?

> - 从意义上说：抽象类是对一组具有关联（有共同的属性、方法或者意义上有关联等）的类的抽象；接口则会是对一组执行相同标准（如都有List接口的子类都有插入、删除等功能）的类的抽象。从这个层面来说，抽象类适用于子类之间有一些共同的属性或者方法，不需要子类去单独实现（但是子类也可以有自己独特的属性或方法）的时候；而接口适用于子类可能只是需要实现特定的一组功能，对其他的并没有约束。
- 从类之间通讯：接口更加适合用于两个类之间需要通过一个标准进行通讯，具体实现并不关心（面向接口编程）；抽象类则会在如果只靠纯粹的接口无法满足通讯需求（有状态等信息需要保存），可以考虑使用抽象类。
- 从功能上来区分：值得注意的是：

>> - 接口不能拥有实例变量，如果子类之间需要共同的变量，只能使用抽象类
>> - 接口的所有方法都必须要被子类实现，如果你设计的方法只想被需要的时候实现，你应该使用抽象类（Java8以前，Java8以后接口也可以有默认方法 default method）。
>> - 你只需要一组标识，不需要任何方法，应该使用接口（只是使用了接口的变量都是<code>public static final</code> 的这一特性）

##11.抽象类是否可以没有方法和属性？

> 可以。抽象类和实现类之间的唯一区别是抽象类中允许存在抽象方法。
>
##12.接口的意义?

> 接口，在Java中叫Interface，在Object_C中叫**protocol（协议）**，在维基百科上接口的解释是**用于沟通的中介物的抽象**。  
> 我们在生活中经常会遇到一种情况就是我们希望这个东西可以做什么，但是并不想了解具体是怎么做的，比如说电饭锅可以烧饭，不管什么品牌什么型号的电饭锅都可以烧饭，我们并不需要知道电饭锅怎么烧饭的，不同品牌的电饭锅烧饭有什么区别，但是我们都知道只要是电饭锅是肯定是可以烧饭的（如果不能烧饭我们就要去12315投诉了）。  
> 在软件开发中我们也需要对一些共通的内容进行抽象，如果我们经常需要比较两个对象的大小，任意的对象都应该有比较大小的方法，比较的过程由具体的对象施行。如果我们有一个规范A（或者协议）规定了比较大小的方法a，那么我们只需要知道某个类B是否实现了这个接口A，如果实现了，那我们知道B肯定有一个方法B.a是用来比较对象B1、B2的大小的，我们直接调用就好了。
> 

##13.泛型中extends和super的区别

extends 确定了泛型类型的上限。
super 确定了泛型类型的下限。

假如现在有现在的继承关系： 

- Animal extends Object
- Person extends Animal
- Students extends Person
- CollegeStudents extends Student

现在假设有一个List：

```java
List<? extends Person> personList = new ArrayList<>();
//那么List可接受类型只能是Person、Student、CollegeStudent,
//不能是Animal,不能是Object


List<? extends Person> personList = new ArrayList<>();
//那么List可接受类型只能是Animal或者Object
//不能是Person、Student、CollegeStudent

 
```


##14.父类的静态方法能否被子类重写？
不能。这个问题可能是想问你类加载以及对象创建和方法调用的过程，这是一大串内容啊，回头有时间再专门写则这个话题吧。
简单点就可以直接回答：我试了，加上@override之后编译不通过。

##15.进程和线程的区别

**进程：** Process,进程是操作系统进行资源分配（内存等）的单位。一个应用至少拥有一个进程（一个应用也可以开启两个进程），一个进程默认拥有一个线程。
**线程：** Thread,线程是CPU进行调度的单位。隶属于同一个进程的不同线程共享该进程的系统资源，共享一块内存，并发执行的效率更高。线程比进程更加轻量，多个线程之间进行交互更加方便。
##16.final，finally，finalize的区别
**final:** 用于声明属性,方法和类, 分别表示属性不可变, 方法不可覆盖, 类不可继承.  

**finally:** 是异常处理语句结构的一部分，表示总是执行.  

**finalize:** 是Object类的一个方法，在垃圾收集器执行的时候会调用被回收对象的此方法，可以覆盖此方法提供垃圾收集时的其他资源回收，例如关闭文件等. JVM不保证此方法总被调用.
##17.序列化的方式
##18.Serializable 和Parcelable 的区别
Java自定义了两种序列化方式，实现Serializable接口或者Externalizable接口。
Android提供了一种序列化的方式，实现Parcelable接口。
区别是：  

- 实现Serializable接口不需要写任何额外的代码；实现Parcelable接口则需要实现接口里的方法，以及创建一个Creator，来完成格式化。
-  实现Serializable接口序列化的对象适合硬盘存储、网络存储等；实现Parcelable接口适合于在内存中传输。
-  实现Serializable接口实现序列化的过程运用了反射技术，所以效率比较低；实现Parcelable接口则序列化的速度比较快。

##19.静态属性和静态方法是否可以被继承？是否可以被重写？以及原因？
可以被继承，不能被重写。  
Java中的绑定指的是将方法的调用与具体的类和类中的方法做一个连接，绑定分为静态绑定和动态绑定。static静态方法的绑定属于静态绑定，在程序执行前已经由Java编译器或者其他连接器进行了绑定，并不是在运行时进行的绑定。 
 
重写指的是Java子类重写（@Override）和覆盖父类的方法，是多态的基础，多态是发生在运行时的。子类可以创建和父类一样的静态方法，但是这时候不是重写，只是子类也建立了一个和父类一样的静态方法而已，引用变量是什么类型，就调用哪个类型的静态方法。  

更严格意义上说，我觉得Java对静态成员的调用就不该使用实例.静态成员，而应该只允许使用类名.静态成员，这样才更符合静态成员属于类本身而不属于类的实例这一特性。
##20.谈谈对kotlin的理解

我的理解是这玩意儿挺好的，但是也不是特别需要急着学，但是会这个肯定是好的，毕竟是趋势。

说说我用的时候感触比较深的几点。

- 与Java的互通性
- 空安全，再也不用担心空指针了
- 字符串的连接更爽，尤其是多个固定字符串和多个变量的连接，超爽
- 类型推断，定义并且初始化时不需要显式指明变量类型
- 方法可以设置默认参数，挺实用的
- public 变量不需要写getter和setter方法了，虽然以前也可以用工具生成，但是代码总是显得有些臃肿。
- 更好的Lambda表达式


##21.闭包和局部内部类的区别
先说说闭包，可以先参考下[这里,评论也看一下](http://rednaxelafx.iteye.com/blog/245022)  
**闭包的概念：**外部环境持有内部函数所使用的自由变量，由此对内部函数形成了闭包。一个函数的自由变量就是既不是局部变量也不是方法参数的变量。  
在Java中，函数只能在类中定义。所以这里的内部函数一定是在一个类中的。  

个人认为Java中的几种闭包的常见形式：

- 类直接作为外部环境： 如果这个函数直接使用了类中的实例变量，那么函数的返回值既依赖于函数的参数，同时也依赖于类的实例变量，这也是一种闭包。
- 类作为包装，放在另外的类里：这是外部类-非静态内部类的一种闭包形式，内部类持有外部类的引用，如果此时内部类的成员方法调用了外部类的实例变量，则外部类就对该函数实现了简洁的闭包。
- 类作为包装，放在其他的方法里：这是一种方法-局部内部类的闭包形式，局部内部类引用了方法中的局部变量，也就形成了闭包。

**局部内部类的概念：**将类定义在方法、代码块的内部就形成了局部内部类，局部内部类的作用范围只在方法或者代码块的作用范围内有效。

根据前面列出的几种形成闭包的形式可以看出，局部内部类可以构成闭包，闭包的形成并不一定需要局部内部类。局部内部类是形成闭包的非充分非必要条件。


##22.string 转换成 integer的方式及原理。
String转换成Integer需要使用Integer类的Integer.parseInt(String)方法。  
把String转换成Integer的方法需要考虑以下几点：  
我觉得1-4都比较容易判断，第5点稍微有点麻烦，看源码看了半天才明白意思。
> 1. String的是否为null，总长度大于零
> 2. 数字部分的长度是否大于0
> 3. 字符串的第一为是否标示正负，以及数字的正负
> 4. 除正负位以外的其他位是否都是“0-9”之间的数字
> 5. 数字大小是否超过Integer能表示的范围
> 

下面说一下源码中转换的流程。（以转换成10进制为例子，暂时不考虑其他进制）  

> 1. 判断条件1，是null或者长度为0，直接抛出异常。
> 2. 判断条件3，并且标记是正数还是负数，__**同时把正数当做负数来处理，最后再做符号的转换**__。然后判断条件2，数字部分长度小于1直接抛异常。
> 3. 循环
>
>>> 1. 判断条件4，同时拿到当前位代表的数字digit； 
>>> 2. 用**方法一**判断5；（方法一的具体逻辑稍后再说） 
>>> 3. 然后将digit乘以10存为结果result。<code>result *= 10</code>
>>> 4. 用**方法二**判断5；（方法二的具体逻辑稍后再说）
>>> 5. 将原来的result和当前位进行合并。<code>result -= digit</code>。因为前面说了当成负数处理的，所以这里result = result-digit。
>
> 4. 根据之前标记的符号位的正负转换成真实的数字。<code>return 是负数？result:-result</code>
> 
下面来说一下前面预留的几个没有解决的问题：

> 1. 为什么转换成负数处理不转换成正数处理，或者分开处理:
> 
>>  - 负数的标示范围比正数大，所以可以把正数转换成负数来进行运算，都转成正数会出问题.  
>> 说一个常见的数字Integer的范围是-2147483648~2147483647,在判断是否超过最大值的时候需要使用，所以我们用同一个变量来标示边界值（最大或者最小）如果用<code>int boundary = Integer.MAX\_VALUE</code>是没有问题的，但是如果使用<code>int boundary = -Integer.MIN\_VALUE</code>这就出问题了，因为int类型变量不能表示2147483648。（我当时也弄不明白为什么不转成正数，后来才发现原来最小的负数的绝对值比正数大1。）
>> - 个人感觉如果是我自己写代码，我就分开写了，虽然有很多代码是重复的，但是感觉逻辑性和可读性会好很多。但是源码没有这么做。


我来贴一段源代码吧

```java
    public static int parseInt(String s, int radix)
            throws NumberFormatException
    {
        /*
         * WARNING: This method may be invoked early during VM initialization
         * before IntegerCache is initialized. Care must be taken to not use
         * the valueOf method.
         */


        if (s == null) {
            throw new NumberFormatException("null");
        }

        if (radix < Character.MIN_RADIX) {
            throw new NumberFormatException("radix " + radix +
                    " less than Character.MIN_RADIX");
        }

        if (radix > Character.MAX_RADIX) {
            throw new NumberFormatException("radix " + radix +
                    " greater than Character.MAX_RADIX");
        }

        int result = 0;//用来保存结算结果
        boolean negative = false;//是否是负数
        int i = 0, len = s.length();//String下标；数组长度
        int limit = -Integer.MAX_VALUE;//边界值，此处是最小值。把正数的最大值也用负数表示
        int multmin;//  在这里等于limit/10，limit值的除了最后一位以外的几位的值在这里应该是214748364
        int digit;//用来保存String的每一位代表的数字

        if (len > 0) {
            char firstChar = s.charAt(0);
            if (firstChar < '0') { // Possible leading "+" or "-"
                if (firstChar == '-') {
                //如果是负数
                    negative = true;
                    limit = Integer.MIN_VALUE;
                } else if (firstChar != '+')
                    throw NumberFormatException.forInputString(s);

                if (len == 1) // Cannot have lone "+" or "-"
                    throw NumberFormatException.forInputString(s);
                i++;
            }
            multmin = limit / radix;//这个值 后面有用。
            while (i < len) {
                // Accumulating negatively avoids surprises near MAX_VALUE
                //Character.digit获取到字符代表的数字，这里也有一个小算法
                digit = Character.digit(s.charAt(i++),radix);
                if (digit < 0) {
                //如果不是0-9之间的数字 直接抛异常
                    throw NumberFormatException.forInputString(s);
                }
                if (result < multmin) {
                	//这里用的是一种提前判断防止越界的办法，如果在result *= radix之后判断时候比最小值小，则可能存在数值越界的问题，为了避免数值越界的问题，可以直接提前进行比较，就是比较计算之前的数值书否会越界。
                    throw NumberFormatException.forInputString(s);
                }
                result *= radix;
                if (result < limit + digit) {
                	//这里也是一种提前判断的方法,如果result-digit已经比limit还小，就肯定已经超过int能表示的最小的数了。所以要提前做一下比较（提前比较的时候没有赋值，result-digit的类型巴拉巴拉），避免赋值的时候出现越界处理。
                    throw NumberFormatException.forInputString(s);
                }
                result -= digit;//如果没有越界，就把后面的数值加到结果里。
            }
        } else {
            throw NumberFormatException.forInputString(s);
        }
        return negative ? result : -result;//根据正负号返回最终结果。
    }
```

所以这道题，简单描述的答案就是：  
**把字符串的第一个数字拿出来乘以十得到一个数字，再把得到的数字加上第二个数字得到一个数字；再上一轮拿得到的数字乘以十，再加上第三个数字得到一个结果，这个结果作为下一轮的初始值，以此类推。这中间可能会穿插一些其他细节的判断和处理。**