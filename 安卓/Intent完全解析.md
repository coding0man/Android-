#Android Intent完全解析

在本篇博客中会涉及以下内容：

* Intent的概念和使用解析
* Intent的创建以及Action、Category、Data的解析
* 显式意图和隐式意图
* IntentFilter的概念和使用（匹配规则）
* Deep Link 和App Link

## Intent概念和使用

Intent 即为意图，可以用来启动一个Activity、发送一个广播、启动或者绑定一个服务。最典型最常见的用法就是启动一个Activity，可以把Intent看作各个Activity之间的粘合剂。Intent对将来要发生的行为进行了抽象的描述，然后在运行时绑定到特定的类，这种描述和绑定主要依赖于Intent中保存的数据。比如它保存了某个Activity的类名，那么在之后通过startActivity\(Intent\)就可以在运行时根据Intent中设置的类名找到并启动该Activity，或者它保存了Intent代表的行为，行为的类别，以及要处理的数据和数据的格式，系统也将根据这些内容以及系统内所有应用的清单文件解析出可以处理该Intent的Activity，并将Intent传递给该Activity。

Intent的主要用途在以下三个方面：
### 启动 Activity：
Activity 表示应用中的一个页面。通过将 Intent 传递给 startActivity()，您可以启动新的 Activity 实例。Intent 描述了要启动的 Activity，并携带了任何必要的数据。

如果您希望在 Activity 完成后收到结果，请调用 startActivityForResult()。在 Activity 的 onActivityResult() 回调中，您的 Activity 将结果作为单独的 Intent 对象接收。

### 启动服务：
Service 是一个不使用用户界面而在后台执行操作的组件。通过将 Intent 传递给 startService()，您可以启动服务执行一次性操作（例如，下载文件）。Intent 描述了要启动的服务，并携带了任何必要的数据。

如果服务旨在使用客户端-服务器接口，则通过将 Intent 传递给 bindService()，您可以从其他组件绑定到此服务。

### 传递广播：
广播是任何应用均可接收的消息。系统将针对系统事件（例如：系统启动或设备开始充电时）传递各种广播。通过将 Intent 传递给 sendBroadcast()、sendOrderedBroadcast() 或 sendStickyBroadcast()，您可以将广播传递给其他应用。

```java
startActivity(intent);//用于启动一个活动
startService(intent);bindService(intent)//用于开启一个服务，只能使用显示意图
sendBroadcast(intent);//发送一个广播
//startActivity,startService,sendBroadcast指明了要开启一个活动或者是启动服务等
```

## Intent的创建和参数说明

###Intent的创建
我们可以使用Intent的构造函数创建一个Intent对象，然后根据自己的需求配置合适的参数。

```java
// 这是一个显示Intent，action、category、data不能影响组件的选择，
// 所以这种情况下这些参数一般也不会去设置
// 但是可以设置一些extra参数供下个页面使用。
// 整体上，下面的代码打开一个应用内名为HaveBothActivity.class的页面
Intent intent = new Intent(MainActivity.this,HaveBothActivity.class);
intent.putExtra("userName","francis.fan");
intent.putExtra("userAvatar","http://www.null.com/user/avatar/36271.jpg")
startActivity(intent);

// 下面的这个是隐式的Intent，
// 通过设置Action，告诉系统该Intent想要做什么类型的工作（这里是打开拨号界面）
// 通过设置Data,告诉系统要操作的参数是什么（这里是某个人的手机号）
// 整体上，下面的代码打开了系统的拨号界面，并且打开时已经填入了号码
Intent intent = new Intent();
intent.setAction(Intent.ACTION_DIAL);
intent.setData(Uri.parse("tel:180****8692"));
startActivity(intent);

// 下面的这个是隐式的Intent，
// 通过设置Action，告诉系统该Intent想要做什么类型的工作（这里是要做选择某一种东西的工作）
// 通过设置mimeType,告诉系统要选择的东西的类型（这里是选择皂片）
// 整体上，下面的代码会打开手机内的某一个应用的选择图片的页面（可能有多个这样的，这时候需要用户手动选择一个）
Intent intent = new Intent();
intent.setAction(Intent.ACTION_PICK);
intent.setType("image/*");
startActivityForResult(intent,2);

// 用于在手机Web浏览器上检索数据，也设置了搜索的关键字
Intent intent = new Intent();
intent.setAction(Intent.ACTION_WEB_SEARCH);
intent.putExtra(SearchManager.QUERY,"api player 的博客");
startActivity(intent);

// 用于发送一条数据，数据的接收方会有很多，由用户选择使用哪一种
// 下面的内容设置包含了主题，标题，正文，附件，邮件接收人以及发送内容的mimeType信息在内的多种信息，但是发送的并不一定是邮件，用户也可以选择其他的应用。
Intent intent = new Intent();
intent.setAction(Intent.ACTION_SEND);
intent.putExtra(Intent.EXTRA_EMAIL,new String[]{"francis.fanfan@test.com","645534173@qq.com"});//邮件的收件人
intent.putExtra(Intent.EXTRA_CC,new String[]{"apilpayer@nullpointer.com"});//邮件的抄送人
intent.putExtra(Intent.EXTRA_TEXT,"这是发送的内容的正文部分");
intent.putExtra(Intent.EXTRA_STREAM,imageUri);//这是发送的内容的附件
intent.putExtra(Intent.EXTRA_TITLE,"这是一个标题");
intent.putExtra(Intent.EXTRA_SUBJECT,"分享的主题");
intent.setType("image/*");//指明发送的数据的类型
startActivity(intent);
```

