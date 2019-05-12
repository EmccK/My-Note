# TabLayou+ViewPager使用教程

1. 首先导入包

   ```
   implementation 'com.android.support:design:28.0.0'
   ```

2. `TabLayout`控件

   | tabGravity |                   显示效果                    |      相关方法      |
   | :--------: | :-------------------------------------------: | :----------------: |
   |   center   | ![1557625534447](..\Images\1557625534447.png) | setTabGravity(int) |
   |    fill    | ![1557625602498](..\Images\1557625602498.png) | setTabGravity(int) |

   | tabIndicatorColor |                   显示效果                    |             相关方法              |
   | :---------------: | :-------------------------------------------: | :-------------------------------: |
   |    @color/blue    | ![1557625810690](..\Images\1557625810690.png) | setSelectedTabIndicatorColor(int) |

   | tabIndicatorHeight |                   显示效果                    |              相关方法              |
   | :----------------: | :-------------------------------------------: | :--------------------------------: |
   |        int         | ![1557626354725](..\Images\1557626354725.png) | setSelectedTabIndicatorHeight(int) |

   |  tabMode   |                   显示效果                    |    相关方法     |
   | :--------: | :-------------------------------------------: | :-------------: |
   |   fixed    | ![1557626759051](..\Images\1557626759051.png) | setTabMode(int) |
   | Scrollable | ![1557626654102](..\Images\1557626654102.png) | setTabMode(int) |

   | tabSelectedTextColor |                   显示效果                    |              相关方法              |
   | :------------------: | :-------------------------------------------: | :--------------------------------: |
   |        @color        | ![1557626893261](..\Images\1557626893261.png) | setTabTextColors(normal, selected) |

   | tabTextColor |                   显示效果                    |              相关方法              |
   | :----------: | :-------------------------------------------: | :--------------------------------: |
   |    @color    | ![1557627025420](..\Images\1557627025420.png) | setTabTextColors(normal, selected) |

   `TabLayout`其他用法

   - `void setupWithViewPager(ViewPager viewPager)`

     一键设置`TabLayout`和`ViewPager`

   - `void addTab`

     - `void addTab(TabLayout.Tab tab, boolean setSelected)`
     - `void addTab (TabLayout.Tab tab, int position)`
     - `void addTab (TabLayout.Tab tab)`
     - `void addTab (TabLayout.Tab tab, int position, boolean setSelected)`

   - `void addView`

     - `void addView (View child, int index)`
     - `void addView (View child)`
     - `void addView (View child, ViewGroup.LayoutParams params)`
     - `void addView (View child, int index, ViewGroup.LayoutParams params)`

   - `void clearOnTabSelectedListeners`

     删除所有以前添加的`TabLayout.OnTabSelectedListener`s。

   - `int getSelectedTabPosition()`

     返回当前选项卡的位置

3. `ViewPager`控件

   - 设置`ViewPager`的`Adapter`

     ```java
     //ViewPager与Fragment结合
     mViewPager.setAdapter(new FragmentPagerAdapter(getSupportFragmentManager()) {
                 @Override
                 public Fragment getItem(int position) {
                     return null;
                 }
     
                 @Override
                 public int getCount() {
                     return 0;
                 }
     
                 //设置标题
                 @Nullable
                 @Override
                 public CharSequence getPageTitle(int position) {
                     return null;
                 }
             });
     
     //ViewPager与View结合
     mViewPager.setAdapter(new PagerAdapter() {
         @Override
         public int getCount() {
             return mList.size();
         }
     
         @Override
         public boolean isViewFromObject(@NonNull View view, @NonNull Object o) {
             return view == o;
         }
     
         @NonNull
         @Override
         public Object instantiateItem(@NonNull ViewGroup container, int position) {
             ((ViewPager) container).addView(mList.get(position));
             return mList.get(position);
         }
     
         @Override
         public void destroyItem(@NonNull ViewGroup container, int position, @NonNull Object object) {
             ((ViewPager) container).removeView(mList.get(position));
         }
     });
     ```

   - 设置页面改变监听

     ```java
     mViewPager.addOnPageChangeListener(new ViewPager.OnPageChangeListener() {
                 @Override
                 public void onPageScrolled(int i, float v, int i1) {
     
                 }
     
                 @SuppressLint("RestrictedApi")
                 @Override
                 public void onPageSelected(int i) {
                     
                 }
     
                 @Override
                 public void onPageScrollStateChanged(int i) {
     
                 }
             });
     ```

   - 预加载

     ```java
     //设置预加载的数量
     //您应该将此限制保持在较低水平，尤其是在页面具有复杂布局的情况下。此设置默认为1。
     int setOffscreenPageLimit()
     ```

   - 其他方法

     ```java
     //设置当前选择的页面
     void setCurrentItem(int item)
     //@smoothScorll 是否平滑滚动到新页面
     void setCurrentItem（int item, boolean smoothScroll）
     //返回当前位置的索引
     int getCurrentItem()
     ```

4. 设置`TabLayout`与`ViewPager`结合

   ```
   //一键设置TabLayout和ViewPager
   void setupWithViewPager(ViewPager viewPager)
   ```

