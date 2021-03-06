# 注解的概念

## 注解和注释
注释是给人看的，注解是给机器（编译器，虚拟机）看的。  


## 注解的概念
注解是一个标识，或者说是标签。

注解是一系列元数据，它提供数据用来解释程序代码，但是注解并非是所解释的代码本身的一部分。注解对于代码的运行效果没有直接影响。  
注解有许多用途，主要如下： 
- 提供信息给编译器： 编译器可以利用注解来探测错误和警告信息 
- 编译阶段时的处理： 软件工具可以用来利用注解信息来生成代码、Html文档或者做其它相应处理。 
- 运行时的处理： 某些注解可以在程序运行的时候接受代码的提取
  
使用注解大概有以下好处
- 提高我们的开发效率
- 更早的发现程序的问题或者错误
- 更好的增加代码的描述能力
- 更加利于我们的一些规范约束
- 提供解决问题的更优解

## 注解长什么样,怎么用，如何生效，效果是什么

需要提前说的是，单单为一个类/一个方法/一个变量/等添加注解，完全不会影响代码地运行。重要的是程序对注解进行提取，根据提到的注解做一定的事情才会起作用（提取以及响应可以由编译器、框架代码或者开发者自己处理，一般框架只会处理框架定义的注解，开发人员也只会处理自己定义的注解）。

先看一个Java基本注解 : @Override
```java
//该注解的定义是这样的
@Target(ElementType.METHOD)//标明该注解的使用对象是方法
@Retention(RetentionPolicy.SOURCE)//标明该注解的生效场景是在源码中
    public @interface Override {
}

//注解的使用是这样的
@Override
public void methodDefineInParentType(){
     // your code...
}
```
- 如何生效
    > 前面注释说到了该注解仅在源码中生效，由编译器识别注解，并且给予开发者提示

- 效果是什么（ps：父类定义了该方法=返回值、方法名、形参列表一致，访问控制符符合规范）
>- 子类方法添加@Override注解时：
>>- 如果父类定义了该方法（返回值，方法名，形参列表一致），则没有什么效果，仅用于提示。
>>- 如果父类没有该方法，则编译器给出提示，并且编译器不允许编译通过（其实编译器也可以允许编译通过，即使通过，也不影响代码的运行效果）
>- 子类方法没有添加@Override注解时：
>>- 如果父类也定义了该方法，则相当于默默加了该注解，没有什么特殊效果。
>>- 如果父类没有定义该方法，则不会给出错误提示，完全当做是一个新方法

一个简单的自定义注解：

```java
/**
 * 自定义注解跟系统定义的注解都一样，可以使用元注解对注解进行说明
 * Target = ElementType.TYPE 说明了该注解可以作用在类和接口上
**/
@Target(ElementType.TYPE)//可以用在类、接口、枚举类上
@Retention(RetentionPolicy.RUNTIME)//保留到运行时
public @interface UselessAnnotation{
        String stringValue() default "test";//添加了一个成员变量，变量的默认值是test
}

//使用
@UselessAnnotation(stringValue = "helloWorld")
public class MainActivity extends AppCompatActivity {
        ...

        //1. 判断MainActivity.class上是否存在特定类型的注解
        //2. 获取MainActivity.class上是特定类型的注解
        //3. 获取注解的变量值并且赋值给一个textView
        if (MainActivity.class.isAnnotationPresent(UselessAnnotation.class)) {
            UselessAnnotation annotation = MainActivity.class.getAnnotation(UselessAnnotation.class);
            String string = annotation.stringValue();
            textView.setText(string);
        }  

        ...
}
```

# 常用注解
我们常用的注解主要集中在下面几个包中：
- java.lang.annotation
    > 定义Java中的五种元注解，以及元注解的参数可选值
- java.lang
    > 定义Deprecated、Override、SuppressWarnings等常用注解
- android.annotation
    > 定义了SuppressLint和TargetApi两个注解
- android.support.annotation
    > 定义安卓中用到的注解
  
## 所在包 java.lang.annotation
该包下定义了Java中的五种元注解。  
作用在注解上的注解叫做元注解。  
Java提供的元注解有五种：
- @Retention : 设置保留期
- @Target ：设置注解的作用对象
- @Documented ：如果设置，则该注解会在javaDoc中呈现
- @Inherited ：如果设置，则该注解可被继承
- @Repeatable ：设置注解可重复，一个元素上可以被设置多个该注解

### @Retention
retention用于说明注解保留的时间
- RetentionPolicy.SOURCE
    > 注解只在源码中存在，保留到编译之前，在编译时就扔掉了
- RetentionPolicy.CLASS
    > 保留到编译器进行编译时，编译时注解可以在编译时被注解处理器进行处理,然后插入生成代码之类的（这也是很多编译时注解框架使用的方法）。
    但是不会保留到运行时