### Intent的参数说明
intent中参数包含了Action,Category,Data(Uri+mimeType), ComponentName,Flag以及Extras等信息。

ComponentName决定有哪个应用的哪个组件来完成该项工作;  
通过setAction告知隐式意图的基本行为;    
通过addCategory为意图添加分类;    
通过setData，setType或者setDataAndType告诉系统要处理的数据和处理数据的类型;  
通过putExtra，将数据存在 Bundl e中，通过Intent传递给处理该意图的组件;

```java
//intent中的变量
private String mAction;//Action
private Uri mData;//Data
private String mType;//Type
private ComponentName mComponent;//classname
private int mFlags;//
private ArraySet<String> mCategories;//Category
private Bundle mExtras;//Bundle
```
下面针对各个参数做下说明：

#### Action解析
意图的一般性行为，通过设置此参数告诉系统我们的隐式Intent想要的操作是大致是做什么的，是查看数据，搜索或者是编辑，选择数据等。如： 

##### Activity的标准Action:
- [ACTION\_MAIN]() 一个应用的主入口
- [ACTION\_VIEW]() 用于向用户展示数据
- [ACTION\_ATTACH\_DATA]() 用于添加数据，比如把一张照片添加给联系人，或者把一段文字添加给一篇笔记上
- [ACTION\_EDIT]() 用于编辑数据,比如编辑图片
- [ACTION\_PICK]() 用于选择数据（比如常见的相册选择照片）
- [ACTION\_CHOOSER]() 提供一个用于某一个Intent的应用选择器
- [ACTION\_GET\_CONTENT]() 用于选择一项内容，不需要指定选择项的来源，还有很多其他的细节可以参考文档
- [ACTION\_DIAL]() 打开拨号页面，并填入Data中设置的电话号码，然后由用户选择是否拨打该电话
- [ACTION\_CALL]() 直接开始拨打一个Data中设定的电话号码，6.0以上需要权限，不建议使用
- [ACTION\_SEND]()用于发送/传送数据（分享功能）
- [ACTION\_SENDTO]() 用于发送一条数据对方，可以通过Data确定发送的途径，如 "sms: 180****6680" 确定使用短信发送，其他和 ACTION\_SEND 一样。
- [ACTION\_ANSWER]() 处理来电
- [ACTION\_INSERT]() Insert an empty item into the given container.
- [ACTION\_DELETE]() Delete the given data from its container.
- [ACTION\_RUN]() Run the data, whatever that means.
- [ACTION\_SYNC]() Perform a data synchronization.
- [ACTION\_PICK\_ACTIVITY]() 用于选择用于某一个Intent的Activity，返回值是一个componentName,可以根据自己的需要做点什么
- [ACTION\_SEARCH]() 用于检索数据，搜索参数放入Extras中,key值使用SearchManager.QUERY
- [ACTION\_WEB\_SEARCH]() 使用web浏览器进行搜索，搜索参数放入Extras中,key值使用SearchManager.QUERY
- [ACTION\_FACTORY\_TEST]() 工厂测试模式使用

##### BroadCast的标准Action:
- [ACTION\_TIME\_TICK]() 时间改变时发出，一分钟一次，代码设置的Receiver才能接收到。也就是说应用在活动状态才能收到
- [ACTION\_TIME\_CHANGED]() 系统时间被修改
- [ACTION\_TIMEZONE\_CHANGED]() 系统时区被修改
- [ACTION\_BOOT\_COMPLETED]() 开机完成
- [ACTION\_PACKAGE\_ADDED]() 一个新的应用安装
- [ACTION\_PACKAGE\_CHANGED]() 一个新的应用被修改
- [ACTION\_PACKAGE\_REMOVED]() 应用删除
- [ACTION\_PACKAGE\_RESTARTED]() 应用重启
- [ACTION\_PACKAGE\_DATA\_CLEARED]() 应用数据删除
- [ACTION\_PACKAGES\_SUSPENDED]() Packages have been suspended.
- [ACTION\_PACKAGES\_UNSUSPENDED]() Packages have been unsuspended.
- [ACTION\_UID\_REMOVED]() 用户被移除
- [ACTION\_BATTERY\_CHANGED]() 电量改变
- [ACTION\_POWER\_CONNECTED]() 充电器连接
- [ACTION\_POWER\_DISCONNECTED]() 充电器断开连接
- [ACTION\_SHUTDOWN]() 关机

还有两个特别的Action值得注意：

- android.intent.action.PICK_ACTIVITY 用于选择用于某一个Intent的Activity，返回值是一个componentName,可以根据自己的需要做点什么
- android.intent.action.CHOOSER 提供一个用于某一个Intent的应用选择器，可以避免直接选择默认应用。

