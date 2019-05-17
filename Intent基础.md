#
Android Intent基础

在本篇博客中会涉及以下内容：

* Intent的概念和基本知识以及使用
* Action解析
* Category解析
* Data和Extras解析
* 建立深度搜索（在浏览器搜索结果中打开APP特定页面）

## Intent概念

Intent 即为意图，意图就是你要干啥。可以通过Intent告诉系统你要干什么，找谁干，怎么干（参数）。Intent可以用来启动一个Activity，可以用来发送一个广播，启动或者绑定一个服务。典型的用法就是启动一个Activity，可以把Intent看作各个Activity之间的粘合剂。Intent对将来要发生了行为进行了抽象的描述，然后在运行时进行绑定，这种描述和绑定主要依赖于Intent中保存的数据。比如它保存了某个Activity的类名，那么在之后通过startActivity\(Intent\)就可以在运行时根据Intent中设置的类名找到并启动该Activity。 

### Intent包含的数据

* **action** 意图的行为，如android.intent.action.VIEW用于查看数据，android.intent.action.SEARCH用于检索数据，android.intent.action.PICK用于选择数据（比如常见的相册选择照片）。
* **data 和 type** ,意图将要处理的数据以及数据的格式，一般情况下数据的格式可由数据进行推断，当然也可以手动进行指定。仅仅根据action可以不够确定一个准确的范围，如android.intent.action.EDIT用于编辑，但是可以编辑文本的Activity未必可以编辑图像，因此data和type进一步确定了范围。
* **category** 类别，声明所属的类别
* **component** 组件，当我们需要显式地指明处理Intent的组件时使用，**所有的service的启动都应该使用显式意图**，如果不指定组件，则由系统进行解析以及由用户指明处理Intent的组件。
* **extras** intent所携带的额外的数据，如要发送一个邮件，可能还需要提前指定收件人主题内容等信息。

###显式意图和隐式意图

* 显式意图

> 如果Intent中component不为空，即开发者使用setComponent或者setClass或者setClassName等显式指定处理意图的组件，此时Intent中设置的action，data等不影响。适用条件是开发者准确知道可以处理该意图的对象的确切的位置，如果调用对象在应用内可以直接指定xxxx.class，如果调用其他应用则需要指定包名以及类名的确切路径。

* 隐式意图

> 隐式意图则是通过设置Action，Category，Data，Type等之后交给系统，由系统以及用户选择处理该意图的组件。

一些代码示例

```java
//启动一个Activity、Service或是一个发送一个广播都离不开Intent
startActivity(intent);
startService(intent);
sendBroadcast(intent);
//startActivity,startService,sendBroadcast指明了要干什么
//找谁干和怎么干就需要设置Intent中的参数了

//intent中的参数
private String mAction;//Action
private Uri mData;//Data
private String mType;//Type
private String mPackage;
private ComponentName mComponent;//classname
private int mFlags;//启动模式
private ArraySet<String> mCategories;//Category
private Bundle mExtras;//Bundle
private Rect mSourceBounds;
private Intent mSelector;
private ClipData mClipData;
private int mContentUserHint = UserHandle.USER_CURRENT;
private String mLaunchToken;
/**
* intent中参数的设置
* setClass方式是由开发者来确定由谁来完成该项工作
* 通过setAction，addCategory，setData，setType告诉系统工作的类型以及要处理的数据和处理数据的类型，
* 由系统决定由谁来完成（可能需要用户的参与）
* putExtra，将数据存在bundle中，通过Intent传递的额外的数据
**/

//显式意图(通过设置ComponentName参数指定完成意图的组件)
//设置Component参数一共有三种方式
intent.setClass(this,MainActivity.class);
intent.setClassName(this,"com.nullpointer.stud.MainActivity.class");
intent.setClassName("com.nullpointer.study","com.nullpointer.study.MainActivity");
//其实质都是通过包名和类名构造了一个ComponentName
//打开当前APP内组件可以不指定包名而通过context指定，但是实质还是使用包名
//打开其他应用的组件需要指定完整包名和全限定类名


//隐式意图
//通过setAction指定操作的类型，通过setData指定操作的数据，
//通过设置Category限定类别
//通过设置setType指定操作的对象的类型

//?下面的示例就指定了一个打开拨号界面的意图，并且指定了拨号界面的数据是朕的手机号码
intent.setAction(Intent.ACTION_DIAL);
intent.setDataAndType(Uri.parse("tel:18013116680"),"uri");

//?下面的示例就打开了指定的网页
intent.setAction(Intent.ACTION_VIEW);
intent.setData(Uri.parse("http://www.google.com"));

//?下面的示例就打开了搜索结果页面
intent.setAction(Intent.ACTION_WEB_SEARCH);
intent.putExtra(SearchManager.QUERY,"Android");

//需要注意的是 setData和setType互斥，如果需要同时设置 请使用setDataAndType();

public @NonNull Intent setData(@Nullable Uri data)
mData = data;
mType = null;
return this;
}
public @NonNull Intent setType(@Nullable String type) {
mData = null;
mType = type;
return this;
}
```

Android 提供了很多标准Intent的使用范例，也预置了很多标准Action

