                    # 自定义view #
## 1、view是什么 ##
View是屏幕上的一块矩形区域，它负责用来显示一个区域，并且响应这个区域内的事件。可以说，手机屏幕上的任意一部分看的见得地方都是View，它很常见，比如 TextView 、ImageView 、Button以及LinearLayout、RelativeLayout都是继承子View的。
 对于Activity来说，我们通过setContentView(view)添加的布局到Activity上，实际上都是添加到了Activity 内部的DecorView上面，这个DecorView，其实就是一个FrameLayout,因此实际上，我们的布局实际上添加到了FrameLayout里面。
![](http://i.imgur.com/WIJIbNC.png)

## 2、view是怎么工作的 ##
View的工作流程分为两部分，第一部分 显示在屏幕上的过程, 第二部分 响应屏幕上的触摸事件的过程。对于显示在屏幕上的过程：是View 从无到有，经过测量大小(Measure)、确定在屏幕中的位置(Layout)、以及最终绘制在屏幕上(Draw) 这一系列的过程。对于响应屏幕上的触摸事件的过程：则是Touch事件的分发过程(这一部分，在这里先不涉及)。 Measure() Layout()方法是final修饰的，无法重写 ，Draw()虽然不是final的，但是也不建议重写该方法。

## 3、如何自定义view ##
View 为我们提供了 onMeasure()  onLayout()  onDraw() 这样的方法，其实所谓的自定义View，也就是对onMeasure() onLayout() onDraw() 三个方法的重写的过程。所谓的自定义View 实际上，就是仿照View的工作流程，去自己实现的过程。根据继承对象的不同自定义View又分为继承View 与ViewGroup两种。

## measure() ##:
View的onMeasure()方法如下：在CODE上查看代码片派生到我的代码片
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {  
    setMeasuredDimension(getDefaultSize(getSuggestedMinimumWidth(), widthMeasureSpec),  
            getDefaultSize(getSuggestedMinimumHeight(), heightMeasureSpec));  
}  
  内部调用了 setMeasuredDimension() 方法，这个方法看名字就知道是设置测量的View的宽/高，传递的参数就是宽度和高度。
  onMeasure通过父View传递过来的大小和模式，以及自身的背景图片的大小得出自身最终的大小，通过setMeasuredDimension()方法设置给mMeasuredWidth和mMeasuredHeight。
  普通View的onMeasure逻辑大同小异，基本都是测量自身内容和背景，然后根据父View传递过来的MeasureSpec进行最终的大小判定，例如TextView会根据文字的长度，文字的大小，文字行高，文字的行宽，显示方式，背景图片，以及父View传递过来的模式和大小最终确定自身的大小。
 ViewGroup本身没有实现onMeasure，但是他的子类都有各自的实现，通常他们都是通过measureChildWithMargins函数或者其他类似于measureChild的函数来遍历测量子View，被GONE的子View将不参与测量，当所有的子View都测量完毕后，才根据父View传递过来的模式和大小来最终决定自身的大小。
  ViewGroup一般都在测量完所有子View后才会调用setMeasuredDimension()设置自身大小。
  这里我们就要介绍一下Android中自己定义的测量规格：MeasureSpec。
  因为测量过程中，系统会把我们代码或者布局中设置的View的LayoutParams 转换成MeasureSpec，然后通过MeasureSpec来测量确定View的宽高。
  MeasureSpec由32位 int值组成，高2位表示的是测量模式(specMode),后面的30位代表的是测量的大小(specSize)。![](http://i.imgur.com/7J7Cq3q.png)经过measure 完成后，我们就可以通过getMeasuredWidth/Height 获取View 的宽高。(注意此时还不能用函数getWidth（）或者getHeight（）来获取宽高，等到onLayout（）完成之后才能获取，因为getWidth（）内部实现实际是右边坐标减去左边坐标，getHeight（）同理)；
## layout（） ##
 Layout() 方法如果是ViewGroup，则循环遍历所有子View，普通View则空实现,因此如果我们继承ViewGroup 我们需要遍历执行所有的child.layout()。
 Layout方法中接受四个参数，是由父View提供，指定了子View在父View中的左、上、右、下的位置。父View在指定子View的位置时通常会根据子View在measure中测量的大小来决定。
子View的位置通常还受有其他属性左右，例如父View的orientation，gravity，自身的margin等等，影响布局的因素非常多。
onLayout是ViewGroup用来决定子View摆放位置的，各种布局的差异都在该方法中得到了体现。
onLayout比layout多一个参数，changed，该参数是在setFrame通过比对上次的位置得出是否发生了变化，通常该参数没有被使用的意义，因为父View位置和大小不变，并不能代表子View的位置和大小没有发生改变。
layout布局流程图：![](http://i.imgur.com/6ZfzseL.png)
## draw（） ##
 draw()的过程就是绘制View到屏幕上的过程，draw()的执行遵循如下步骤：
       -  * Draw traversal performs several drawing steps which must be executed 
-         * in the appropriate order: 
-         * 
-         *      Draw the background(绘制背景) 
-         *      2. If necessary, save the canvas' layers to prepare for fading(保存画布的图层来准备色变) 
-         *      3. Draw view's content(绘制内容) 
-         *      4. Draw children(绘制children) 
-         *      5. If necessary, draw the fading edges and restore layers(画出褪色的边缘和恢复层) 
-         *      6. Draw decorations (scrollbars for instance)(绘制装饰 比如scollbar) 
  一般2和5 是可以跳过的。
  在TextView中在该方法中绘制文字、光标和CompoundDrawable 、ImageView中相对简单，只是绘制了图片。
  View 的绘制主要通过dispatchDraw()
先根据自身的padding剪裁画布，所有的子View都将在画布剪裁后的区域绘制。
遍历所有子View，调用子View的computeScroll对子View的滚动值进行计算。
根据滚动值和子View在父View中的坐标进行画布原点坐标的移动，根据子在父View中的坐标计算出子View的视图大小，然后对画布进行剪裁，请看下面的示意图。
dispatchDraw的逻辑其实比较复杂，但是幸运的是对子View流程都采用该方式，而ViewGroup已经处理好了，我们不必要重载该方法对子View进行绘制事件的派遣分发。![](http://i.imgur.com/2YGoLEj.png)
理解了以上工作流程，就可以试着去自定义View了。
  自定义View一般就分为 继承View/VieGroup两种，在我们自定义View的时候要让自定义的View的功能尽可能向
系统提供的考虑，比如支持wrap_content,padding 等。

View 的几个比较重要的方法：

  requestLayout:
  当view确定自身已经不再适合现有的区域时，该view本身调用这个方法要求parent view（父类的视图）重新调用他的onMeasure onLayout来重新设置自己位置。特别是当view的layoutparameter发生改变，并且它的值还没能应用到view上时，这时候适合调用这个方法。

  postInvalidate 与 invalidate

  界面刷新 onDraw方法会执行，区别就是Invalidate不能直接在线程中调用，因为他是违背了单线程模型：Android UI操作并不是线程安全的，并且这些操作必须在UI线程中调用。 鉴于此，如果要使用invalidate的刷新，那我们就得配合handler的使用，使异步非ui线程转到ui线程中调用，如果要在非ui线程中直接使用就调用postInvalidate方法即可，这样就省去使用handler的烦恼。