- RetentionPolicy.RUNTIME
    > 保留到Java字节码文件中，并且被加载到虚拟机中，故而可以在运行时通过反射获取

```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.ANNOTATION_TYPE)
public @interface Retention {
    /**
     * Returns the retention policy.
     * @return the retention policy
     */
    RetentionPolicy value();
}

public enum RetentionPolicy {
    /**
     * Annotations are to be discarded by the compiler.
     * 注解只在源码中存在，保留到编译之前，在编译时就扔掉了
     */
    SOURCE,

    /**
     * Annotations are to be recorded in the class file by the compiler
     * but need not be retained by the VM at run time.  This is the default
     * behavior.
     * 保留到编译器进行编译时，编译时注解可以在编译时被注解处理器进行处理,然后插入生成代码之类的（这也是很多编译时注解框架使用的方法）。
     * 但是不会保留到运行时
     */
    CLASS,

    /**
     * Annotations are to be recorded in the class file by the compiler and
     * retained by the VM at run time, so they may be read reflectively.
     * 保留到Java字节码文件中，并且被加载到虚拟机中，故而可以在运行时通过反射获取
     *
     * @see java.lang.reflect.AnnotatedElement
     */
    RUNTIME
}
```
### @Target

Target 是目标的意思，@Target 指定了注解运用的地方。

@Target 有下面的取值

- ElementType.ANNOTATION_TYPE 可以作用于注解
- ElementType.CONSTRUCTOR 可以作用于构造方法
- ElementType.FIELD 可以作用于属性
- ElementType.LOCAL_VARIABLE 可以作用于局部变量
- ElementType.METHOD 可以作用于方法
- ElementType.PACKAGE 可以作用于包
- ElementType.PARAMETER 可以作用于参数
- ElementType.TYPE 可以作用于一个类型，比如类、接口、枚举

### @Documented
顾名思义，这个元注解肯定是和文档有关。它的作用是能够将注解中的元素包含到 Javadoc 中去

```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.ANNOTATION_TYPE)
public @interface Documented {
}
```

```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.ANNOTATION_TYPE)
public @interface Target {
    /**
     * Returns an array of the kinds of elements an annotation type
     * can be applied to.
     * @return an array of the kinds of elements an annotation type
     * can be applied to
     */
    ElementType[] value();
}
```

### @Inherited
如果一个超类被 @Inherited 注解过的注解进行注解的话，那么如果它的子类没有被任何注解应用的话，那么这个子类就继承了超类的注解。 
```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.ANNOTATION_TYPE)
public @interface Inherited {
}
```

### @Repeatable
@Repeatable 是 Java 1.8 才加进来的，所以算是一个新的特性。
什么样的注解会多次应用呢？通常是注解的值可以同时取多个(这似乎可以被数组取代)。暂时不明白它的独特点。

```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.ANNOTATION_TYPE)
public @interface Repeatable {
    /**
     * Indicates the <em>containing annotation type</em> for the
     * repeatable annotation type.
     * @return the containing annotation type
     */
    Class<? extends Annotation> value();
}
```

## 所在包 java.lang

定义了四个Java标准注解：

- @Deprecated 常用于标记某个类或者某个方法已经被废弃
- @FunctionalInterface 标记某个类或者接口是一个函数式接口
- @Override 标记当前方法是重写的父类方法
- @SafeVarargs 标明在可变长参数中的泛型是类型安全的
- @SuppressWarnings 让编译器忽略某种类型的警告

### @Deprecated 

> 标明一个元素已经被废弃，即将被删除。虽然现在还能用，但是不建议用，一般会给出替代的元素。可以设置在包、类、方法、变量、参数等上面。  

```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(value={CONSTRUCTOR, FIELD, LOCAL_VARIABLE, METHOD, PACKAGE, PARAMETER, TYPE})
public @interface Deprecated {
}   
```    
    
### @FunctionalInterface
> 标明这是一个函数式接口（Java8中的概念），该接口的定义必须满足函数式接口的规范。作用在接口上。
```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
public @interface FunctionalInterface {}
```
### @Override
> 标明重写了父类的方法。作用在方法上
```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.SOURCE)
public @interface Override {
}
```
### @SafeVarargs
> 标明在可变长参数中的泛型是类型安全的，作用在构造函数和方法上。
```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.CONSTRUCTOR, ElementType.METHOD})
public @interface SafeVarargs {}
```
### @SuppressWarnings
  
