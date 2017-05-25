                                # 自定义view #
## android 屏幕区域划分 ##

![](http://i.imgur.com/6JpbcVN.png)
//获取屏幕区域的宽高等尺寸获取

DisplayMetrics metrics = new DisplayMetrics();
getWindowManager().getDefaultDisplay().getMetrics(metrics);
int widthPixels = metrics.widthPixels;
int heightPixels = metrics.heightPixels;


//应用程序App区域宽高等尺寸获取

Rect rect = new Rect();
getWindow().getDecorView().getWindowVisibleDisplayFrame(rect);

//获取状态栏高度

Rect rect= new Rect();
getWindow().getDecorView().getWindowVisibleDisplayFrame(rect);
int statusBarHeight = rectangle.top;

//View布局区域宽高等尺寸获取

Rect rect = new Rect();  
getWindow().findViewById(Window.ID_ANDROID_CONTENT).getDrawingRect(rect);  

特别注意：上面这些方法最好在Activity的onWindowFocusChanged ()方法或者之后调运，因为只有这时候才是真正的显示OK

## view坐标分析 ##

![](http://i.imgur.com/6fSWaG5.png)
![](http://i.imgur.com/RLaA4j4.png)

## Activity onCreate方法中获取view的width、height、getLeft等为什么都为0？ ##

对于View，ViewGroup来说，width、height、top、left等属性值是在Measure与Layout过程完成之后才开始正确赋值的，而Measure与Layout却都晚于onCreate方法执行，所以onCreate中getLeft根本就取不到值！
解决方法：1. 监听Draw/Layout事件：ViewTreeObserver
2. 将一个runnable添加到Layout队列中：View.post()
3. 重写View的onLayout方法
4. 重写Activity的onWindowFocusChanged方法,在该方法中获取

## view相关坐标的获取方法 ##
![](http://i.imgur.com/anmkbbs.png)
![](http://i.imgur.com/iamzNWF.png)

## 实现view移动的几种方法 ##
![](http://i.imgur.com/p5FjV0i.png)

1.layout

layout(getLeft() + 200, getTop() + 400, getRight() + 200, getBottom() + 400);
![](http://i.imgur.com/3xRIR53.png)
移动后getLeft等值改变

2.offsetLeftAndRight、offsetTopAndBottom

offsetLeftAndRight(200);
offsetTopAndBottom(400);
![](http://i.imgur.com/ha4LTFM.png)
移动后getLeft等值改变

3.修改LayoutParams

 ViewGroup.MarginLayoutParams lp = (ViewGroup.MarginLayoutParams) getLayoutParams();
                lp.leftMargin = getLeft() + 200;
                lp.topMargin = getTop() + 400;
                setLayoutParams(lp);
![](http://i.imgur.com/ZEL5CSA.png)
移动后getLeft等值不改变

4.scrollTo

((View)getParent()).scrollTo(-200,-400);
![](http://i.imgur.com/jxiydfL.png)
移动后getLeft等值不改变

5.scrollBy

((View)getParent()).scrollBy(-200,-400);
![](http://i.imgur.com/QEGAWkO.png)
移动后getLeft等值不改变

对于scrollTo、scrollBy需要注意的有两个问题 
问题1：

移动计算值 = 最开始点坐标 - 最后移动到的坐标 
原因是因为最终会调用这个方法 
—— invalidateInternal(l - scrollX, t - scrollY, r - scrollX, b - scrollY, true, false); 
其中l,t,r,b为原来坐标点，scrollX,scrollY为目标坐标点，只有当目标坐标点值是负数时，移动到的位置才为正数！ 
例如scrollTo ，我们要从（0,0）移动到（200,400）这个点，根据上面的公式可知为负值
问题2

为什么需要加上 ((View)getParent())

TestTextView本身是View，scrollTo、scrollBy移动的都是View的Content，如果不加的话，使用的效果则是TestTextView的文字位置变化，而TestTextView本身不会变化。 
如果在ViewGroup中使用scrollTo、scrollBy，则移动的是ViewGroup中的View.我们这里需要让TestTextView移动，则需要先 ((View)getParent())，然后再((View)getParent()).scrollTo…
6.属性动画

我就以ObjectAnimator为例子

AnimatorSet set = new AnimatorSet();
                set.playTogether(
                        ObjectAnimator.ofFloat(this, "translationX", 200),
                        ObjectAnimator.ofFloat(this,"translationY", 400)

                );
                set.start();
![](http://i.imgur.com/fCRBue2.png)
移动后getLeft等值不改变

7.位移动画

TranslateAnimation anim = new TranslateAnimation(0,200,0,400);
                anim.setFillAfter(true);
                startAnimation(anim);
![](http://i.imgur.com/5LBE1T0.png)
移动后getLeft等值不改变

关于位移动画的补充点：

我们经常用这样的需求，要求一个popupwindow从屏幕底部弹出或者从屏幕顶部弹出。 
这里的位移设置同样还是如此（原点在屏幕左上角，向右x为正，向下y为正）

这篇博文可以参考学习下，下面这张神图也是来自这篇博文（Android动画之translate(位移动画)） 
 [http://www.cnblogs.com/bavariama/archive/2013/01/29/2881225.html](http://www.cnblogs.com/bavariama/archive/2013/01/29/2881225.html)
![](http://i.imgur.com/sVTOpOz.png)

注意点

属性动画是真实改变View的位置的，虽然属性动画、位移动画的getLeft等没有改变，但是属性动画的getX、getY是改变了的，位移动画的getX、getY仍未改变！
