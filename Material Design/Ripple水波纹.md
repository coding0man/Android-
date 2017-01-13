#Material Design使用
1. [ripple水波纹](https://github.com/coding0man/Android-/blob/master/Material%20Design/Ripple%E6%B0%B4%E6%B3%A2%E7%BA%B9.md)  
2. [BottomNavigationView](https://github.com/coding0man/Android-/blob/master/Material%20Design/BottomNavigationView.md) 


##1.Ripple水波纹效果
[爸爸原文链接](https://developer.android.com/reference/android/graphics/drawable/RippleDrawable.html)  

1. 继承关系  
java.lang.Object  
   ↳	android.graphics.drawable.Drawable  
 	   ↳	android.graphics.drawable.LayerDrawable  
 	 	   ↳	android.graphics.drawable.RippleDrawable
  
2. Added in API level 21(android 5.0)
3. ripple配合selector使用可以创造出更加多变的点击反馈效果

```java
<?xml version="1.0" encoding="utf-8"?>
<ripple xmlns:android="http://schemas.android.com/apk/res/android"
    android:color="#721632">
    <item android:state_pressed="false">
        <shape android:shape="rectangle">
            <solid android:color="@color/white"/>
        </shape>
    </item>
</ripple>
```