#修复bug  

##3-13日审查的bug  

1. [x] fragment 状态异常   
	
	> 错误原因:  
	>activity 已经不存在，还在commit，造成状态异常  
	>commit();  
	>commitallowstateslos();  
	 
	错误发生处
	>- OthersHomeActivity2 line438   
	>- MainActivity  line156  
	>- BaseActivity onbackpressed
	 
 	修复过程 commit改为 commitallowstateslos()
 	
2. [x] Camera 异常   
	> 错误原因:  
	>> 
		Camera  is being used after Camera.release() was called 
		是因为在之前调用_mCamera.startPreview()方法之前，调用了  
		_mCamera.setPreviewCallback(xxActivity.this)，导致在手动调用上面		stopPreview()的时候，  
		xxActivity.this 实现的PreviewCallback接口onPreviewFrame方法还在不停	调用，  
		具体调用频率就是当前预览的FrameRate，当stopPreview()执行完_mCamera.release()时，onPreviewFrame再次被调用时就出现了该异常。  
		在调用camera.release之前一定要先调用camera.setpreviewcallback(null)  
		
	>错误发生处
	>>- CaptureActivity line 519  
	>>- ScanActivity2 line 229  

	
	
3. [x] fragment not attach to the activity  

	>造成这种事情的原因是fragment还没有添加到Activity上。因为fragment的上下文	环境是依附在他所附属的Activity上的，在调用getActivity返回值为空时， 	fragment没有上下文环境，调用系统资源会出现此错误。  
	可能会是下面的情况，
	>>- 在create函数中使用系统资源，getStringREsoure()
	>>- 在已经detach之后（网络请求或者异步任务中可能存在），使用toast等的时候

	错误发生处
	
	>- BookFindPageItemFragment  网络请求的 OnSuccess中使用了Toast

	
4. [x] BadTokenException: Unable to add window -- token null is not valid; is your activity running  

 
	>- [x] UnKownBooksActivity popwindow  oncreate 弹框  BookNoticeActivity
	>- [x] UpdatePopupWindow 造成的异常，但是这个类有80个使用类，需要一一排查是哪个使用类造成的异常 ->已查明 是一个叫TrendsDetailActivity的页面在onCreate函数里进行了网络请求，网络请求的毁掉函数中包含弹框

	>popwindow是一个必须依附于Activity的组件，必须在Activity.onCreate()调用结束之后才可以弹出弹框。Activity还没创建完，弹出popwindow是不对的
	
5. [x] 空指针异常  没有判空  

	>错误处：  
	>>- PicUtil bmptoWXByteArray(Bitmap bitmap)    
	>>- CameraPreview onSurfaceChange     
	
	>错误原因    
	>>没有对形参bitmap进行判空操作 导致抛出空指针异常  
	>>没有对 mPreviewSize进行判空操作  
	>  
	  
6. [ ] 数组越界 IndexOutOfBoundException    
 	>unknowbkooklist arraylist.remove(position)  不清楚问什么会有这个问题，
 	>或许是庄海明已经改了

7. [x] recyclerView 异常 validateViewHolderForOffsetPosition 
	> 这是Google的一个错误，解决方法也很粗暴，继承LinearLayoutManager创建一个自己的    	LayoutManager，在onLayout函数中捕捉异常，不让程序崩溃就行。已解决

8. [x] 空指针异常 
	>EditBookActivity   
	>equals 方法 没有进行判空操作
	
	
	
9. [x] RuntimeException: takePicture failed 
	>OcrActivity  
	>在快速点击拍照（连拍）时，系统对上一张照片的处理尚未完成，相机硬件仍在使用中，会造成这个运行时异常
	>解决方法是，启用一个标志位safeTakePic，在拍照时将该位置为false,拍照完成之后，置为true；
	
	
10. [ ] outOfMemory内存溢出  
	>在编辑我的最爱的书籍的页面发生过一次，猜测是因为用户书籍太多，造成链表过大，造成内存溢出。发生概率比较低，回头再进行优化

11. [ ]IllegalStateException: You need to use a Theme.AppCompat theme (or descendant) with this activity	

	>reloginActivity 继承自AppCompatActivity ,在清单文件中的Theme使用的是DialogTheme，因此造成异常  

	>解决方案，重写reloginActivity,发生几率很低，稍后再改


12. [ ] NullPointerException 检查权限时Activity 为空，只发生了一次，尚不清楚原因，待查
	
	
## 3-14日 

1. [x] 相机相关bug的探索和修复

## 3-15日 

