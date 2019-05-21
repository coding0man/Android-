# 自定义 ViewPager 的切换效果
本篇内容主要讲述和ViewPager切换效果相关的内容，ViewPager的基本使用，和Tablayout的联动等内容不做阐述，有需要的可以搜索其他相关内容。
先看一个切换效果：  
<iframe height=480 width=562 src="https://i.bmp.ovh/imgs/2019/05/55466b5c188849f4.giff">

这个效果需要以下知识配合完成
1. clipChild(chilpToPadding)
2. 布局层级，setElevation
3. 还有最重要的ViewPager.PageTransformer


## PageTransformer
关于 ViewPager 的切换动画，官方提供了一个内部接口 ViewPager.PageTransformer 来供我们实现自定义切换动效。这个接口里只提供了一个方法 public void transformPage(View view, float position)。

```java
    /**
     * A PageTransformer is invoked whenever a visible/attached page is scrolled.
     * This offers an opportunity for the application to apply a custom transformation
     * to the page views using animation properties.
     *
     * <p>As property animation is only supported as of Android 3.0 and forward,
     * setting a PageTransformer on a ViewPager on earlier platform versions will
     * be ignored.</p>
     */
    public interface PageTransformer {
        /**
         * Apply a property transformation to the given page.
         *
         * @param page Apply the transformation to this page
         * @param position Position of page relative to the current front-and-center
         *                 position of the pager. 0 is front and center. 1 is one full
         *                 page position to the right, and -1 is one page position to the left.
         */
        void transformPage(@NonNull View page, float position);
    }
```
transformPage 方法有两个参数，一个是 View ，这个好理解就是当前要设置动效的页面。这个页面并不单单是指当前显示的页面，即将滑出的页面、即将滑入的页面、已经隐藏的页面，也就是说这个 View 是指所有的页面。那如何分辨 View 到底是指哪个页面呢，这个需要根据第二个参数 position 来辨别。 
- 不设置PageMargin 
> 千万不要把 position 理解成了 ViewPager 页面的下标，一定要看仔细，这个 position 可是 float 类型，下标怎么可能是浮点型呢！
从 doc 注释来看，当前选中的 item 的 position 永远是 0 ，被选中 item 的前一个为 -1，被选中 item 的后一个为 1。

- 设置PageMargin(ViewPager.setPageMargin)
> 其实这里文档的描述是在ViewPager没有设置PageMargin时的值，
如果你设置了 pageMargin，前后 item 的 position 需要分别加上（或减去，前减后加）一个偏移量（偏移量的计算方式为 pageMargin / pageWidth）。

- 对比  
>
 >也就是说在不设置pageMargin的情况下，viewPager的step是1，各个子View非滑动状态下的排列是：...-3,-2,-1,0,1,2,3...
>   
 >在加了pageMargin后，viewPager的step是（pageMargin+pageWindth）/pageWidth，我们假如该值是1.2。则此时各个子View非滑动状态下的排列是  
 ...-3.6,-2.4,-1.2,0,1.2,2.4,3.6...以此类推
## 不设置PageMargin 的常规情况
在用户滑动界面的时候，position 是动态变化的，下面以左滑为例（以向左为正方向）:

- 选中 item 的 position：从 0 渐至 -1  
- 前一个 item 的 position：从 -1 渐至 -2  
- 前两个 item 的 position：从 -2 渐至 -3，再往前就以此类推  
- 后一个 item 的 position：从 1 渐至 0，再往后就以此类推  

知道了当前View所在的position，根据position知道当前是第几个View，进而算出滑动完成的百分比progress，再根据progress给View一个透明度，一个旋转角度，一个缩放比例。那么当一次滑动时间的所有位置全部回调一遍时组合起来就有了动效。  
下面这个Transformer就实现了越靠近0的位置的View越清晰，越远离越模糊的效果（设置alpha）
```java
public class AlphaTransformer implements ViewPager.PageTransformer {
    @Override
    public void transformPage(@NonNull View view, float position) {
        if (position < -1){
            view.setAlpha((float) 0.2);
        }else if (position <= 0) {
            // 中心位置左边第一个
            Log.e("===",position+"");
            double progress = -position;
            view.setAlpha((float) (1-0.8*progress));
        } else if (position <= 1) {
            // 中心位置右边的第一个
            float progress = 1 - position;
            view.setAlpha((float) (0.2+0.8*progress));
        } else if (position > 1) {
            // 中心位置右边的第二个 第三个 第四个等
            view.setAlpha((float) 0.2);
        }
    }
}
```
基本步骤如下
1. 确定View需要变化的属性  
2. 确定该属性的初始值，终值  
3. 确定该View对应的position变化的梯度  
4. 根据position的变化梯度，计算出需要变化的属性的变化梯度  
5. 剩下的就是调用属性动画的API了
6. 对多个属性分别按照12345的顺序进行分析设计和编码后就能实现各种复杂酷炫  