```java

// 这是一个选择照片的隐式Intent
Intent intent = new Intent();
intent.setAction(Intent.ACTION_PICK);
intent.setType("image/*");
                
// 打开一个能和上面的Intent匹配的所有应用的页面的一个列表页
Intent chooseIntent = new Intent();
chooseIntent.setAction(Intent.ACTION_PICK_ACTIVITY);
chooseIntent.putExtra(Intent.EXTRA_INTENT, intent);
startActivityForResult(chooseIntent, 2);
// 返回值是一个ComponentName，在onActivityResult中接受并处理，可直接用于显式启动
ComponentName componentName = data.getComponent();
startActivityForResult(new Intent().setComponent(componentName), 4);

// 打开一个带标题的应用选择器，和上面的类似，区别是这次打开之后用户选择之后就直接打开应用了，
// 如果EXTRA_INTENT需要返回值，需要使用startActivityForResult，否则直接使用startActivity即可
Intent chooseIntent = new Intent();
chooseIntent.setAction(Intent.ACTION_CHOOSER);
chooseIntent.putExtra(Intent.EXTRA_INTENT,intent);
chooseIntent.putExtra(Intent.EXTRA_TITLE,"选择要打开的应用");
startActivityForResult(chooseIntent,8);
```

#### Data解析
 data分为两部分：Uri和mimeType.
  
- Uri：scheme://host:port/path（协议名称://主机地址:端口/特定服务的路径），其中path = folder/folder/file。(这几个字段存在线性依赖关系，前面的字段不存在，则后面的字段配置了也会被忽略。)  
如常见的content://表示来自系统ContentProvider中的内容；file://表示来自本机文件系统的文件；http://host:port/path 表示要处理的数据来自于基于http协议网络主机上；用户也可以指定适合自己应用格式的特定的Uri格式，如果是自定义的就不具有通用性了。  
>下面说几个Android应用中常见的几个Scheme和用法：
>> - ftp / http / https: 你懂得
> - content: ContentProvider的资源。content://djsh/whjdhs/shja.jpg
> - file: 本机文件。file://dsjj/jajs/hjah.pdf
> - tel: 电话功能。tel:180****6680
> - smsto: 消息功能。smsto:180****6680
> - emailto: 邮件功能。emailto:180****6680
> - 自定义: 网页打开应用。