> 避免编译器给出某种类型的警告。***不建议大面试使用哦！***  
> 作用在类、方法、变量、参数等上  
```java
@Target({TYPE, FIELD, METHOD, PARAMETER, CONSTRUCTOR, LOCAL_VARIABLE})
@Retention(RetentionPolicy.SOURCE)
public @interface SuppressWarnings {
    String[] value();
}
```
> 接受一个String[]类型的参数，可选值如下：  
- all : 抑制所有警告
- boxing : 抑制装箱、拆箱相关的警告
- cast : 抑制强转相关的警告
- dep-ann : 抑制过时注解相关的警告
- fallthrough : 抑制没有 break 的 switch 语句的警告
- finally : 抑制 finally 块没有 return 的警告
- hiding : 抑制关于隐藏的本地变量的警告
- incomplete-switch : 抑制 switch 语句中 case 不完整的警告（当 case 是枚举时）
- nls : 抑制创建无法翻译的字符串的警告 （nls : National Language Support）
- null : 抑制关于可能为空的警告
- rawtypes : 抑制使用泛型作为类参数时没有指明参数类型的警告
- restriction : 抑制使用不建议或者禁止的引用的警告
- serial : 抑制一个可序列化类中没有 serialVersionUID 的警告
- static-access : 抑制一个不正确的静态访问相关的警告
- synthetic-access : 抑制未优化的内部类访问相关的警告
- unchecked : 抑制未经检查的操作（比如强转）的警告
- unqualified-field-access : 抑制不合格的属性访问的警告
- unused : 抑制未使用代码相关的警告
- FieldCanBeLocal ：抑制全局变量只使用一次，可以被当做局部变量的警告
- SpellCheckingInspection ：拼写错误

## 所在包 android.annotation
- @SuppressLint :避免lint的检查
- @TargetApi : 标明target的sdk版本，从而可以使某个元素使用高版本的方法，属性等。
  > android.support.annotation包下有一个@RequireApi注解跟他很相似

## 所在包 android.support.annotation
安卓中可以使用的注解大都定义在该包下，这里面的注解又可以分为几类：
- 资源注解注解 ：用于限制参数、变量、返回值的必须使用某种类型的资源id（int值）
- 线程标识注解：标识该线程**希望**运行在哪种线程里。
- 限制取值范围的注解：可用于替代enum
- 空检查注解：限制一个元素是否可以为空
- 值约束注解：如限制传参的范围
- 其他注解，
  
### 资源注解
如下的22种资源，资源注解就是帮助编译器在编译阶段检查方法参数是否是需要的资源id
- @AnimRes
- @AnimatorRes
- @AnyRes
- @ArrayRes
- @AttrRes
- @BoolRes
- @ColorRes
- @DimenRes
- @DrawableRes
- @FractionRes
- @IdRes
- @IntegerRes
- @InterpolatorRes
- @LayoutRes
- @MenuRes
- @PluralsRes
- @RawRes
- @StringRes
- @StyleRes
- @StyleableRes
- @TransitionRes
- @XmlRes

### 线程标识注解
Android中提供了五个与线程相关的注解。  
它的判断依据是,如果调用方和被调用方的注解不一致才会错误提示.如果有一方没有线程注解,则不提示。
- @AnyThread，任何线程
- @UiThread,通常可以等同于主线程,标注方法需要在UIThread执行,比如View类就使用这个注解
- @MainThread 主线程,经常启动后创建的第一个线程
- @WorkerThread 工作者线程,一般为一些后台的线程,比如AsyncTask里面的 doInBackground就是这样的.
- @BinderThread 注解方法必须要在BinderThread线程中执行,一般使用较少.

### 限定取值范围的注解
如：
```java
    public void setVisibility(@Visibility int visibility) {
        setFlags(visibility, VISIBILITY_MASK);
    }

    @IntDef({VISIBLE, INVISIBLE, GONE})//限定Visibility的取值范围
    @Retention(RetentionPolicy.SOURCE)
    public @interface Visibility {}// 定义注解名称
    public static final int VISIBLE = 0x00000000;
    public static final int INVISIBLE = 0x00000004;
    public static final int GONE = 0x00000008;
```
涉及到的注解
- @IntDef : 限制int的取值
- @LongDef：限制长整型的取值
- @StringDef：限制String的取值

### 空检查注解
和空检查相关的注解有两个
- @Nullable 注解的元素可以是Null
- @NonNull 注解的元素不能是Null

NonNull检测生效的条件（编译器给错错误提示的条件）
- 显式传入null
- 在调用方法之前已经判断了参数为null时

### 值约束注解
- @IntRange
- @FloatRange
- @Size
- @ColorInt
- @ColorLong
- @HalfFloat
- @Dimension
- @Px

### 其他注解
- @RequiresApi
- @RequiresPermission
- @ColorInt
- @CheckResult
- @CallSuper
- @Keep
- @GuardedBy
- @RestrictTo
- @VisibleForTesting


