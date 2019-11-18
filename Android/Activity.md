# Activty

作为一个Android开发者，每天都在写Activity，但是，但是真的让我讲讲我这个每天的玩伴，好像我并不能系统的讲出来很多很多。所以，我要做一下整理，把常用的，或者我认为我需要知道的，都整理一下。
## 本篇文章将按主要讲解以下内容  
1. 简介：Activity是什么？  
2. 如何创建一个Activity？  
3. 如何启动和关闭一个Activity？   
4. 管理Activity的生命周期？  
5. 如何保存Activity状态？


### 1. 简介：Activity是什么？
> Activity 是一个应用组件，用户可与其提供的屏幕进行交互，以执行拨打电话、拍摄照片、发送电子邮件或查看地图等操作。 每个 Activity 都会获得一个用于绘制其用户界面的窗口。窗口通常会充满屏幕，但也可小于屏幕并浮动在其他窗口之上。
一个应用通常由多个彼此松散联系的 Activity 组成。 一般会指定应用中的某个 Activity 为“主”Activity，即首次启动应用时呈现给用户的那个 Activity。 而且每个 Activity 均可启动另一个 Activity，以便执行不同的操作。 每次新 Activity 启动时，前一 Activity 便会停止，但系统会在堆栈（“返回栈”）中保留该 Activity。 当新 Activity 启动时，系统会将其推送到返回栈上，并取得用户焦点。 返回栈遵循基本的“后进先出”堆栈机制，因此，当用户完成当前 Activity 并按“返回”按钮时，系统会从堆栈中将其弹出（并销毁），然后恢复前一 Activity。 （任务和返回栈文档中对返回栈有更详细的阐述。）
当一个 Activity 因某个新 Activity 启动而停止时，系统会通过该 Activity 的生命周期回调方法通知其这一状态变化。Activity 因状态变化—系统是创建 Activity、停止 Activity、恢复 Activity 还是销毁 Activity— 而收到的回调方法可能有若干种，每一种回调都会为您提供执行与该状态变化相应的特定操作的机会。 例如，停止时，您的 Activity 应释放任何大型对象，例如网络或数据库连接。 当 Activity 恢复时，您可以重新获取所需资源，并恢复执行中断的操作。 这些状态转变都是 Activity 生命周期的一部分。
本文的其余部分阐述有关如何创建和使用 Activity 的基础知识（包括对 Activity 生命周期工作方式的全面阐述），以便您正确管理各种 Activity 状态之间的转变。  

### 2. 如何创建一个Activity？
>要创建 Activity，您必须创建 Activity 的子类。您需要在子类中实现 Activity 在其生命周期的各种状态之间转变时（例如创建 Activity、停止 Activity、恢复 Activity 或销毁 Activity 时）系统调用的回调方法。 两个最重要的回调方法是：  
*onCreate()*:  
您必须实现此方法。系统会在创建您的 Activity 时调用此方法。您应该在实现内初始化 Activity 的必需组件。 最重要的是，您必须在此方法内调用 setContentView()，以定义 Activity 用户界面的布局。  
*onPause()*:  
系统将此方法作为用户离开 Activity 的第一个信号（但并不总是意味着 Activity 会被销毁）进行调用。 您通常应该在此方法内确认在当前用户会话结束后仍然有效的任何更改（因为用户可能不会返回）  
在安卓中，我们大多数情况下都使用xml布局来实现我们需要的用户界面，使用XML建造用户界面的一个好处就是我们将布局和Activity的逻辑代码分开，并以此提高代码可读性以及代码的可维护性。当然，你也可以在Activity中创建布局文件，然后将布局文件的根布局通过setcontentview设置给Activity。  
要使用我们创建的Activity 我们还必须在清单文件中申明。

### 3. 如何启动和关闭一个Activity？
>通常情况下，启动一个Activity的方式有两种，即显式启动和隐式启动。  
>Android中，四大组件的启动都是通过intent来启动的。  
>*显式启动*：在intent中显式指定Activity的名字，如startActivity(new Intent(mContext,MainActivity.class))  
>*隐式启动*： 在Intent中指定Activity的Action，指定了Action后由系统进行筛选，如果只有一个符合的就直接启动，有多个就全部展示出来，给用户选择。这种方式也正是intent强大的原因，启动一个外部Activity，比如选择照片，打电话，发短信，发邮件等等，我们不必记住每一个Activity的名字，我们只要知道他们所属的Action，然后交给系统去识别。即便有好几个邮件客户端，也一样不影响，交给用户选择他想使用的邮件客户端就可以了。如startActivity(new Intent("ActionName"))
>*关闭Activity*： 直接调用Activity.finish()方法。  