## 设置PageMargin的情况
前边介绍的时候已经说了，在加了PageMargin之后，非滑动状态下的默认位置不再是-2 -1 0 1 2这样了。  
 那么 在加了PageMargin之后的非滑动状态下的默认位置的position值是多少呢？  
 要回答这个问题，不妨先看看position值是怎么算出来的：

```java
/**
     * This method will be invoked when the current page is scrolled, either as part
     * of a programmatically initiated smooth scroll or a user initiated touch scroll.
     * If you override this method you must call through to the superclass implementation
     * (e.g. super.onPageScrolled(position, offset, offsetPixels)) before onPageScrolled
     * returns.
     *
     * @param position Position index of the first page currently being displayed.
     *                 Page position+1 will be visible if positionOffset is nonzero.
     * @param offset Value from [0, 1) indicating the offset from the page at position.
     * @param offsetPixels Value in pixels indicating the offset from position.
     */
    @CallSuper
    protected void onPageScrolled(int position, float offset, int offsetPixels) {
        // 执行滑动相关的代码
        ...
        ...

        // PageTransformer相关的代码
        if (mPageTransformer != null) {
            final int scrollX = getScrollX();
            final int childCount = getChildCount();
            for (int i = 0; i < childCount; i++) {
                final View child = getChildAt(i);
                final LayoutParams lp = (LayoutParams) child.getLayoutParams();

                if (lp.isDecor) continue;
                final float transformPos = (float) (child.getLeft() - scrollX) / getClientWidth();
                mPageTransformer.transformPage(child, transformPos);
            }
        }

        mCalledSuper = true;
    }

    private int getClientWidth() {
        return getMeasuredWidth() - getPaddingLeft() - getPaddingRight();
    }
```

关键点就是一行代码：
```java
final float transformPos = (float) (child.getLeft() - scrollX) / getClientWidth();
```
先看初始状态：  
// todo 这里补一张图

这里我们假设ViewPager的子View没有设置Padding值，即getClientWidth=View的宽度。  
我们很容易得出如下的表格：  
|View|ViewPager.scrollX|View.getLeft|getClientWidth|transformPos|
|:-:|:-:|:-:|:-:|:-:|
|view0|0|0|100|0|
|view1|0|100+10|100|1.1|
|view2|0|100+10+100+10|100|2.2|

现在我们假设现在滑动到了第二页：
此时ViewPager.scrollX不再是0而是100+10，我们可以再次计算一下：
|View|ViewPager.scrollX|View.getLeft|getClientWidth|transformPos|
|:-:|:-:|:-:|:-:|:-:|
|view0|100+10|0|100|-1.1|
|view1|100+10|100+10|100|0|
|view2|100+10|100+10+100+10|100|1.1|

由此可见当前选中的item的position永远是0。左右相邻的两个Item之间的position的差距始终是(pageWidth+pageMargin)/PageWidth,在本例中该值为1.1。

知道了position的确切含义，我想接下来的问题一定难不住你。只需要提前计算好step = (pageWidth+pageMargin)/PageWidth的值，然后以step作为if条件的分割，然后用progress替代原来的position进行动效的设计就可以了。  
### 一个简单的通用示例代码：  
设置可以作为通用代码，因为它在是否设置PageMargin的情况下都适用。
```java
public class AlphaTransformer implements ViewPager.PageTransformer {
    private float mStep;
    @Override
    public void transformPage(View view, float position) {
        if (mStep == 0){
            mStep = 1+((ViewPager) view.getParent()).getPageMargin()/(float)view.getWidth();
        }
        if (position < -mStep){
            view.setAlpha((float) 0.2);
        }else if (position <= 0) {
            // 中心位置左边第一个
            Log.e("===",position+"");
            double progress = -position/mStep;
            view.setAlpha((float) (1-0.8*progress));
        } else if (position <= mStep) {
            // 中心位置右边的第一个
            float progress = 1 - position/mStep;
            view.setAlpha((float) (0.2+0.8*progress));
        } else if (position > mStep) {
            // 中心位置右边的第二个 第三个 第四个等
            view.setAlpha((float) 0.2);
        }
    }
}
```

## 一个稍显复杂的卡片折叠效果
todo 加入图片
如图展示的效果，选中的图片是正常显示的，两边的图片的尺寸比选中的图片小，他会盖在左侧的图片上并且右侧的图片上，左侧的图片不会完全滑出。  
- 可以一次显示多张图片，只要ViewPager的父布局不限制ViewPager的显示区域即可
- 右侧的图片显示在上方，可以通过设置elevation来实现
- 左侧的图片不完全滑出屏幕，在滑动中形成了一种视差效果，我们通过在滑动中不断平移左侧图片的位置实现
- 选中图的两侧的卡片的比选中的小，可以在滑动中不断设置卡片的缩放比例完成

