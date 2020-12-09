# RecyclerView绘制流程

## onMeasure

分三种情况

1. `LayoutManager`为空，执行默认`onMeasure`方法

2. `LayoutManager`不为空，开启自动测量，然后根据`mState.LayoutStep`的状态（`STEP_START`、`LAOYUT`、`ANIMATIONS`），分别调用`dispatchLayoutStep123`

   1. `STEP_START`：表示`RecyclerView`还未经历`dispatchLayoutStep1`，因为调用`dispatchLayoutStep1`之后会变成`STEP_LAYOUT`
   2. `STEP_LAYOUT`：表示此时正处于`layout`阶段，在这个阶段调用`dispatchLayoutStep2`方法`layout` `RecyclerView`的`children`，然后进入`STEP_ANIMATIONS`阶段。
   3. `STEP_ANIMATIONS`：表示`RecyclerView`处于第三个阶段，即执行动画的阶段，也就是调用`dispatchLayoutStep3`方法，调用完成之后变成默认值，即`STEP_START`

   ----

   ---

   ---

   1. `dispatchLayoutStep1`：处理`Adapter`更新、决定是否执行`ItemAnimator`、保存`ItemView`的动画信息。本方法也被称为`preLayout（预布局）`，当`Adapter`更新时，会保存`ItemView`的旧信息。
   2. `dispatchLayoutStep2`：对`RecyclerView`的子`child`进行真正的测量和布局。
   3. `dispatchLayoutStep3`：执行在`step1`中保存的动画信息。

3. `LayoutManager`为空，则首先判断`setHasFixed`如果为`true`，则直接调用`LayoutManager`的`onMeasure`方法，否则先处理是否有数据更新，如果有数据更新，则先进行更新，然后调用`LayoutManager`的`onMeasure`方法。

## onLayout

这个方法中主要是`dispatchLayout`方法，在这个方法中保证`RecyclerView`可以执行完整`dispatchLayoutStep123`步

如果开启了自动测量，则在`measure`阶段已经将`Children`进行布局完了，如果没有开启，则在`layout`阶段才布局`Children`

## onDraw

分为三步：

1. 调用`supre.draw`方法：将`Children`的绘制分发给`ViewGroup`，将分割线的绘制发给`ItemDecoration`
2. 如果需要的话，调用`ItemDecoration`的`onDrawOver`方法。通过这个方法，我们在每个`ItemView`上面可以画很多东西。
3. 如果`RecyclerView`调用了`setClipToPadding`，会实现一种特殊的滑动效果----每个`ItemView`可以滑动到`Padding`区域

