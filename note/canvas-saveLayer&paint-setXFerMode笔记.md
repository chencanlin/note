#canvans saveLayer 与paint setXFerMode 的PorterDuffXfermode#

saveLayer可以为canvas创建一个新的透明图层，在新的图层上绘制，并不会直接绘制到屏幕上，而会在restore之后，绘制到上一个图层或者屏幕上（如果没有上一个图层）。为什么会需要一个新的图层，例如在处理xfermode的时候，原canvas上的图（包括背景）会影响src和dst的合成，这个时候，使用一个新的透明图层是一个很好的选择。又例如需要当前绘制的图形都带有一定的透明度，那么创建一个带有透明度的图层，也是一个方便的选择。

paint setXferMode时图层的影响：

如果没有创建图层直接在canvas上画图，假如先画一个圆（背景图）在固定的一个矩形区域，然后设置paint的xferMode 为new PorterDuffXfermode(PorterDuff.Mode.SRC_IN) 继续画另外一张bitmap图片(源图像)在这个矩形区域，你的想象中是不是会显示一张圆形的图片（取交集部分显示源图像内容），但是实际上会把你的源图像全部展现出来，这是为什么呢？因为canvas的背景默认不是透明（猜测，因为xfermode实际上与透明度有关系，后面会讲），canvas的背景与之前画的圆圈都被算作了背景图，所以你画源图像时交集部分实际是整个源图像显示的也就是整个源图片，代码如下

	     @Override
	    protected void onDraw(Canvas canvas) {
	//        super.onDraw(canvas);
	        Bitmap bitmap = BitmapFactory.decodeResource(getResources(), R.drawable.guide1);
	        mPaint.setFlags(Paint.ANTI_ALIAS_FLAG);
			// 获取源图片的一块矩形（正方形）区域
	        Rect bitmap1DstRect = getRect(bitmap.getWidth(), bitmap.getHeight());
			// 获取控件画图区域的一块矩形（正方形在画图区域的正中间）区域
	        Rect screenRect = getRect(getWidth(), getHeight());
			去掉下面两行注释代码的话会显示你的期望现象否则就是所说的实际现象
	//        int i = canvas.saveLayer(0, 0, getWidth(), getHeight(), mPaint, Canvas.ALL_SAVE_FLAG);
	        canvas.drawARGB(0,0,0,0);
	        mPaint.setColor(Color.GREEN);
			// 先画一个圆在区域正中间
	        canvas.drawCircle(getWidth() / 2, getHeight() / 2, getWidth() / 2, mPaint);
			// 设置画笔模式为显示源图像交集部分
	        mPaint.setXfermode(new PorterDuffXfermode(PorterDuff.Mode.SRC_IN));
			// 画源图像
	        canvas.drawBitmap(bitmap, bitmap1DstRect, screenRect, mPaint);
	//        canvas.restoreToCount(i);
	    }

为什么 加上下面这两句代码就会出现不同的现象呢？
 int i = canvas.saveLayer(0, 0, getWidth(), getHeight(), mPaint, Canvas.ALL_SAVE_FLAG);

canvas.restoreToCount(i);

调用saveLayer方法实际上是创建一块透明的区域，你在透明区域中间画一个圆（实心圆）但其他部分区域都是透明的，这个时候你在设置paint的xfermode去获取你想要的部分的话就会是你所期望的了，它们的交集部分就变成了这个圆形区域了显示的是源图片内容，也就是把上面注释的代码放开就行了。
