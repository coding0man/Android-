#Material Design使用
1. [ripple水波纹](https://github.com/coding0man/Android-/blob/master/Material%20Design/Ripple%E6%B0%B4%E6%B3%A2%E7%BA%B9.md)  
2. [BottomNavigationView](https://github.com/coding0man/Android-/blob/master/Material%20Design/BottomNavigationView.md)


##2. BottomNavigationView 底部导航控件

[爸爸原文链接](https://developer.android.com/reference/android/support/design/widget/BottomNavigationView.html)

[中文参考](http://blog.csdn.net/crazy1235/article/details/53458022)

###1. 继承关系

	java.lang.Object  
   		↳	android.view.View  
 	   		↳	android.view.ViewGroup  
 	 	   		↳	android.widget.FrameLayout  
 	 	 	   		↳	android.support.design.widget.BottomNavigationView
 	 	 	   		
###2. 基本用法  

```java
	//布局文件中
	<android.support.design.widget.BottomNavigationView
        android:id="@+id/bottomNavigationView"
        android:layout_width="match_parent"
        android:layout_height="50dp"
        android:layout_alignParentBottom="true"
        android:layout_marginTop="5dp"
        app:itemIconTint="@color/yellow_menu_selector"
        app:itemTextColor="@color/yellow_menu_selector"
        app:menu="@menu/main_bottom_menu"/>
        
   //设置我们需要显示的menu
	<?xml version="1.0" encoding="utf-8"?>
	<menu xmlns:android="http://schemas.android.com/apk/res/android">
    	<item
        	android:orderInCategory="0"
        	android:title="首页"
        	android:icon="@drawable/icon_home_default" />
    	<item
        	android:orderInCategory="1"
        	android:title="消息"
        	android:icon="@drawable/icon_dynamic_default" />
    	<item
        	android:orderInCategory="2"
        	android:title="财富"
        	android:icon="@drawable/icon_treasure_default" />
	</menu>
	
	//activity中
	bottomNavigationView.setOnNavigationItemSelectedListener(new BottomNavigationView.OnNavigationItemSelectedListener() {
            @Override
            public boolean onNavigationItemSelected(@NonNull MenuItem item) {
                viewPager.setCurrentItem(item.getOrder());
                return false;
            }
        });
    
```
###3. 先上两张效果图，第一次进行adb命令录屏，然后工具转GIF，稍后也记录一下。
###4. 源码导读，一看究竟
>Google推荐我们在有3-5个顶层视图时使用这个控件，但在实际使用过程中发现在底部MenuItem的个数大于3的时候显示的形式和我想要的还是有点区别的。  
接下来我们就要去查看是什么造成了这种显示效果的区别。   
BottomNavigationView类包含在这样这样一个成员变量  
 
```java
private final BottomNavigationMenuView mMenuView;
```
	这是我们在xml中设置的menu，我猜出现不同显示效果的原因就是由它控制的，我们去看源码。哎！我看到了这个！
	
```java
mShiftingMode = mMenu.size() > 3;
for (int i = 0; i < mMenu.size(); i++) {
        mPresenter.setUpdateSuspended(true);
        mMenu.getItem(i).setCheckable(true);
        mPresenter.setUpdateSuspended(false);
        BottomNavigationItemView child = getNewItem();
        mButtons[i] = child;
        child.setIconTintList(mItemIconTint);
        child.setTextColor(mItemTextColor);
        child.setItemBackground(mItemBackgroundRes);
        child.setShiftingMode(mShiftingMode);
        child.initialize((MenuItemImpl) mMenu.getItem(i), 0);
        child.setItemPosition(i);
        child.setOnClickListener(mOnClickListener);
        addView(child);
    }
	
```
	
这是个什么玩意儿呢，它是怎么造成显示异常的呢？继续看源码。看到这里
	
```java	 
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        final int width = MeasureSpec.getSize(widthMeasureSpec);
        final int count = getChildCount();

        final int heightSpec = MeasureSpec.makeMeasureSpec(mItemHeight, MeasureSpec.EXACTLY);

        if (mShiftingMode) {
            final int inactiveCount = count - 1;
            final int activeMaxAvailable = width - inactiveCount * mInactiveItemMinWidth;
            final int activeWidth = Math.min(activeMaxAvailable, mActiveItemMaxWidth);
            final int inactiveMaxAvailable = (width - activeWidth) / inactiveCount;
            final int inactiveWidth = Math.min(inactiveMaxAvailable, mInactiveItemMaxWidth);
            int extra = width - activeWidth - inactiveWidth * inactiveCount;
            for (int i = 0; i < count; i++) {
                mTempChildWidths[i] = (i == mActiveButton) ? activeWidth : inactiveWidth;
                if (extra > 0) {
                    mTempChildWidths[i]++;
                    extra--;
                }
            }
        } else {
            final int maxAvailable = width / (count == 0 ? 1 : count);
            final int childWidth = Math.min(maxAvailable, mActiveItemMaxWidth);
            int extra = width - childWidth * count;
            for (int i = 0; i < count; i++) {
                mTempChildWidths[i] = childWidth;
                if (extra > 0) {
                    mTempChildWidths[i]++;
                    extra--;
                }
            }
        } 
    }   
```
	从代码中可以看出，  
	在mShiftingMode = true 时，经过一系列的算法之后确定不同状态下的MenuItem的尺寸，出于选中状态下的menu的宽度是比其他的menuItem 大很多的。由于除法是取整的，最后的循环就是为了将剩下的几个像素分散在前几个MenuItem上。  
	在mShiftingMode =false 时，直接将宽度等分分配给了每一个MenuItem。那我们如果希望在menuItem数量为4甚至5的时候使用等分效果，直接设置mShiftingMode =false不就行了吗？！事实上也就是如此，但是该怎么做呢我们又不能修改源码，难道要重写？算了太麻烦了，还是开启上帝模式吧（反射）。可别高兴的太早，当你这么做了之后，发现虽然是等分了，可是为什么menuTitle没有显示呢？  
	*答案在这里*
	
```java
	public void buildMenuView() {
        if (mButtons != null) {
            for (BottomNavigationItemView item : mButtons) {
                sItemPool.release(item);
            }
        }
        removeAllViews();
        if (mMenu.size() == 0) {
            return;
        }
        mButtons = new BottomNavigationItemView[mMenu.size()];
        mShiftingMode = mMenu.size() > 3;
        for (int i = 0; i < mMenu.size(); i++) {
            mPresenter.setUpdateSuspended(true);
            mMenu.getItem(i).setCheckable(true);
            mPresenter.setUpdateSuspended(false);
            BottomNavigationItemView child = getNewItem();
            mButtons[i] = child;
            child.setIconTintList(mItemIconTint);
            child.setTextColor(mItemTextColor);
            child.setItemBackground(mItemBackgroundRes);
            //！！！！！看这里！！！！！！！
            child.setShiftingMode(mShiftingMode);
            child.initialize((MenuItemImpl) mMenu.getItem(i), 0);
            child.setItemPosition(i);
            child.setOnClickListener(mOnClickListener);
            addView(child);
        }
        mActiveButton = Math.min(mMenu.size() - 1, mActiveButton);
        mMenu.getItem(mActiveButton).setChecked(true);
    }
```
	
	为什么设置了这个就可以了呢？继续看MenuItem的源码。
	
```java
	public void setChecked(boolean checked) {
        mItemData.setChecked(checked);

        ViewCompat.setPivotX(mLargeLabel, mLargeLabel.getWidth() / 2);
        ViewCompat.setPivotY(mLargeLabel, mLargeLabel.getBaseline());
        ViewCompat.setPivotX(mSmallLabel, mSmallLabel.getWidth() / 2);
        ViewCompat.setPivotY(mSmallLabel, mSmallLabel.getBaseline());
        if (mShiftingMode) {
            if (checked) {
                LayoutParams iconParams = (LayoutParams) mIcon.getLayoutParams();
                iconParams.gravity = Gravity.CENTER_HORIZONTAL | Gravity.TOP;
                iconParams.topMargin = mDefaultMargin;
                mIcon.setLayoutParams(iconParams);
                //！！！这里！！！
                mLargeLabel.setVisibility(VISIBLE);
                ViewCompat.setScaleX(mLargeLabel, 1f);
                ViewCompat.setScaleY(mLargeLabel, 1f);
            } else {
                LayoutParams iconParams = (LayoutParams) mIcon.getLayoutParams();
                iconParams.gravity = Gravity.CENTER;
                iconParams.topMargin = mDefaultMargin;
                mIcon.setLayoutParams(iconParams);
                //！！！这里！！！
                mLargeLabel.setVisibility(INVISIBLE);
                ViewCompat.setScaleX(mLargeLabel, 0.5f);
                ViewCompat.setScaleY(mLargeLabel, 0.5f);
            }
            //mShiftingMode模式下 小字体会被隐藏
            mSmallLabel.setVisibility(INVISIBLE);
        } else {
            if (checked) {
                LayoutParams iconParams = (LayoutParams) mIcon.getLayoutParams();
                iconParams.gravity = Gravity.CENTER_HORIZONTAL | Gravity.TOP;
                iconParams.topMargin = mDefaultMargin + mShiftAmount;
                mIcon.setLayoutParams(iconParams);
                //！！！这里！！！
                mLargeLabel.setVisibility(VISIBLE);                					mSmallLabel.setVisibility(INVISIBLE);
                ViewCompat.setScaleX(mLargeLabel, 1f);
                ViewCompat.setScaleY(mLargeLabel, 1f);
                ViewCompat.setScaleX(mSmallLabel, mScaleUpFactor);
                ViewCompat.setScaleY(mSmallLabel, mScaleUpFactor);
            } else {
                LayoutParams iconParams = (LayoutParams) mIcon.getLayoutParams();
                iconParams.gravity = Gravity.CENTER_HORIZONTAL | Gravity.TOP;
                iconParams.topMargin = mDefaultMargin;
                mIcon.setLayoutParams(iconParams);
                //！！！这里！！！
                mLargeLabel.setVisibility(INVISIBLE);
                mSmallLabel.setVisibility(VISIBLE);

                ViewCompat.setScaleX(mLargeLabel, mScaleDownFactor);
                ViewCompat.setScaleY(mLargeLabel, mScaleDownFactor);
                ViewCompat.setScaleX(mSmallLabel, 1f);
                ViewCompat.setScaleY(mSmallLabel, 1f);
            }
        }

        refreshDrawableState();
    }
```
	
	什么 代码太多懒得看，来，我来带你看。很简单啊，根据mShiftingMode的值做了不同处理啊。对什么进行了区别处理？我猜就是文字的显示和隐藏。看15,24和36,47有什么不一样，相信你一定已经看出来了。相信想探底的你肯定还不满足，再来看一段代码。  
	
```java
	<merge xmlns:android="http://schemas.android.com/apk/res/android">
    <ImageView
        android:id="@+id/icon"
        android:layout_width="24dp"
        android:layout_height="24dp"
        android:layout_gravity="center_horizontal"
        android:layout_marginTop="@dimen/design_bottom_navigation_margin"
        android:layout_marginBottom="@dimen/design_bottom_navigation_margin"
        android:duplicateParentState="true" />
    <android.support.design.internal.BaselineLayout
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginBottom="@dimen/design_bottom_navigation_margin"
        android:layout_gravity="bottom|center_horizontal"
        android:duplicateParentState="true">
        <TextView
            android:id="@+id/smallLabel"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:textSize="@dimen/design_bottom_navigation_text_size"
            android:duplicateParentState="true" />
        <TextView
            android:id="@+id/largeLabel"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:visibility="invisible"
            android:textSize="@dimen/design_bottom_navigation_active_text_size"
            android:duplicateParentState="true" />
    </android.support.design.internal.BaselineLayout>
</merge>
```
	看到smallLabel和largeLabel了吗，mShiftingMode模式下 小字体会被隐藏，选中时显示字体，未选中不显示字体（暂时没有考虑动效）。ok,事已至此，针对性的解决问题就好了。来看代码
	
```java
	public class BottomNavigationViewHelper {

        public void disableShiftMode(BottomNavigationView navigationView) {
            BottomNavigationMenuView menuView = (BottomNavigationMenuView) navigationView.getChildAt(0);
            try {
                //修改BottomNavigationMenuView中mShiftingMode变量的值
                //达到均分宽度的效果
                Field shiftingMode = menuView.getClass().getDeclaredField("mShiftingMode");
                shiftingMode.setAccessible(true);
                shiftingMode.setBoolean(menuView, false);
                shiftingMode.setAccessible(false);

                //修改BottomNavigationItemView中mShiftingMode变量的值
                //达到正常显示文字的效果
                for (int i = 0; i < menuView.getChildCount(); i++) {
                    BottomNavigationItemView itemView = (BottomNavigationItemView) menuView.getChildAt(i);
                    itemView.setShiftingMode(false);
                    itemView.setChecked(itemView.getItemData().isChecked());
                }

            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }
```
###5. 实战演练

* MenuItem = 3  参考基础用法


* MenuItem = 4

```java
//XML
<android.support.design.widget.BottomNavigationView
	android:id="@+id/bottomNavigationView"
	android:layout_width="match_parent"
	android:layout_height="50dp"
	android:layout_alignParentBottom="true"
	android:layout_marginTop="5dp"
	app:itemIconTint="@color/yellow_menu_selector"
	app:itemTextColor="@color/yellow_menu_selector"
	app:menu="@menu/main_bottom_menu"/>
	
//MenuItem
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android">
    <item
        android:orderInCategory="0"
        android:title="首页"
        android:icon="@drawable/icon_home_default" />
    <item
        android:orderInCategory="1"
        android:title="消息"
        android:icon="@drawable/icon_dynamic_default" />
    <item
        android:orderInCategory="2"
        android:title="财富"
        android:icon="@drawable/icon_treasure_default" />
    <item
        android:orderInCategory="3"
        android:title="我的"
        android:icon="@drawable/icon_my_default" />
</menu>

private void initNavigationView() {
    BottomNavigationViewHelper.disableShiftMode(bottomNavigationView);
    bottomNavigationView.setOnNavigationItemSelectedListener(new BottomNavigationView.OnNavigationItemSelectedListener() {
            @Override
            public boolean onNavigationItemSelected(@NonNull MenuItem item) {
                viewPager.setCurrentItem(item.getOrder());
                return false;
            }
        });   
}

public class BottomNavigationViewHelper {
    public void disableShiftMode(BottomNavigationView navigationView) {
        BottomNavigationMenuView menuView = (BottomNavigationMenuView) navigationView.getChildAt(0);
        try {
            //修改BottomNavigationMenuView中mShiftingMode变量的值
            //达到均分宽度的效果
            Field shiftingMode = menuView.getClass().getDeclaredField("mShiftingMode");
            shiftingMode.setAccessible(true);
            shiftingMode.setBoolean(menuView, false);
            shiftingMode.setAccessible(false);

            //修改BottomNavigationItemView中mShiftingMode变量的值
            //达到正常显示文字的效果
            for (int i = 0; i < menuView.getChildCount(); i++) {
                BottomNavigationItemView itemView = (BottomNavigationItemView) menuView.getChildAt(i);
                itemView.setShiftingMode(false);
                itemView.setChecked(itemView.getItemData().isChecked());
            }

        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}

```

###6.0 The End



























22