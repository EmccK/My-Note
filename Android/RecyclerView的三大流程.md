# RecyclerView的三大流程

## measure

主要分为三种情况：

- `LayoutManager`为空
  - 直接调用`RecyclerView`的`defaultonMeasure`方法。
  - `defaultOnMeasure`方法中：首先调用`LayoutManager`的`chooseSize`方法计算，然后调用`setMeasuredDismission`方法设置宽高。
  - `chooseSize`通过`RecyclerView`的`MeasureSpec`来计算宽高。
- `LayoutManager`不为空，并开启了自动测量
  - 这种情况下主要有**三个方法**：`dispatchLayoutStep1`、`dispatchLayoutStep2`、`dispatchLayoutStep3`
  - 然后`mState.mLayoutStep`的**三个值**对应三个方法：`State.STEP_START`、`State.STEP_LAYOUT`、`State.STEP_ANIMATION`
  - **进入方法**：
    - 首先调用`LayoutManager`的`onMeasure`方法，传统的LayoutManager都没有实现`onMeasure`方法。
      - `onMeasure`方法就是调用`RecyclerView`的`defaultOnMeasure`方法
      - `RecyclerView`将`onMeasure`方法传递给子类实现，但是传统的`LayoutManager`没有实现这个方法，所以在自定义`LayoutManager`时，尽量不要实现`onMeasure`方法
    - 如果`mState.mLayoutStep == State.STEP_START`，进入`dispatchLayoutStep1`方法，然后执行完毕之后进入`dispatchLayoutStep2`方法。
      - `dispatchLayoutStep1`：**1.**处理`Adapter`更新；**2.**判断是否执行`ItemAnimator`；**3.**保存`ItemView`的动画信息。这个方法也被称为`preLayout（预布局）`，当`Adapter`更新时，会保存每个`ItemView`的信息（`oldViewHolderInfo`），从这个方法里面可以看出，`RecyclerView`第一次加载数据的时候不会执行动画。
      - `dispatchLayoutStep2`：测量和布局子类。**1.**首先调用`Adapter`的`getItemCount`获取到子类的数量；**2.**调用`LayoutManager`的`onLayoutChildren`方法，系统的`onLayoutChildren`方法为空，即需要子类自己去实现`onLayoutChildren`方法，自己布局子`View`的位置。
      - `dispatchLayoutStep3`：主要作用就是执行第一步中保存的动画信息。
    - 如果需要二次测量的话，会再次执行`dispatchLayoutStep2`方法。
- `LayoutManager`不为空，未开启自动测量
  - 如果设置了`mHasFixedSize`为`true`，则直接调用`LayoutManager`的`OnMeasure`方法
  - 如果为`false`，并且此时有数据更新的话，先处理数据更新事务，再调用`LayoutManager`的`onMeasure`方法

## layout

直接进入方法，主要是调用了`dispatchLayout`方法

在`dispatchLayout`中，保证了`RecyclerView`必须经历`dispatchLayoutStep`那三个方法，根据`mState.mLayoutStep == State.STEP_START`进入`dispatchLayoutStep1`方法， 然后执行完毕之后执行第二步，最后执行第三步。

在`dispatchLayoutStep3`中，主要是跟`ItemAnimator`有关，并且在这个方法里面将`mState.mLayoutStep`重置为了默认状态。保证下一次重新开始`dispatchLayout`时，`RecyclerView`会经历三个方法

## draw

`draw`是`RecyclerView`三大流程里面的最后一个方法，而`RecyclerView`的`draw`方法主要分为三步

1. 调用`super.draw()`方法，这里主要做两件事：
   - 将`Children`的绘制分发给`ViewGroup`
   - 将分割线的绘制分发给`ItemDecoration`

2. 如果需要的话，调用`ItemDecoration`的`onDrawOver`方法。通过这个方法，我们可以在每个ItemView上画很多东西。
3. 如果`RecyclerView`调用了`setClipToPadding`，则会实现一种特殊的滑动效果-----每个`ItemView`可以滑动到`Padding`区域。

> `ItemDecoration`的`onDrawOver`方法是在`onDraw`方法之后执行的