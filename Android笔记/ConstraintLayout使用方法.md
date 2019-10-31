# Constraint Layout使用方法

- 添加依赖

  ```
  implementation 'com.android.support.constraint:constraint-layout:1.1.3'
  ```

- 相对定位

  相对定位是部件对于另一个位置的约束，如下面这个例子

  ![1562585209739](E:\MyDocument\Document\My-Note\Images\1.png)

  `TextView2`在`TextView1`的右边，所以在`TextView2`里面有`...toRightof="@id/text1"`

  `TextView3`在`TextView1`的下面，所以在`TextView3`里面有`...toBottomof=@id/text1`

  

  