[谷歌在线文档](https://developer.android.com/reference/android/content/Intent.html)

[一个不错的中文示例](http://blog.csdn.net/zhangjg_blog/article/details/10901293)

有些东西没必要去死记硬背，需要的时候去翻文档就好了

## IntentFilter

就像前面讲到的，我们可以通过设置Action，设置data来让系统帮我们选择进行此操作的组件，或者应用。Action指定了将要操作的行为，是打开一个网页还是去打开一个拨号页面，或者是发短信、发邮件等等等等。Data就指定了要操作的数据。

我们使用了系统提供的Action可以方便地调用系统的一些服务。如果我们的应用需要被别的应用调用呢？这个时候我们可以设置自己应用Activity或者Service或者BrocastReceiver的interFilter来告诉系统我们的组件可以处理哪些数据。相信下面的代码大家一定不会陌生：

```xml
//这是应用的启动页，设置action=main和category=launcher表示这是应用的主屏，打开应用时默认打开的页面
<activity android:name=".MainActivity"
android:launchMode="singleTop">
<intent-filter>
<action android:name="android.intent.action.MAIN" />
<category android:name="android.intent.category.LAUNCHER" />
</intent-filter>
</activity>
```

#### Action:

一个想要执行的动作的名称，通常是系统已经定义好的值或者自定义的值。使用action进行匹配需要Intent中的Action可以和xml中定义的Action中的一个相同。

#### Data/Type:

Intent附带的数据以及数据的类型。分为两部分：URI和mimeType.
URI的格式：scheme://host:port/path\|pathPattern\|pathPrefix
mimeType的常见类型：image/\*、video/\*、string、text/plain等

#### Category：用的比较少，我也不是很懂。

系统也预定义了一些Category，或者开发者可以自定义。使用category进行过滤的规则是，如果Intent中包含Category,则每一个Category都被包含在xml中Intent-Filter中的设置的category中时才能匹配成功。

根据上面的介绍的匹配规则看一下下面的示例：

```xml
<!--假设现在有一个Activity在清单文件中的设置如下：-->
<activity android:name=".TestActivity"
android:launchMode="singleTop">
<intent-filter>
<action android:name="android.intent.action.SEND"/>
<action android:name="android.intent.action.SENDTO"/>
<action android:name="android.intent.action.SEND_MULTIPLE" />

<category android:name="android.intent.category.APP_MAPS" />
<category android:name="android.intent.category.APP_BROWSER" />
<category android:name="android.intent.category.VOICE"/>

<data
android:scheme="http"
android:host="www.shaishufang.com"
android:mimeType="text/plain"
/>
</intent-filter>
</activity>
```

如果根据Action 进行匹配：下面两个Intent都可以与该Activity匹配成功

```java
Intent intent1 = new Intent();
intent1.setAction("Intent.ACTION_SEND");

Intent intent2 = new Intent();
intent2.setAction("Intent.ACTION_SENDTO");
```

如果根据Data进行匹配：intent1匹配不成功，intent2则匹配成功

```java
Intent intent1 = new Intent();
intent1.setDataAndType(Uri.parse("http://www.google.com"),"text/plain");

Intent intent2 = new Intent();
intent2.setData(Uri.parse("http://www.shaishufang.com/bookdetail?bid=1234"),"text/plain");
```

如果根据Category进行匹配：

1. intent1可以匹配成功，因为添加的两个category在Activity的intent-filter中都有设置
2. intent2匹配不成功，因为虽然Maps有设置，但是Email没有设置，根据规则必须两个都在intent-filter中设置才可以匹配成功。

```java
Intent intent1 = new Intent();
intent1.addCategory(Intent.CATEGORY_APP_MAPS);
intent1.addCategory(Intent.CATEGORY_APP_BROWSER);

Intent intent2 = new Intent();
intent2.addCategory(Intent.CATEGORY_APP_MAPS);
intent2.addCategory(Intent.CATEGORY_APP_EMAIL);
```

如果将三种匹配规则进行混合匹配，则必须全部满足才能匹配成功。

### 实战演练

下面通过两个应用中的实例来直观的感受一下如何在我们的APP中设置intent-filter达到特定的目的。
现在假设用户可以通过我们的应用给好友发送数据,但是只能发文字，我们可以这么设置。（如果也可以发图片，把注释放开就可以了）：

```xml
<!--
在这里，我们通过Action设置了我们的应用可以执行发送数据的操作，
并且通过data属性指定了我们的这个页面可以操作的数据对象是text/plain格式的数据
这样，当其他的APP通过隐式意图的时候，系统匹配到我们的这个页面有这个功能，
并且将我们的应用和其他的同样具有此功能的APP展示给用户，由用户选择一个APP进行发送
-->
<activity android:name="ShareActivity">
<intent-filter>
<action android:name="android.intent.action.SEND"/>
<category android:name="android.intent.category.DEFAULT"/>
<data android:mimeType="text/plain"/>
<!--<data android:mimeType="image/*"/>-->
</intent-filter>
</activity>
```

现在又有一个常见需求，在浏览器打开网页，自动跳转了app,也是通过设置IntentFilter实现的。

```xml
<!--
在网页中打开一个Uri格式为shaishufang://shaishufang.com?type=book&bid=1234的链接时，
如果您的手机安装的有晒书房，浏览器会自动跳转到应用内打开特定页面（根据后面的参数会知道用户要访问的内容）
又有如果您允许允许谷歌搜索您的APP，建立深度链接，把注释放开就可以了
您可以试试使用谷歌搜索“丝袜 知乎”，随便点击搜索结果前几个中的一个，
会直接打开知乎的该问题页面，百度搜索就不行。
我本来也想做来着，可是我们的产品是必须登录才有权限访问，不符合谷歌的建议，就没做
如果您有兴趣，可以跟产品建议一下，为用户提供更好的使用体验吧
-->
<intent-filter>
<!-- 网页打开 -->
<action android:name="android.intent.action.VIEW" />
<category android:name="android.intent.category.BROWSABLE" />
<category android:name="android.intent.category.DEFAULT" />
<data
android:host="shaishufang.com"
android:scheme="shaishufang" />
<!--<data
android:host="shaishufang.com"
android:scheme="http" />-->
</intent-filter>
```

好了，关于Intent的基本知识就先到这里为止了。