1. [ ] universalimageloader.DiskLruCache.checkNotClosed(DiskLruCache.java:642) 
	>  
	>发生次数 0-0-61	
	>错误原因  
	>图片加载框架缓存出现问题，删除缓存的方法不对可能造成异常。  
	>This Exception comes from the checkNotClosed method of DiskLruCache .When the journalWriter is null, 
	>such expception will happen.  
	>When the journal size is cut half, journalWriter will close and start rebuild. Any operation during 
	>this time will cause such Exception.  
	>The journal is under the cache folder,check if it has changed or deleted.  
	>If you want to clear the DiskCache, it is best to use the framework api to clear the DiskCache.  
	>当用户清楚缓存的时候可能造成的错误，解决方法是使用UNiversalImagerLoader 的清楚缓存的方法。
	
	>>
	```java
	java.lang.IllegalStateException: cache is closed
	com.nostra13.universalimageloader.cache.disc.impl.ext.DiskLruCache.checkNotClosed(DiskLruCache.java:642)
	com.nostra13.universalimageloader.cache.disc.impl.ext.DiskLruCache.get(DiskLruCache.java:413)
	com.nostra13.universalimageloader.cache.disc.impl.ext.LruDiskCache.get(LruDiskCache.java:133)
	com.nostra13.universalimageloader.core.ImageLoaderEngine$1.run(ImageLoaderEngine.java:72)
	java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1113)
	java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:588)
	java.lang.Thread.run(Thread.java:833)
	```
	>这个问题还没有被解决，官方也没给出原因。[Github地址](https://github.com/nostra13/Android-Universal-Image-Loader/issues/1147)
	
	
2. [x] recyclerView.LayoutManager 的问题，已解决。  
	>发生次数 0-19-30

3. [x] CaptureActivity.onPause(CaptureActivity.java:519)
Camera is being used after Camera.release() was called ，已解决的问题

	>发生次数 0-14-29

4. [x] UpdatePopupWindow 弹框的问题，
	>在1.4中已经提到过，在今天下班前改好，因为涉及的比较多
	>发生次数 0-11-28
	>-> 原因已查明，TrendsDetailActivity(动态详情页面) oncreate函数中包含网络请求，网络请求的回调函数中有弹框

5. [x]  OtherHomeActivity onsucess 函数里有fragment事务，但是Activity可能已经不存在了

	>发生次数 0-13-17  
	
	> 错误原因:  
	>>activity 已经不存在，还在commit，造成状态异常  
	>>commit();  
	>>commitallowstateslos();  
	 
	>错误发生处
	>>- OthersHomeActivity2 line438    
	 
 	>修复过程  
 	>> commit改为 commitallowstateslos()
 	>
 	
8. [x] fragment not attach to the activity  

	>发生次数 1-10-15  
	错误原因 : 
	>>造成这种事情的原因是fragment还没有添加到Activity上。因为fragment的上下文	环境是依附在他所附属的Activity上的，在调用getActivity返回值为空时， 	fragment没有上下文环境，调用系统资源会出现此错误。  
	可能会是下面的情况，
	>>- 在create函数中使用系统资源，getStringREsoure()
	>>- 在已经detach之后（网络请求或者异步任务中可能存在），使用toast等的时候

	>错误发生处:    
	>>- BookFindPageItemFragment  网络请求的 OnSuccess中使用了Toast
	
7. [x] IllegalArgumentException: Illegal character in query at index 62

	>发生次数 0-11-15  
	>错误原因： 
	>>- URL 采用的编码格式是ASCII码，任何非西欧文字都会造成异常。   
	>>- 在一个URL中一些特殊字符，比如“&”，“=”，这些字符在URL中有特殊的含义，如果你的请求参数中含有这些内容，则可能会造成服务器端对URL的解析异常。 
	 
	>[参考文章:Web开发：URL编码与解码](http://blog.jobbole.com/14442/)  
	>错误发生处：
	>>SearchResultActivity
	
8. [x] BadTokenException: Unable to add window --BookNoticePopWindow

	>发生次数 0-10-14  
	>错误原因：  
	>>Activity的create尚未结束就有popupwindow弹出  
	
	>错误发生处：  
	>>BookNoticePopWindow，使用者UnKownBooksActivitx
	
9. [x] IllegalStateException: Can not perform this action after onSaveInstanceState

	>0-5-14
	>Fragment事务的commit
	>MainActivity
	
10. [ ] SQLiteException: cannot rollback - no transaction is active (code 1)

	> 0-4-4
	>尚不清楚发生的原因，stackOverFlower上给出的结果是：偶然发生在特定设备上的硬件错误
	>

11. [x] NullPointerException PreviewSize CameraPreview 

	> 0-7-13
	>没有在OnSurfaceChange 中判断mPreviewSize是否为空，
	
12. [x] IllegalStateException: Can not perform this action after onSaveInstanceState 

	> 0-9-13
	> commit状态丢失
	
13. [ ] FileUriExposedException: file:///storage/emulated/0/shaishufang/cache/ktemp.kpg exposed beyond app through Intent.getData()

	> 0-4-10
	>Android 7.0 文件权限适配问题
	>未解决
	
14. [x] 问题 同1.2  已解决

15. [x] NullPointerException: Attempt to invoke virtual method 'android.content.res.Resources android.content.Context.getResources()' on a null object reference

	> 0-5-6
	> 不允许在非活动状态的Activity弹出Toast Activity实例不存在
	
16. [ ] ScanResultActivity onActivityResult 的空指针异常

	> 0-4-4
	>不清楚问题出在哪里，但是requestcode 被 写成了resultcode  也许这是错误的出处
	
17. [x] badTokenException 弹框问题 是NewBookDetailActivity

	>项目中有很多popupwindow的问题，大多都是涉及网络请求后发生异常的
	>由于网络请求时间的不确定性，不要再oncreate函数中请求数据（如果数据处理中涉及弹框）
	>错误出处：  
	>> NewBookDetailActivity
	
	
##3月17bug修复

1. [ ]BookFindPageItemFragment onrefresh nullPointerException 没找到原因 可能被修正了

	>
	>>
	```java
	java.lang.NullPointerException
	com.peptalk.client.shaishufang.fragment.BookFindPageItemFragment.onRefresh(BookFindPageItemFragment.java:230)
	com.peptalk.client.shaishufang.fragment.BookFindPageItemFragment.b(BookFindPageItemFragment.java:140)
	com.peptalk.client.shaishufang.fragment.BookFindPagerFragment$8.onClick(BookFindPagerFragment.java:373)
	android.view.View.performClick(View.java:4240)
	android.view.View$PerformClick.run(View.java:17746)
	android.os.Handler.handleCallback(Handler.java:730)
	android.os.Handler.dispatchMessage(Handler.java:92)
	android.os.Looper.loop(Looper.java:137)
	android.app.ActivityThread.main(ActivityThread.java:5178)
	java.lang.reflect.Method.invokeNative(Native Method)
	java.lang.reflect.Method.invoke(Method.java:525)
	com.android.internal.os.ZygoteInit$MethodAndArgsCaller.run(ZygoteInit.java:745)
	com.android.internal.os.ZygoteInit.main(ZygoteInit.java:561)
	dalvik.system.NativeStart.main(Native Method)
	```
2. [ ]NullPointerException   WelcomeFragment.a(WelcomeFragment.java:466)

	> 未找到异常的发生原因
	>>
	```java
	java.lang.RuntimeException: java.lang.NullPointerException
	com.loopj.android.http.AsyncHttpResponseHandler.onUserException(AsyncHttpResponseHandler.java:324)
	com.loopj.android.http.AsyncHttpResponseHandler.handleMessage(AsyncHttpResponseHandler.java:415)
	com.loopj.android.http.AsyncHttpResponseHandler$ResponderHandler.handleMessage(AsyncHttpResponseHandler.java:195)
	android.os.Handler.dispatchMessage(Handler.java:102)
	android.os.Looper.loop(Looper.java:157)
	android.app.ActivityThread.main(ActivityThread.java:5883)
	java.lang.reflect.Method.invokeNative(Native Method)
	java.lang.reflect.Method.invoke(Method.java:515)
	com.android.internal.os.ZygoteInit$MethodAndArgsCaller.run(ZygoteInit.java:871)
	com.android.internal.os.ZygoteInit.main(ZygoteInit.java:687)
	dalvik.system.NativeStart.main(Native Method)
	Caused by : java.lang.NullPointerException
	android.content.ComponentName.(ComponentName.java:77)
	android.content.Intent.(Intent.java:4168)
	com.peptalk.client.shaishufang.fragment.WelcomeFragment.a(WelcomeFragment.java:466)
	com.peptalk.client.shaishufang.fragment.WelcomeFragment.a(WelcomeFragment.java:61)
	com.peptalk.client.shaishufang.fragment.WelcomeFragment$3.onSuccess(WelcomeFragment.java:393)
	com.loopj.android.http.TextHttpResponseHandler.onSuccess(TextHttpResponseHandler.java:98)
	com.loopj.android.http.AsyncHttpResponseHandler.handleMessage(AsyncHttpResponseHandler.java:371)
	```
	
3. [ ] 