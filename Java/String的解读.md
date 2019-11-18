String,StringBuilder,StringBuffer,常量池，堆，常量，变量

常量池：常量池属于类信息的一部分，而类信息反映到 JVM 内存模型中对应于方法区，也就是说，常量池位于方法区。常量池主要存放两大常量：字面量(Literal) 和 符号引用(Symbolic References)。其中，字面量主要包括字符串字面量，整型字面量 和 声明为final的常量值等；  
而符号引用则属于编译原理方面的概念，包括了下面三类常量：

- 类和接口的全限定名
- 字段的名称和描述符
- 方法的名称和描述符  
  
堆：Java对象的存放区域
常量：final描述的变量。对于基本类型而言，内存地址不变则其值也不变。对于对象而言，内存地址不变，则引用也不变，但是其代表的内容却有可能改变，通过反射或者有多层引用。
final List<String> stringList = new ArrayList<>;
stringList永远指向内存中的地址0x0001，然而0x0001代表的内容却有可能改变，list的长度可变，list的内容同样可变。

String：String是不可变量，String类型的变量如果值变了则地址一定变。

两个操作：+ 和 new String();
+通过StringBuilder来进行字符串的拼接。最后通过StringBuilder的toString方法返回一个代表该值的字符串地址。toString方法中通过一个new String方法新建一个对象后返回对象地址。
new String:在堆区新建一个对象，返回该地址。同时检查常量池中是否存在该值，不存在则放入常量池。  

常量的编译期优化：
final s0="def";
s1 = "abc"+s0;
s0的值在编译期就能确定，上面的两行代码会在编译期进行优化，s1指向常量池中值为abcdef的对象地址。如果去掉s0的final修饰，则使用stringBuilder的append方法以及toString方法构造，在堆中新建一个对象返回给s1.