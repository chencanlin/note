                      # view的时间分析 #

## dispatchTouchEvent ##

超链接：[http://blog.csdn.net/yanbober/article/details/45887547](http://blog.csdn.net/yanbober/article/details/45887547 "Android触摸屏事件派发机制详解与源码分析一(View篇)")

dispatchTouchEvent()源码

    public boolean dispatchTouchEvent(MotionEvent event) {
        // If the event should be handled by accessibility focus first.
        if (event.isTargetAccessibilityFocus()) {
            // We don't have focus or no virtual descendant has it, do not handle the event.
            if (!isAccessibilityFocusedViewOrHost()) {
                return false;
            }
            // We have focus and got the event, then use normal event dispatch.
            event.setTargetAccessibilityFocus(false);
        }

        boolean result = false;

        if (mInputEventConsistencyVerifier != null) {
            mInputEventConsistencyVerifier.onTouchEvent(event, 0);
        }

        final int actionMasked = event.getActionMasked();
        if (actionMasked == MotionEvent.ACTION_DOWN) {
            // Defensive cleanup for new gesture
            stopNestedScroll();
        }

        if (onFilterTouchEventForSecurity(event)) {
            //noinspection SimplifiableIfStatement
            ListenerInfo li = mListenerInfo;
            if (li != null && li.mOnTouchListener != null
                    && (mViewFlags & ENABLED_MASK) == ENABLED
                    && li.mOnTouchListener.onTouch(this, event)) {
                result = true;
            }

            if (!result && onTouchEvent(event)) {
                result = true;
            }
        }

        if (!result && mInputEventConsistencyVerifier != null) {
            mInputEventConsistencyVerifier.onUnhandledEvent(event, 0);
        }

        // Clean up after nested scrolls if this is the end of a gesture;
        // also cancel it if we tried an ACTION_DOWN but we didn't want the rest
        // of the gesture.
        if (actionMasked == MotionEvent.ACTION_UP ||
                actionMasked == MotionEvent.ACTION_CANCEL ||
                (actionMasked == MotionEvent.ACTION_DOWN && !result)) {
            stopNestedScroll();
        }

        return result;
    }

if (onFilterTouchEventForSecurity(event))语句判断当前View是否没被遮住等

if (li != null && li.mOnTouchListener != null && (mViewFlags & ENABLED_MASK) == ENABLED && li.mOnTouchListener.onTouch(this, event))语句就是重点，
首先li对象自然不会为null，只要设置了view的onTouchListner那么li.mOnTouchListener就一定不是空的，(mViewFlags & ENABLED_MASK) == ENABLED 这句话的意思是view是不是enabled状态，view的状态一般默认是enabled的（ImageView不是enabled，但是可以在xml声明为enabled或者代码设置），li.mOnTouchListener.onTouch(this, event)这个判断条件就是调用view.onTouch()方法，onTouch（）方法返回值默认是false。如果这个if的条件全部返回true的话，result就是true，那么下面的 if (!result && onTouchEvent(event)) {result = true;}就不会去判断第二个条件，也就是说view的onTouchEvent（）方法不会被调用，如果调用了那么dispatchTouchEvent（）的返回值和onTouchEvent（）返回值一致。

总结：

1、触摸控件（View）首先执行dispatchTouchEvent方法。

2、在dispatchTouchEvent方法中先执行onTouch方法，后执行onClick方法（onClick方法在onTouchEvent中执行，下面会分析）。

3、如果控件（View）的onTouch返回false或者mOnTouchListener为null（控件没有设置setOnTouchListener方法）或者控件不是enable的情况下会调运onTouchEvent，dispatchTouchEvent返回值与onTouchEvent返回一样。

4、如果控件不是enable的设置了onTouch方法也不会执行，只能通过重写控件的onTouchEvent方法处理（上面已经处理分析了），dispatchTouchEvent返回值与onTouchEvent返回一样。

5、如果控件（View）是enable且onTouch返回true情况下，dispatchTouchEvent直接返回true，不会调用onTouchEvent方法。

## View onTouchEvent(MotionEvent ev) 方法源码##

    ` public boolean onTouchEvent(MotionEvent event) {
        final float x = event.getX();
        final float y = event.getY();
        final int viewFlags = mViewFlags;

        if ((viewFlags & ENABLED_MASK) == DISABLED) {
            if (event.getAction() == MotionEvent.ACTION_UP && (mPrivateFlags & PFLAG_PRESSED) != 0) {
                setPressed(false);
            }
            // A disabled view that is clickable still consumes the touch
            // events, it just doesn't respond to them.
            return (((viewFlags & CLICKABLE) == CLICKABLE ||
                    (viewFlags & LONG_CLICKABLE) == LONG_CLICKABLE));
        }

        if (mTouchDelegate != null) {
            if (mTouchDelegate.onTouchEvent(event)) {
                return true;
            }
        }

        if (((viewFlags & CLICKABLE) == CLICKABLE ||
                (viewFlags & LONG_CLICKABLE) == LONG_CLICKABLE)) {
            switch (event.getAction()) {
                case MotionEvent.ACTION_UP:
                    boolean prepressed = (mPrivateFlags & PFLAG_PREPRESSED) != 0;
                    if ((mPrivateFlags & PFLAG_PRESSED) != 0 || prepressed) {
                        // take focus if we don't have it already and we should in
                        // touch mode.
                        boolean focusTaken = false;
                        if (isFocusable() && isFocusableInTouchMode() && !isFocused()) {
                            focusTaken = requestFocus();
                        }

                        if (prepressed) {
                            // The button is being released before we actually
                            // showed it as pressed.  Make it show the pressed
                            // state now (before scheduling the click) to ensure
                            // the user sees it.
                            setPressed(true, x, y);
                       }

                        if (!mHasPerformedLongPress) {
                            // This is a tap, so remove the longpress check
                            removeLongPressCallback();

                            // Only perform take click actions if we were in the pressed state
                            if (!focusTaken) {
                                // Use a Runnable and post this rather than calling
                                // performClick directly. This lets other visual state
                                // of the view update before click actions start.
                                if (mPerformClick == null) {
                                    mPerformClick = new PerformClick();
                                }
                                if (!post(mPerformClick)) {
                                    performClick();
                                }
                            }
                        }

                        if (mUnsetPressedState == null) {
                            mUnsetPressedState = new UnsetPressedState();
                        }

                        if (prepressed) {
                            postDelayed(mUnsetPressedState,
                                    ViewConfiguration.getPressedStateDuration());
                        } else if (!post(mUnsetPressedState)) {
                            // If the post failed, unpress right now
                            mUnsetPressedState.run();
                        }

                        removeTapCallback();
                    }
                    break;

                case MotionEvent.ACTION_DOWN:
                    mHasPerformedLongPress = false;

                    if (performButtonActionOnTouchDown(event)) {
                        break;
                    }

                    // Walk up the hierarchy to determine if we're inside a scrolling container.
                    boolean isInScrollingContainer = isInScrollingContainer();

                    // For views inside a scrolling container, delay the pressed feedback for
                    // a short period in case this is a scroll.
                    if (isInScrollingContainer) {
                        mPrivateFlags |= PFLAG_PREPRESSED;
                        if (mPendingCheckForTap == null) {
                            mPendingCheckForTap = new CheckForTap();
                        }
                        mPendingCheckForTap.x = event.getX();
                        mPendingCheckForTap.y = event.getY();
                        postDelayed(mPendingCheckForTap, ViewConfiguration.getTapTimeout());
                    } else {
                        // Not inside a scrolling container, so show the feedback right away
                        setPressed(true, x, y);
                        checkForLongClick(0);
                    }
                    break;

                case MotionEvent.ACTION_CANCEL:
                    setPressed(false);
                    removeTapCallback();
                    removeLongPressCallback();
                    break;

                case MotionEvent.ACTION_MOVE:
                    drawableHotspotChanged(x, y);

                    // Be lenient about moving outside of buttons
                    if (!pointInView(x, y, mTouchSlop)) {
                        // Outside button
                        removeTapCallback();
                        if ((mPrivateFlags & PFLAG_PRESSED) != 0) {
                            // Remove any future long press/tap checks
                            removeLongPressCallback();

                            setPressed(false);
                        }
                    }
                    break;
            }

            return true;
        }

        return false;
    }`

首先地6到14行可以看出，如果控件（View）是disenable状态，同时是可以clickable的则onTouchEvent直接消费事件返回true，反之如果控件（View）是disenable状态，同时是disclickable的则onTouchEvent直接false。多说一句，关于控件的enable或者clickable属性可以通过Java或者xml直接设置，每个view都有这些属性。

接着22行可以看见，如果一个控件是enable且disclickable则onTouchEvent直接返回false了；反之，如果一个控件是enable且clickable则继续进入过于一个event的switch判断中，然后最终onTouchEvent都返回了true。switch的ACTION_DOWN与ACTION_MOVE都进行了一些必要的设置与置位，接着到手抬起来ACTION_UP时你会发现，首先判断了是否按下过，同时是不是可以得到焦点，然后尝试获取焦点，然后判断如果不是longPressed则通过post在UI Thread中执行一个PerformClick的Runnable，也就是performClick方法。具体如下：

    ` public boolean performClick() {
        final boolean result;
        final ListenerInfo li = mListenerInfo;
        if (li != null && li.mOnClickListener != null) {
            playSoundEffect(SoundEffectConstants.CLICK);
            li.mOnClickListener.onClick(this);
            result = true;
        } else {
            result = false;
        }

        sendAccessibilityEvent(AccessibilityEvent.TYPE_VIEW_CLICKED);
        return result;
    }`

这个方法也是先定义一个ListenerInfo的变量然后赋值，接着判断li.mOnClickListener是不是为null(如果设置了View的OnClickListener就不是null)，决定执行不执行onClick。


总结：

1、onTouchEvent方法中会在ACTION_UP分支中触发onClick的监听。

2、当dispatchTouchEvent在进行事件分发的时候，只有前一个action返回true，才会触发下一个action。


整个view事件分发总结：

1、触摸控件（View）首先执行dispatchTouchEvent方法。

2、在dispatchTouchEvent方法中先执行onTouch方法，后执行onClick方法（onClick方法在onTouchEvent中执行，下面会分析）。

3、如果控件（View）的onTouch返回false或者mOnTouchListener为null（控件没有设置setOnTouchListener方法）或者控件不是enable的情况下会调运onTouchEvent，dispatchTouchEvent返回值与onTouchEvent返回一样。

4、如果控件不是enable的设置了onTouch方法也不会执行，只能通过重写控件的onTouchEvent方法处理（上面已经处理分析了），dispatchTouchEvent返回值与onTouchEvent返回一样。

5、如果控件（View）是enable且onTouch返回true情况下，dispatchTouchEvent直接返回true，不会调用onTouchEvent方法。

6、当dispatchTouchEvent在进行事件分发的时候，只有前一个action返回true，才会触发下一个action（也就是说dispatchTouchEvent返回true才会进行下一次action派发）。