- mimeType：Multipurpose Internet Mail Extensions,多目的网络邮件扩展，最初是用来表示邮件的附件的类型的，现在也用来表明附件的类型信息。  
一个MIME类型至少包括两个部分：一个类型（type）和一个子类型（subtype）。此外，它还可能包括一个或多个可选参数（optional parameter）。    
比如，HTML文件的互联网媒体类型可能是<code>text/html; charset = UTF-8</code>
在这个例子中，文件类型为text，子类型为html，而charset是一个可选参数，其值为UTF-8。  
mimeType的常见类型: [这里有对mimeType更加详尽的介绍](https://www.cnblogs.com/jsean/articles/1610265.html)
> 
> - 超文本标记语言文本 .html text/html 
> - 普通文本 .txt text/plain 
> - RTF文本 .rtf application/rtf 
> - GIF图形 .gif image/gif 
> - JPEG图形 .ipeg,.jpg image/jpeg 
> - au声音文件 .au audio/basic 
> - MIDI音乐文件 mid,.midi audio/midi,audio/x-midi 
> - RealAudio音乐文件 .ra, .ram audio/x-pn-realaudio 
> - MPEG文件 .mpg,.mpeg video/mpeg 
> - AVI文件 .avi video/x-msvideo 
> - GZIP文件 .gz application/x-gzip 
> - TAR文件 .tar application/x-tar
> 
mimeType的类型和子类型也可以用*来表示：image/\*、video/\*等
 
#### Category解析
 类别，声明所属的类别.一个包含应处理 Intent 组件类型的类别信息的字符串。 您可以将任意数量的类别描述放入一个 Intent 中，但大多数 Intent 均不需要类别。 
 下面是一些标准的Category：（很多我都不知道啊干嘛的）

- [CATEGORY\_DEFAULT]() 只有配置了此Category的Activity才可以被隐式意图启动，否则只能被显式意图启动。
- [CATEGORY\_BROWSABLE]() 目标 Activity 允许本身通过网络浏览器启动，以显示链接引用的数据，如图像或电子邮件。常用于结合自定义Scheme从浏览器启动自己的应用。
- [CATEGORY\_VOICE]() 可以进行语音交互
- [CATEGORY\_TAB]()
- [CATEGORY\_ALTERNATIVE]()
- [CATEGORY\_SELECTED\_ALTERNATIVE]()
- [CATEGORY\_LAUNCHER]() 该 Activity 是任务的初始 Activity，在系统的应用启动器中列出。该Activity的Label就是显示在桌面图标下面的那个字符串，和Application中的Label表示的所有页面的默认标题，如果其他页面不做指定，默认显示的就是Application的label属性设置的值，和Application中Theme与Activity中设置的Theme的关系一样。
- [CATEGORY\_INFO]()
- [CATEGORY\_OPENABLE]() 返回的数据是一个可以打开的Stream
- [CATEGORY\_HOME]()
- [CATEGORY\_PREFERENCE]()
- [CATEGORY\_TEST]()
- [CATEGORY\_CAR\_DOCK]() 车座
- [CATEGORY\_DESK\_DOCK]()充电器
- [CATEGORY\_LE\_DESK\_DOCK]()
- [CATEGORY\_HE\_DESK\_DOCK]()
- [CATEGORY\_CAR\_MODE]() 行车模式
- [CATEGORY\_APP\_MARKET]() 应用市场
- [CATEGORY\_VR\_HOME]()

 
#### ComponentName
 组件，包含了指定组件的包名和类名,当我们需要显式地指明处理Intent的组件时指定 ComponentName 的值。  
**所有的service的启动都应该使用显式意图**  
如果不指定组件，则由系统根据Intent中的其他信息解析出适合的组件，如果满足条件的组件有多个，则由用户指明使用哪一个组件（如果用户之前没有指定过默认打使用的组件）。


#### Extras 
Intent所携带的额外的数据，如页面跳转的时候，我们可能经常需要传递一些信息到下一个页面去，或者从当前页面返回数据给上一个页面。系统也提供了一些常用的Extra的Key值，比如：

- [EXTRA\_ALARM\_COUNT]()
- [EXTRA\_BCC]()
- [EXTRA\_CC]()  邮件的抄送人列表
- [EXTRA\_CHANGED\_COMPONENT\_NAME]()
- [EXTRA\_DATA\_REMOVED]()
- [EXTRA\_DOCK\_STATE]()
- [EXTRA\_DOCK\_STATE\_HE\_DESK]()
- [EXTRA\_DOCK\_STATE\_LE\_DESK]()
- [EXTRA\_DOCK\_STATE\_CAR]()
- [EXTRA\_DOCK\_STATE\_DESK]()
- [EXTRA\_DOCK\_STATE\_UNDOCKED]()
- [EXTRA\_DONT\_KILL\_APP]()
- [EXTRA\_EMAIL]()邮件的收件人列表
- [EXTRA\_INITIAL\_INTENTS]()
- [EXTRA\_INTENT]() 主要用在ACTION_PICK_ACTIVITY和和ACTION_CHOOSER上，说明要操作的Intent数据
- [EXTRA\_KEY\_EVENT]()
- [EXTRA\_ORIGINATING\_URI]()
- [EXTRA\_PHONE\_NUMBER]()
- [EXTRA\_REFERRER]()
- [EXTRA\_REMOTE\_INTENT\_TOKEN]()
- [EXTRA\_REPLACING]()
- [EXTRA\_SHORTCUT\_ICON]() 快捷键icon
- [EXTRA\_SHORTCUT\_ICON\_RESOURCE]()
- [EXTRA\_SHORTCUT\_INTENT]()
- [EXTRA\_STREAM]() 附件的Uri地址，通过Uri可以获得附件的数据流
- [EXTRA\_SHORTCUT\_NAME]()
- [EXTRA\_SUBJECT]() 消息的主题
- [EXTRA\_TEMPLATE]()
- [EXTRA\_TEXT]() 消息的正文
- [EXTRA\_TITLE]() 消息的标题
- [EXTRA\_UID]()

#### Flag 
在 Intent 类中定义的、充当 Intent 元数据的标志。 标志可以指示 Android 系统如何启动 Activity（例如，Activity 应属于哪个任务栈），以及启动之后如何处理。
关于常用Flag的解析，看看[任玉刚的这篇博客](https://blog.csdn.net/singwhatiwanna/article/details/9294285)。
注意：如果是从BroadcastReceiver启动一个新的Activity，或者是从Service往一个Activity跳转时，不要忘记添加Intent的Flag为FLAG\_ACTIVITY\_NEW\_TASK。


##显式意图和隐式意图

显式和隐式意图可以根据 Intent 中是否指定了要启动的组件（mComponent 是否指定），指定了特定组件的就是显式意图否则就是隐式意图。

### 显式意图

按名称（完全限定类名）指定要启动的组件。 通常，您会在自己的应用中使用显式 Intent 来启动组件，这是因为您知道要启动的 Activity 或服务的类名。例如，启动新 Activity 以响应用户操作，或者启动服务以在后台下载文件。
如果 Intent 中 mComponent 不为空，即开发者使用setComponent或者setClass或者setClassName等显式指定处理意图的组件，此时 Intent 中设置的 action、category、data 等不影响组件的选择。适用条件是开发者准确知道可以处理该意图的对象的确切的位置（包名和类名），如果调用对象在应用内可以直接指定 xxxx.class ，如果调用其他应用则需要指定包名以及类名的确切路径。
```java
//显式意图(通过设置 ComponentName 参数指定完成意图的组件)
//设置Component参数一共有三种方式
intent.setClass(this,MainActivity.class);
intent.setClassName(this,"com.nullpointer.stud.MainActivity.class");
intent.setClassName("com.nullpointer.study","com.nullpointer.study.MainActivity");
//其实质都是通过包名和类名构造了一个ComponentName
//打开当前APP内组件可以不指定包名而通过context指定，但是实质还是使用包名
//打开其他应用的组件需要指定完整包名和全限定类名
```

### 隐式意图

不会指定特定的组件，而是声明要执行的常规操作，从而允许其他应用中的组件来处理它。 例如，如需在地图上向用户显示位置，则可以使用隐式 Intent，请求另一具有此功能的应用在地图上显示指定的位置。
创建隐式 Intent 时，Android 系统通过将 Intent 的内容与在设备上其他应用的清单文件中声明的 IntentFilter 进行比较，从而找到要启动的相应组件。 如果 Intent 与 IntentFilter 匹配，则系统将启动该组件，并向其传递 Intent 对象。 如果多个 Intent 过滤器兼容，则系统会显示一个对话框，支持用户选取要使用的应用。
隐式意图则是通过设置 Action，Category，Data等之后交给系统（ Intent 中的 action 一般都需要指定，Category 可以不设置，如果不设置则默认为只有一个 CATEGORY_DEFAULT ），由系统以及用户选择处理该意图的组件，如果 Activity 希望被隐式意图启动，则必须包含 android.intent.category.DEFAULT。

```java
//通过setAction指定操作的类型，通过setData指定操作的数据，
//通过设置Category限定类别
//通过设置setType指定操作的对象的类型

//Intent.ACTION_DIAL tel:180****6680用于打开拨号界面
//下面的示例就指定了一个打开拨号界面的意图，并且指定了拨号界面的数据是朕的手机号码
intent.setAction(Intent.ACTION_DIAL);
intent.setData(Uri.parse("tel:180****6680"));

//下面的示例就打开了指定的网页
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
Android 提供了很多标准Intent的使用范例，也预置了很多标准Action,Category，Extra的字段等，下面列举一下常用的。


Android 提供了很多标准Intent的使用范例，也预置了很多标准Action

[谷歌在线文档](https://developer.android.com/reference/android/content/Intent.html)

[一个不错的中文示例](http://blog.csdn.net/zhangjg_blog/article/details/10901293)

有些东西没必要去死记硬背，需要的时候去翻文档就好了

## IntentFilter 及其 匹配规则

Intent 过滤器是应用清单文件中的一个表达式，它指定该组件要接收的 Intent 类型。 例如，通过为 Activity 声明 Intent 过滤器，您可以使其他应用能够直接使用某一特定类型的 Intent 启动 Activity。同样，如果您没有为 Activity 声明任何 Intent 过滤器，则 Activity 只能通过显式 Intent 启动。

就像前面讲到的，我们可以通过设置 action、category、data参数来让系统帮我们选择进行此操作的组件，或者应用。 Action 指定了将要操作的行为，是打开一个网页还是去打开一个拨号页面，或者是发短信、发邮件等等等等。 Data 就指定了要操作的数据。

我们使用了系统提供的 Action 可以方便地调用系统的一些服务。如果我们的应用需要被别的应用调用呢？这个时候我们可以设置自己应用 Activity 或者 Service 或者 BrocastReceiver 的 interFilter 来告诉系统我们的组件可以处理哪些数据。相信下面的代码大家一定不会陌生：

```xml
<!--这是应用的启动页，设置action=main和category=launcher表示这是应用的主屏，打开应用时默认打开的页面-->
<activity android:name=".MainActivity"
	android:label="launcher上显示的名字"
	android:launchMode="singleTop">
	<intent-filter>
		<action android:name="android.intent.action.MAIN" />
		<category android:name="android.intent.category.LAUNCHER" />
	</intent-filter>
</activity>
```
下面介绍一下Action，Data,Category的含义和作用，然后给出几个常用的值。

###Intent的匹配规则
根据Intent中的配置找到满足条件的IntentFilter，进而找到对应的组件。此处的Intent全部指的是隐式Intent,因为不管显式Intent中action,category,data配置的是什么，Intent都将会传递给显式指定的组件。  

Intent的整体匹配规则是：

- Intent和组件匹配通过的条件：每个组件都可以包含若干个IntentFilter，只要有一个IntentFilter匹配通过，则该组件与该Intent匹配成功。
- IntentFilter与Intent匹配通过的条件：action，category，data全部匹配通过

提前说一下数学中子集的概念：如果集合A的任意一个元素都是集合B的元素，那么集合A称为集合B的子集。空集合是所有集合的子集。

#### action的匹配规则：（Intent中的Action是IntentFilter中Action的子集。）
一个IntentFilter可以包含0-n个Action，一个Intent只能包含一个Action。

- 如果Action字段没有指定，则data里的mimeType字段必须指定。这个是在源码里找到的答案，我之前在测试的时候，想尝试看看如果不指定Action会怎样，后来去源码里找到了答案。
- Intent中设置的action被包含在IntentFilte所配置的action中，则匹配成功。
- 另外，如果过滤器没有配置Action，则所有的隐式Intent均无法匹配成功；相反地，如果Intent没有设置Action，则所有配置了Action的IntentFilter均可以与之匹配。

#### Category的匹配规则：（Intent中的Category是IntentFilter中Category的子集）
一个Intent可以设置多个Category，一个IntentFilter也可以包含0-n个Category。  
另外，所有的隐式Intent都包含一个CATEGORY\_DEFAULT的Category。

- Intent中设置的Category全部被包含在IntentFilter中配置的Category中则匹配成功。
- 另外，Android系统会将CATEGORY_DEFAULT类别传递给startActivity（intent）和startActivityForResult（intent）中的intent，所以只有至少配置了CATEGORY\_DEFAULT类别的IntentFilter可以与之匹配；相反地，如果过滤器没有配置Category，则所有的隐式Intent均无法与之匹配成功。

#### Data的匹配规则：
一个 IntentFilter 可以配置0-n个data,一个Intent只能包含一个data.  
前面已经说过了data分为两部分： Uri 和 mimeType ,因此 data 的匹配也要分成两部分进行匹配。 

##### Uri的匹配规则：
根据前面所讲到的知道Uri也分为四个部分：scheme、host、port、path。则Uri的匹配规则如下

- 如果 IntentFilter 中值配置了 scheme ,则 Intent 中的 scheme 与 IntentFilter 中的 scheme 一致即可， 如果 Intent 中 scheme 之后配置有后面的字段，不影响匹配结果。
- 如果 IntentFilter 中值配置了 scheme 和 host ,则Intent中的 scheme 和 host 必须分别与 IntentFilter 中的 scheme 和 host 一致才能匹配成功。如果 Intent 中 host 之后配置有后面的字段，不影响匹配结果。
- 如果 IntentFilter 中值配置了 scheme 、 host 、port ,则Intent中的 scheme 、 host 、port
 必须分别与 IntentFilter 中的 scheme 、 host 、port一致才能匹配。如果 Intent 中 port 之后配置有后面的字段，不影响匹配结果。
- 如果 IntentFilter 中值配置了 scheme 、 host 、port 、path ,则Intent中的 scheme 、 host 、port、path
 必须分别与 IntentFilter 中的 scheme 、 host 、port、path 一致才能匹配。**IntentFilter 中path中可以包含通配符，只要 IntentFilter 中 path 与 Intent 中的 path Match即可通过匹配**。
- Uri中path的匹配path、pathPrefi、pathPattern都可以用来进行path的配置，下面说一下他们之间的区别

> - path 用来匹配完整的路径，如："https://developer.android.com/reference/android/content/Intent.html" ，这里将 path 设置为 "/reference/android/content/Intent.html" 才能够进行匹配；
- pathPrefix 用来匹配路径的开头部分，拿上面的 Uri 来说，这里将 pathPrefix 设置为 "/reference" 或者 "/android/content" 就能进行匹配了；
- pathPattern 用表达式来匹配整个路径，本例中可以用 ".\*\\.html" 匹配所有以.html 结尾的路径(注意字符的反义)。这里需要说下两种匹配符号。
  
>>匹配符号：
  
>> - "\*" 用来匹配0次或更多，如："a*" 可以匹配""、"a"、"aa". 
>> - "." 用来匹配任意单个字符，如："." 可以匹配"a"、"b"，"c".
  
> 因此 ".\*" 就是用来匹配任意字符0次或更多，如：".\*\.html" 可以匹配 "abc/def/ghi/jkl.html"、"ccbh.tml"；"/reference/.\*.html"表示以 "/reference" 开始 以".html"结尾的路径。(".html" 表示 html 前面有任意一个字符，"\.html" 表示以 .html 结尾)

##### mimeType的匹配规则:
- 父类型是\*则子类型只能是\*,可以通配所有的mimeType。如\*/\*可以匹配abc/def也可以匹配acb/fed。
- 父类型为常规值，子类型是\*,可以统配父类型下的所有子类型。如image/*既可以匹配image/jpg也可以匹配image/png。
- 父类型和子类型均为常规值，则需要两个都同时匹配才可以匹配成功。如text/html只能匹配text/html

#### Uri和MimeType结合时的匹配规则(从Intent的角度出发)
1. 如果Intent没有配置Uri和mimeType,则只能匹配到也未配置Uri和mimeType的IntentFilter。
2. 如果Intent包含Uri但是不包含mimeType（没有显式指定，也无法根据Uri推断出来），则只能匹配到Uri匹配且也未配置mimeType的IntentFilter。换言之，如果IntentFilter配置了mimeType，则无法匹配成功。
3. 如果Intent包含mimeType但是不包含Uri,则只能匹配到mimeType匹配成功，且没有配置Uri的IntentFilter.换言之，如果IntentFilter配置了Uri，则无法匹配成功。
4. 如果Intent同时包含mimeType和Uri，匹配成功的IntentFilter必须满足的首要条件是配置了的mimeType与Intent中设置的相互匹配。其次匹配成功的第二个条件是：

>- Intent的Uri为content或者file类型:IntentFilter可以没有配置Uri（**如果IntentFilter配置了mimeType，没有配置Uri，系统假定组件支持content:或者file:类型的Uri**）,或者也配置了与Intent中Uri匹配的Uri.
>- Intent的Uri不为content或者file类型：IntentFilter也配置了与Intent中Uri匹配的Uri.


1、2和4可能还算用的多的，3并不会怎么用。3是不包含Uri但是指定了mimeType，资源都没有，资源的类型也就毫无意义了，所以不是很常用。

#### Uri和MimeType结合时的匹配规则(从IntentFilter的角度出发)
1. 如果IntentFilter没有配置Uri和mimeType，则只能匹配到没有配置Uri和mimeType的Intent。
2. 如果IntentFilter配置了Uri没有配置mimeType，则只能匹配到同样Uri且未配置mimeType的Intent。
3. 如果IntentFilter配置了mimeType没有配置Uri，则系统假定组件支持content:或者file:类型的Uri，所以只能匹配到配置了相同mimeType（没有显示指定但是可以推断出的也可以）且Uri类型为content:或者file:的Intent。
4. 如果IntentFilter同时配置了Uri和mimeType，则可以匹配到配置了匹配的Uri和匹配的mimeType的Intent。

>  其实上面的规则很好理解：IntentFilter是用来处理数据的，所以他要处理的数据的范围必须要大于Intent中设置的数据的范围（范围同时包含Uri类型和mimeType类型）。如果遵守的协议都不一样，肯定没办法处理啊，如果能处理的mimeType也无法匹配，那就自然也处理不了了。   
> 有一个规律就是，Intent中未配置的项默认范围最大，IntentFilter未配置的项默认范围最小。比如：
>  
>- 如果Intent中不包含Uri则默认Intent配置的Uri是最大的范围，所以配置了Uri的IntentFilter的Uri范围自然变小，不是包含关系，所以没法处理；mimeType也是同样的。
>- 如果IntentFilter中不包含Uri,系统给了content和file的默认值，不包含mimeType时，所有配置了mimeType的Intent均无法与之匹配成功。
>


根据上面的介绍的匹配规则看一下下面的示例：

```xml
<!--假设现在有一个Activity在清单文件中的设置如下：-->
    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/AppTheme">
        
        <!--该Activity将会出现在Launcher上，并且Launcher上的标题是“启动页面”-->
        <activity android:name=".MainActivity"
        		android:label="启动页面">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>

		<!--因为没有任何设置Action和data,所以无法通过隐式Intent启动-->
        <activity android:name=".NoActionActivity"
            android:label="NoActionActivity">
            <intent-filter>
                <category android:name="com.nullpointer.intentdemo.DEMO_CATEGORY" /
                <category android:name="android.intent.category.DEFAULT"/>
            </intent-filter>
        </activity>
        
        <!--因为没有设置任何Category，所以无法通过隐式Intent启动-->
        <activity android:name=".NoCategoryActivity"
            android:label="NoCategoryActivity">
            <intent-filter>
                <action android:name="com.nullpointer.intentdemo.DEMO_ACTION" />
            </intent-filter>
        </activity>
        
        <!--因为没有设置默认Category，所以无法通过隐式Intent启动-->
        <activity android:name=".NoDefaultCategoryActivity"
            android:label="NoDefaultCategoryActivity">
            <intent-filter>
                <action android:name="com.nullpointer.intentdemo.DEMO_ACTION" />
                <category android:name="com.nullpointer.intentdemo.DEMO_CATEGORY" />
            </intent-filter>
        </activity>
        
        <!--匹配成功的隐式Intent必须满足满足以下条件-->
        <!--1. Action属性只能是 com.nullpointer.intentdemo.DEMO_ACTION -->
        <!--2. Category属性可以不配置，否则只能添加一个com.nullpointer.intentdemo.DEMO_CATEGORY-->
        <!--3. Uri和mimeType均没有进行配置-->
        <activity android:name=".NoDataActivity"
            android:label="NoDataActivity">
            <intent-filter>
                <action android:name="com.nullpointer.intentdemo.DEMO_ACTION" />
                <category android:name="com.nullpointer.intentdemo.DEMO_CATEGORY" />
                <category android:name="android.intent.category.DEFAULT"/>
            </intent-filter>
        </activity>
        
        <!--匹配成功的隐式Intent必须满足满足以下条件-->
        <!--1. Action属性只能是 com.nullpointer.intentdemo.DEMO_ACTION 或者 android:name="android.intent.action.VIEW"-->
        <!--2. Category属性可以不配置，否则只能添加一个com.nullpointer.intentdemo.DEMO_CATEGORY-->
        <!--3. Uri进行了配置，并且协议用的"demo"，剩下的字段（host、port、path）是否配置，配置成什么都无所谓,mimeType未进行配置-->
        <activity android:name=".NoMimeTypeActivity"
            android:label="NoMimeTypeActivity">
            <intent-filter>
                <action android:name="android.intent.action.VIEW"/>
                <action android:name="com.nullpointer.intentdemo.DEMO_ACTION" />
                <category android:name="com.nullpointer.intentdemo.DEMO_CATEGORY" />
                <category android:name="android.intent.category.DEFAULT"/>
                <data android:scheme="demo" />
            </intent-filter>
        </activity>
        
        <!--匹配成功的隐式Intent必须满足满足以下条件-->
        <!--1. Action属性如果不配置，则mimeType必须配置为“demo/mime”，或者只能是  com.nullpointer.intentdemo.DEMO_ACTION -->
        <!--2. Category属性可以不配置，或者添加com.nullpointer.intentdemo.DEMO_CATEGORY1/2/3中的一个两个或者三个全部添加-->
        <!--3. mimeType进行配置为“demo/mime”或者“*/*”,或者未进行配置-->
        <activity android:name=".NoUriActivity"
            android:label="NoUriActivity">
            <intent-filter>
                <action android:name="com.nullpointer.intentdemo.DEMO_ACTION" />
                <category android:name="com.nullpointer.intentdemo.DEMO_CATEGORY1" />
                <category android:name="com.nullpointer.intentdemo.DEMO_CATEGORY2" />
                <category android:name="com.nullpointer.intentdemo.DEMO_CATEGORY3" />
                <category android:name="android.intent.category.DEFAULT"/>
                <data android:mimeType="demo/mime"/>
            </intent-filter>
        </activity>
        
        <!--匹配成功的隐式Intent必须满足满足以下条件-->
        <!--1. Action属性如果不配置，则mimeType必须配置为“demo/mime”，或者只能是  android.intent.action.VIEW -->
        <!--2. Category属性可以不配置，或者添加com.nullpointer.intentdemo.DEMO_CATEGORY-->
        <!--3. mimeType进行配置为“demo/mime”或者“*/*”,Uri的scheme定义为“demo”，-->
        <activity android:name=".HaveBothActivity"
            android:label="HaveBothActivity">
            <intent-filter>
                <action android:name="android.intent.action.VIEW"/>
                <category android:name="com.nullpointer.intentdemo.DEMO_CATEGORY" />
                <category android:name="android.intent.category.DEFAULT"/>
                <data android:scheme="demo" android:host="test" android:port="1234" android:path="path" android:mimeType="demo/mime"/>
            </intent-filter>
        </activity>
    </application>
```


## Deep Link 和App Link

这个问题可以说是很大的问题了。简要说一下，详细的就太多内容了。
参考：
[欧阳辰的专栏-移动DeepLink的前世今生](https://zhuanlan.zhihu.com/p/20694818)
[google training-Deep Linking and Android App Links](https://developer.android.com/training/app-links/index.html)


### 网页打开应用
常见的应用大都会为他们的应用提供App和网页两种访问途径，但是显而易见的是APP可以提供更好的用户体验，因此从网页进入我们的App也就成为了一种需求。目前从网页进入App的途径就是通过设置Activity的IntentFilter。  
如下面的这个Activity就可以允许网页打开自己。  

```xml
<activity 
	android:name=".UserInfoActivity"
	android:label="用户信息">
        <intent-filter>
            <action android:name="android.intent.action.VIEW"/>
            <category android:name="android.intent.category.BROWSABLE"/>
            <category android:name="android.intent.category.DEFAULT"/>
            <data android:scheme="nullpointer" host="com.nullpointer.demo" path="/userInfo"/>
         </intent-filter>
</activity>
```
上面这个页面是一个用于展示用户信息的页面，我们配置了一个用于网页打开的过滤器（当然也可以再配置一些用于其他用途的过滤器），我们在这个过滤器中配置了 action,category 以赋予我们的应用可以被网页打开的能力，设置data的值来确保我们的当前只响应我们感兴趣的Uri.

ACTION_VIEW 用于展示数据，总是必须的。  
CATEGORY_BROWSABLE 允许我们的应用可以被浏览器打开，因此也是必须的。
CATEGORY_DEFAULT 允许我们的应用允许被隐式意图打开，因而也是必须的。
通过设置Data中scheme、host、path或者path的变种，允许特定的链接才能打开我们的当前页面。

将下面的网页在浏览器中打开，然后点击页面上的“打开我的应用”链接，即可打开我们的App:

```html
<!DOCTYPE html>  
<html>  
	<body>
		<h1>Test Scheme</h1> 
		<!—自动加载隐藏页面跳转—>
		<!-- <iframe src="myscheme://www.orangecpp.com:80/mypath?key=mykey" style="display:none"></iframe> -->
		<!—手动点击跳转—>
		<a href="nullpointer://com.nullpointer.demo/userInfo?userId=001">打开我的应用</a>
	</body>  
</html>
```

然后我们在App中这样获取应用的参数：  
在这里，我们是直达需要的页面，有时候也可以配置一个统一的入口，然后根据参数来确定打开哪个页面。

```java
//因为可以打开我们的这个页面，所以前面的host，path都不要再提了
Intent intent = getIntent();
if (TextUtils.equals("nullpointer",intent.getScheme())){
    Uri uri = intent.getData();
    String userId = uri.getQueryParameter("userId");
    //获取了userId，就爱干啥干啥吧
    Toast.makeText(this,"获取的用户Id是"+userId,Toast.LENGTH_SHORT).show();
}
```
### 建立深度链接

在上面的实例中，我们只是确保了我们的app中的页面可以被浏览器打开，然后通过携带的参数进行相应的展示，这并不符合谷歌对深度链接的定义。  
深度链接的一个显著特征是：在用户通过浏览器得到的搜索结果中，当点击某一条来自与当前App配套的网页的结果的时候，浏览器可以直接打开我们的应用（甚至不需要用户选择要打开的App），而不是先进入网页，然后再打开我们的App。



```xml
<!--通过站点地图添加深度链接-->
<?xml version="1.0" encoding="UTF-8" ?><urlset    xmlns="http://www.sitemaps.org/schemas/sitemap/0.9"    xmlns:xhtml="http://www.w3.org/1999/xhtml">    <url>        <loc>example://gizmos</loc>            <xhtml:link                rel="alternate"                href="android-app://com.example.android/example/gizmos" />    </url>    ...</urlset>

<!--在特定的页面上添加深度链接-->
<html><head>    <link rel="alternate"          href="android-app://com.example.android/example/gizmos" />    ...</head><body> ... </body>

```

好了，关于Intent的基本知识就先到这里为止了。