```xml
    <FrameLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:background="#6A7075"
        android:paddingStart="8dp"
        android:paddingEnd="16dp"
        android:clipChildren="false">

        <android.support.v4.view.ViewPager
            android:id="@+id/viewPager"
            android:layout_marginTop="30dp"
            android:layout_marginBottom="30dp"
            android:layout_marginStart="8dp"
            android:layout_marginEnd="60dp"
            android:layout_width="match_parent"
            android:clipChildren="false"
            android:layout_height="140dp"/>
    </FrameLayout>
```

```java
    private void initViewPager() {
        ViewPager mViewPager = findViewById(R.id.viewPager);
        mViewPager.setPageMargin(ScreenUtils.dip2px(16));
        mViewPager.setOffscreenPageLimit(3);
        mViewPager.setAdapter(new PagerAdapter() {

            @Override
            public Object instantiateItem(ViewGroup container, int position) {

                View view = new View(MainActivity.this);
                switch (position) {
                    case 0:
                        view.setBackgroundResource(R.drawable.red);
                        break;
                    case 1:
                        view.setBackgroundResource(R.drawable.green);
                        break;
                    case 2:
                        view.setBackgroundResource(R.drawable.blue);
                        break;
                }
                // 使右侧的图片在上
                view.setElevation(position);
                container.addView(view);
                return view;
            }

            @Override
            public void destroyItem(ViewGroup container, int position, Object object) {
                container.removeView((View) object);
            }

            @Override
            public int getCount() {
                return 3;
            }

            @Override
            public boolean isViewFromObject(View view, Object o) {
                return view == o;
            }
        });
        mViewPager.setPageTransformer(false, new CustomerTransformer());
    }
```

```java
public class CustomerTransformer implements ViewPager.PageTransformer {
    private float mStep;
    private int mPageWidth;
    private int mPageHeight;

    @Override
    public void transformPage(View view, float position) {
        if (mStep == 0){
            ViewPager viewPager = (ViewPager) view.getParent();
            mPageWidth = view.getWidth();
            mPageHeight = view.getHeight();
            mStep = 1+viewPager.getPageMargin()/(float)mPageWidth;
        }
        // 如果不加pageMargin，分界值会是-1,0,1这种
        // 因为我们加了pageMargin，所以需要先计算出当前pageMargin下的分界值
        // 先计算出分界值

        if (position <=  -mStep) {
            // 中心位置左边第二个
            // 1、先根据position和临界值算出滑动到下个位置的progress
            // 2、缩放原则是从0.9变到0.8。据此算出缩放比例。
            double progress = Math.abs(position / mStep) - 1;
            float scaleFactor = (float) (0.9f - progress * 0.1f);
            view.setPivotY(mPageHeight >> 1);
            view.setPivotX(0);
            view.setScaleX(scaleFactor);
            view.setScaleY(scaleFactor);
            
            float transX = (float) (mPageWidth * -position - ScreenUtils.dip2px(5) + ScreenUtils.dip2px(3) * -progress);
            view.setTranslationX(transX);
            float alpha = (float) (0.9 - 0.45 * progress);
            view.setAlpha(alpha);
        } else if (position <= 0) {
            // 中心位置左边第一个
            view.setPivotY(mPageHeight >> 1);
            view.setPivotX(0);
            double progress = Math.abs(position / mStep);
            float scaleFactor = (float) (1 - progress * 0.1f);
            view.setScaleX(scaleFactor);
            view.setScaleY(scaleFactor);
            // 根据position算出如果不做任何操作，当前View应该在的位置
            // 根据progress算出需要向左偏的距离,
            // 两者相加得到我们希望当前View所在的位置
            int transX = (int) (mPageWidth * -position + ScreenUtils.dip2px(5) * -progress);
            view.setTranslationX(transX);
            // 根据progress计算出希望的透明度
            float alpha = (float) (1 - 0.1 * progress);
            view.setAlpha(alpha);
        } else if (position <= mStep) {
            // 中心位置右边的第一个
            float progress = 1 - Math.abs(position / mStep);
            float scaleFactor = 0.9f + progress * 0.1f;
            view.setPivotX(0);
            view.setPivotY(mPageHeight >> 1);
            view.setScaleX(scaleFactor);
            view.setScaleY(scaleFactor);
        } else if (position > mStep) {
            // 中心位置右边的第二个 第三个 第四个等
            float scaleFactor = 0.9f;
            view.setPivotX(0);
            view.setPivotY(mPageHeight >> 1);
            view.setScaleX(scaleFactor);
            view.setScaleY(scaleFactor);
        }
    }
}
```