### 4. 管理Activity的生命周期？
[Activity生命周期](https://developer.android.com/images/activity_lifecycle.png?hl=zh-cn)  
先看一张图：  
![生命周期](https://raw.githubusercontent.com/coding0man/Android-/master/attachment/声明周期函数.png)
>名为“是否能事后终止？”的列表示系统是否能在不执行另一行 Activity 代码的情况下，在方法返回后随时终止承载 Activity 的进程。 有三个方法带有“是”标记：(onPause()、onStop() 和 onDestroy())。由于 onPause() 是这三个方法中的第一个，因此 Activity 创建后，onPause() 必定成为最后调用的方法，然后才能终止进程 — 如果系统在紧急情况下必须恢复内存，则可能不会调用 onStop() 和 onDestroy()。因此，您应该使用 onPause() 向存储设备写入至关重要的持久性数据（例如用户编辑）。不过，您应该对 onPause() 调用期间必须保留的信息有所选择，因为该方法中的任何阻止过程都会妨碍向下一个 Activity 的转变并拖慢用户体验。
>在是否能在事后终止？列中标记为“否”的方法可从系统调用它们的一刻起防止承载 Activity 的进程被终止。 因此，在从 onPause() 返回的时间到 onResume() 被调用的时间，系统可以终止 Activity。在 onPause() 被再次调用并返回前，将无法再次终止 Activity。
>##### 注：根据图 1 中的定义属于技术上无法“终止”的 Activity 仍可能被系统终止 — 但这种情况只有在无任何其他资源的极端情况下才会发生。进程和线程处理文档对可能会终止 Activity 的情况做了更详尽的阐述。  

### 5. 如何保存Activity状态？  
>管理 Activity 生命周期的引言部分简要提及，当 Activity 暂停或停止时，Activity 的状态会得到保留。 确实如此，因为当 Activity 暂停或停止时，Activity 对象仍保留在内存中 — 有关其成员和当前状态的所有信息仍处于活动状态。 因此，用户在 Activity 内所做的任何更改都会得到保留，这样一来，当 Activity 返回前台（当它“继续”）时，这些更改仍然存在。

>不过，当系统为了恢复内存而销毁某项 Activity 时，Activity 对象也会被销毁，因此系统在继续 Activity 时根本无法让其状态保持完好，而是必须在用户返回 Activity 时重建 Activity 对象。但用户并不知道系统销毁 Activity 后又对其进行了重建，因此他们很可能认为 Activity 状态毫无变化。 在这种情况下，您可以实现另一个回调方法对有关 Activity 状态的信息进行保存，以确保有关 Activity 状态的重要信息得到保留：onSaveInstanceState()。

>系统会先调用 onSaveInstanceState()，然后再使 Activity 变得易于销毁。系统会向该方法传递一个 Bundle，您可以在其中使用 putString() 和 putInt() 等方法以名称-值对形式保存有关 Activity 状态的信息。然后，如果系统终止您的应用进程，并且用户返回您的 Activity，则系统会重建该 Activity，并将 Bundle 同时传递给 onCreate() 和 onRestoreInstanceState()。您可以使用上述任一方法从 Bundle 提取您保存的状态并恢复该 Activity 状态。如果没有状态信息需要恢复，则传递给您的 Bundle 是空值（如果是首次创建该 Activity，就会出现这种情况）。

>##### 注：无法保证系统会在销毁您的 Activity 前调用 onSaveInstanceState()，因为存在不需要保存状态的情况（例如用户使用“返回”按钮离开您的 Activity 时，因为用户的行为是在显式关闭 Activity）。 如果系统调用 onSaveInstanceState()，它会在调用 onStop() 之前，并且可能会在调用 onPause() 之前进行调用。

>不过，即使您什么都不做，也不实现 onSaveInstanceState()，Activity 类的 onSaveInstanceState() 默认实现也会恢复部分 Activity 状态。具体地讲，默认实现会为布局中的每个 View 调用相应的 onSaveInstanceState() 方法，让每个视图都能提供有关自身的应保存信息。Android 框架中几乎每个小部件都会根据需要实现此方法，以便在重建 Activity 时自动保存和恢复对 UI 所做的任何可见更改。例如，EditText 小部件保存用户输入的任何文本，CheckBox 小部件保存复选框的选中或未选中状态。您只需为想要保存其状态的每个小部件提供一个唯一的 ID（通过 android:id 属性）。如果小部件没有 ID，则系统无法保存其状态。  

>您还可以通过将 android:saveEnabled 属性设置为 "false" 或通过调用 setSaveEnabled() 方法显式阻止布局内的视图保存其状态。您通常不应将该属性停用，但如果您想以不同方式恢复 Activity UI 的状态，就可能需要这样做。
尽管 onSaveInstanceState() 的默认实现会保存有关您的Activity UI 的有用信息，您可能仍需替换它以保存更多信息。例如，您可能需要保存在 Activity 生命周期内发生了变化的成员值（它们可能与 UI 中恢复的值有关联，但默认情况下系统不会恢复储存这些 UI 值的成员）。

>由于 onSaveInstanceState() 的默认实现有助于保存 UI 的状态，因此如果您为了保存更多状态信息而替换该方法，应始终先调用 onSaveInstanceState() 的超类实现，然后再执行任何操作。 同样，如果您替换 onRestoreInstanceState() 方法，也应调用它的超类实现，以便默认实现能够恢复视图状态。

>#####注：由于无法保证系统会调用 onSaveInstanceState()，因此您只应利用它来记录 Activity 的瞬态（UI 的状态）— 切勿使用它来存储持久性数据，而应使用 onPause() 在用户离开 Activity 后存储持久性数据（例如应保存到数据库的数据）。

>您只需旋转设备，让屏幕方向发生变化，就能有效地测试您的应用的状态恢复能力。 当屏幕方向变化时，系统会销毁并重建 Activity，以便应用可供新屏幕配置使用的备用资源。 单凭这一理由，您的 Activity 在重建时能否完全恢复其状态就显得非常重要，因为用户在使用应用时经常需要旋转屏幕。*（对于大多数应用，我们都是设置不可旋转屏幕的）*

###6.Activity的启动模式和onNewIntent方法
* Standard
* SingleTop
* SingleTask
* SingleInstance

####1. Standard 
标准模式下，每次启动Activity都会新建一个Activity实例并往Activity栈中放入。如果所有的Activity均为标准模式可能存在这样的情况。
A->B->A->C->B->A->A->A....
####2.SingleTop
在Activity栈的栈顶只会存在一个Activity实例，如果栈顶就是当前要创建Activity的实例，不创建当前实例，调用该Activity实例的onNewInstance方法；如果栈顶不是该对象的实例，则创建该对象的实例加入Activity栈的栈顶。
因此会存在栈中存在多个某Activity的实例，但是不会连续出现两个。如：
A->B->A->C->B->A->B->C->B->A->C....
####3.SingleTask
在Activity栈中只存在一个Activity的实例，如果栈中存在要创建的Activity的实例，则不会新创建Activity，并且会将该Activity上的其他Activity清除掉，同时会调用onNewIntent方法。如原来栈中元素为：A->B->C,如果A的启动模式为singleTask,则再次启动A后,B和C会被清除，栈中元素变成了A;复用上面的例子，如果要启动的是B并且B为SingleTask,再次启动B后栈中元素变成了AB.
####4.SingleInstance
会创建一个新的Activity栈，并且在新的Activity栈中加入该Activity实例，同时该栈中实例会被其他应用共享，这个没有实际用过。
####5.onNewIntent方法
在谷歌的原档中只提到了该方法会在启动模式设为SingleTop的时候调用，但是实际上似乎设为SingleTask时也同样会触发该方法。
下面是谷歌文档中的原话：
> void onNewIntent (Intent intent)
This is called for activities that set launchMode to "singleTop" in their package, or if a client used the FLAG_ACTIVITY_SINGLE_TOP flag when calling startActivity(Intent). In either case, when the activity is re-launched while at the top of the activity stack instead of a new instance of the activity being started, onNewIntent() will be called on the existing instance with the Intent that was used to re-launch it.
An activity will always be paused before receiving a new intent, so you can count on onResume() being called after this method.
Note that getIntent() still returns the original Intent. You can use setIntent(Intent) to update it to this new Intent.

谷歌爸爸是说在清单文件或者启动Activity时设置了SingleTop,在栈顶重新启动时会调用onNewIntent方法。然后呢，要注意的一点时是接收新的Intent之前Activity会先进入onPause状态，还有一点是这个时候需要手动调用setIntent方法，将参数中传递的Intent保存下来，否则下次调用getIntent方法的时候，拿到的还是第一次的那个Intent。嗯，就酱。
