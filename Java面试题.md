#Java基础面试题    

- [ ] java中==和equals和hashCode的区别
- [ ] int、char、long各占多少字节数
- [ ] int与integer的区别
- [ ] 探探对java多态的理解
- [ ] String、StringBuffer、StringBuilder区别
- [ ] 什么是内部类？内部类的作用
- [ ] 抽象类和接口区别
- [ ] 抽象类的意义
- [ ] 抽象类与接口的应用场景
- [ ] 抽象类是否可以没有方法和属性？
- [ ] 接口的意义
- [ ] 泛型中extends和super的区别
- [ ] 父类的静态方法能否被子类重写
- [ ] 进程和线程的区别
- [ ] final，finally，finalize的区别
- [ ] 序列化的方式
- [ ] Serializable 和Parcelable 的区别
- [ ] 静态属性和静态方法是否可以被继承？是否可以被重写？以及原因？
- [ ] 静态内部类的设计意图
- [ ] 成员内部类、静态内部类、局部内部类和匿名内部类的理解，以及项目中的应用
- [ ] 谈谈对kotlin的理解
- [ ] 闭包和局部内部类的区别
- [ ] string 转换成 integer的方式及原理

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
  
> - == 是相等运算符，用于比较基本变量的值是否相等，当比较的对象是对象类型时比较的是对象的内存地址  
> - equals 是Object类的一个方法，长用于判断两个对象的内容是否一致。Object类的equals方法的默认实现只是比较了对象的内存地址是否一样，所以我们需要重写equals方法来判断对象内容是否一致。equals方法的重写原则是: 
> 
> > 
1. 自反性 x.equals(x) = true
2. 对称性 x.eqlas(y) = y.equals(x)
3. 传递性 x.eqlas(y) = true && y.equals(z)=true  则 x.equals(z)=true
4. 一致性 每次调用返回的结果应该一致（如果参与equals判断的字段没有改变）
5. x.equals(null) = false
6. x==y  则x.equals(y)=true
7. equal objects must have equal hash codes
8. 如果x.equals(y),如果x和y的类型不一致，应该返回false  

> - hashCode 为对象提供一个标示值，常常用在HashMap或者HashTable中，hashCode方法的规则如下:

>>
1. 一个应用程序一个执行片段（线程）中HashCode值应该不变，但是并不需要保证在两个不同的执行片段中HashCode值也要相等
2. equal的两个对象的HashCode值需要保证相等
3. not equal的两个对象的HashCode值不需要保证不相等，但是如果可以不一致能够提高HashMap等的效率
4. Object类的hashCode方法是根据对象在内存中的地址转换出的
> 我的另外一篇博客有专门写到这个 [博客地址]()
