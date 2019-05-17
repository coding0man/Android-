# 自定义 ViewPager 的切换效果
本篇内容主要讲述和ViewPager切换效果相关的内容，ViewPager的基本使用，和Tablayout的联动等内容不做阐述，有需要的可以搜索其他相关内容。
先看一个切换效果：  
<!-- <iframe height=300 width=500 src="https://raw.githubusercontent.com/coding0man/Android-/master/attachment/viewPager.gif"> -->

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
> 你千万不要把 position 理解成了 ViewPager 页面的下标，一定要看仔细，这个 position 可是 float 类型，下标怎么可能是浮点型呢！
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

