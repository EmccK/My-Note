# View事件分发机制

- `(Activity)dispatchTouchEvent`
- 通过`getWindow.superDispatchTouchEvent`，最终实现事件传递到ViewGroup
- `(ViewGroup)dispatchTouchEvent`
- 判断`(ViewGroup)onInterceptTouchEvent`是否为`true`或者没有子View接收事件，为true则进行事件拦截，不再传递，调用ViewGroup父类的`dispatchTouchEvent`，即View的`dispatchTouchEvent`，然后自己处理该事件`onTouch->onTouchEvent->performClick->onClick`
- ViewGroup默认返回`false`，然后在`dispatchTouchEvent`方法中找到被点击的子View，然后将点击事件传递到子View，调用`dispatchTransformedTouchEvent`方法。
- 然后调用`(View)dispatchTouchEvent`
- 然后调用`(View)onTouch`方法，如果返回`true`，则表示事件被消费，不再继续传递，不会调用`onTouchEvent`方法，即不会调用`onClick()`方法
- 如果`(View)onTouch`返回`false`，则调用`onTouchEvent`方法，`performClick`方法，`onClick`方法，只要View的`CLICKABLE`和`LONG_CLICKABLE`有一个为`true`，`onTouchEvent`就会返回`true`，消费掉这个事件。

