### 前言
首先请大家看几张图：
![](http://note.youdao.com/yws/public/resource/a1960d4a5bc124121e27e7f5fc52baf8/xmlnote/WEBRESOURCEbc97e05cb264dd785a924d711d85b612/10733)
以上的效果，一般我们统称为沉浸式状态栏。其实，这是叫法不是很准确，而且也没有沉浸式状态栏这一说，只有沉浸模式。以上几种情况，可以称为透明状态栏或者状态栏着色。

### 一、两种状态
进行Android开发的我们都应该知道Translucent Bar（透明栏）和Immersive Mode（沉浸模式），两种方式都会对状态栏进行设置。两者的区别，比较直观的一点，就是体现在屏幕中的View可点击区域，如下所示。

+ 沉浸模式  
隐藏status bar（状态栏）,使屏幕全屏，让Activity接收所有的（整个屏幕的）触摸事件。如上面所示的直播全屏播放。
+ 状态栏着色  
设置状态栏颜色，布局侵入状态栏的后面，必须启用`fitsSystemWindows`属性来调整布局才不至于被状态栏覆盖。如透明状态栏、与titlebar颜色一致的状态栏。

### 二、如何沉浸
从3.x版本开始, 系统DecorView提供了setSystemUiVisibility方法, 可以通过Flag改更改所谓SystemUI的属性。各个设置的参数含义如下所示：

```
View.SYSTEM_UI_FLAG_VISIBLE Level 14  
默认标记

View.SYSTEM_UI_FLAG_LOW_PROFILE Level 14  
低功耗模式, 会隐藏状态栏图标, 在4.0上可以实现全屏

View.SYSTEM_UI_FLAG_LAYOUT_STABLE Level 16  
保持整个View稳定, 常跟bar 悬浮, 隐藏共用, 使View不会因为SystemUI的变化而做layout

View.SYSTEM_UI_FLAG_FULLSCREEN Level 16  
状态栏隐藏

View.SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN Level 16  
状态栏上浮于Activity

View.SYSTEM_UI_FLAG_HIDE_NAVIGATION Level 14  
隐藏导航栏, 
=========================================================================
4.0 - 4.3如果使用这个属性,将会导致在下一次touch时候自动show出status跟navigation bar,源于系统clear掉其所有的状态
=========================================================================

View.SYSTEM_UI_FLAG_LAYOUT_HIDE_NAVIGATION Level 16  
导航栏上浮于Activity

View.SYSTEM_UI_FLAG_IMMERSIVE Level 19  
Kitkat新加入的Flag, 沉浸模式, 可以隐藏掉status跟navigation bar, 并且在第一次会弹泡提醒, 它会覆盖掉之前两个隐藏bar的标记, 并且在bar出现的位置滑动可以呼出bar

View.SYSTEM_UI_FLAG_IMMERSIVE_STIKY Level 19  
与上面唯一的区别是, 呼出隐藏的bar后会自动再隐藏掉

```

综上所述，要实现全屏沉浸，我们只需如此设置即可：

```
//Hide
getWindow().getDecorView().setSystemUiVisibility(
                View.SYSTEM_UI_FLAG_LAYOUT_STABLE
                        | View.SYSTEM_UI_FLAG_LOW_PROFILE
                        | View.SYSTEM_UI_FLAG_LAYOUT_HIDE_NAVIGATION
                        | View.SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN
                        | View.SYSTEM_UI_FLAG_FULLSCREEN
                        | View.SYSTEM_UI_FLAG_IMMERSIVE);
```  

![放一张全屏沉浸的图]()

### 三、如何着色

#### 3.1 两个系统节点
对于状态栏着色，有两个比较关键的系统节点需要关注，分别是4.4和5.0。基于两个系统节点，我们又可以分成三个阶段进行讨论。

+ 4.4以前  
状态栏不支持设置颜色。

+ 4.4 ～ 5.0  
状态栏支持透明效果，但是系统不提供接口进行颜色设置（有办法，文章后面会介绍）。

+ 5.0以上  
状态栏不仅支持透明效果，系统也提供接口对状态栏进行颜色设置。

#### 3.2 实现的两种方式
##### 3.2.1 主题和布局设置（相关坑）
1. 在values、values-v19、values-v21的style.xml都设置一个 Translucent System Bar 风格的Theme
	+ values/style.xml
	
	```
	<style name="ImageTranslucentTheme" parent="AppTheme">
    	<!--在Android 4.4之前的版本上运行，直接跟随系统主题-->
	</style>
	```
	+ values-v19/style.xml
	
	```
	 <style name="ImageTranslucentTheme" parent="Theme.AppCompat.Light.NoActionBar">
        <item name="android:windowTranslucentStatus">true</item>
    </style>	
    ```
	
	+ values-v21/style.xml
	
	```
	<style name="ImageTranslucentTheme" parent="Theme.AppCompat.Light.NoActionBar">
        <item name="android:windowTranslucentStatus">false</item>
        <!--Android 5.x开始需要把颜色设置透明，否则导航栏会呈现系统默认的浅灰色-->
        <item name="android:statusBarColor">@android:color/transparent</item>
    </style>
	```
2. 在AndroidManifest.xml中对指定Activity的theme进行设置

```
<activity android:name=".MainActivity"
    android:theme="@style/ImageTranslucentTheme"
    >
</activity>
```
3. 在Activity的布局文件中设置背景图片，同时，需要把android:fitsSystemWindows设置为true

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/activity_main"
    android:orientation="vertical"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@mipmap/paperscan_guide_step_two_inside"

    >

    <TextView
        android:text="透明状态栏~"
        android:textColor="@android:color/white"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content" />
</LinearLayout>
```
如此可以得到效果如下所示  
![实现透明效果的图]()

那么，为什么要将android:fitsSystemWindows设置为true呢？如果不设置会怎么样？我们可以来测试下，下图是不设置（默认为false）的效果。可见内容跑到了状态栏的下面，如果状态栏不是透明的，内容就被覆盖了。所以，最直观的一点我们可以发现，设置了android:fitsSystemWindows为true，可以让内容不会顶到状态栏的下面。文章[Android沉浸式状态栏实现](http://ks.netease.com/blog?id=6650)对`itsSystemWindows`已经做了详细的分析，大家有兴趣可以看看。
![android:fitsSystemWindows为false的图]()

接着上面的问题，既然我们实现透明状态栏效果的页面都需要设置`fitsSystemWindows`属性，所以我们想到了一种方便的方法，在theme中加上如下的android:fitsSystemWindows设置：

```
<item name="android:fitsSystemWindows">true</item>
```
运行发现真的可以，显示的内容也没有和状态栏重叠。但是，当我们显示一个toast的时候，发现问题了。

![toast错位的图片]()

如图所示，Toast打印出来的文字都向上便宜了。原因是因为我们是在Theme中设置的fitsSystemWindows属性，会影响使用了该theme的activity或application的行为，造成依附于Application Window的Window（比如Toast）错位。针对Toast错位的问题，解决方法也简单，就是**使用应用级别的上下文**：

```
Toast.makeText(getApplicationContext(),"toast sth...",Toast.LENGTH_SHORT).show();
```
虽说Toast错误问题也是有方法可以解决，但是如果这样使用，不经意间会给我们的应用埋下很多坑。所以我的建议是：不要滥用，只在有需要的地方添加fitsSystemWindows属性。

##### 3.2.2 代码设置
正如前面提到的，只有4.4以上的系统才支持透明状态栏设置，5.0以上的系统还支持设置状态栏任意颜色。所以5.0以上的系统设置状态栏的颜色就很简单了，跟着系统给的api走就可以了：

```
// 设置此flag才可对状态栏进行颜色设置
activity.getWindow().addFlags(WindowManager.LayoutParams.FLAG_DRAWS_SYSTEM_BAR_BACKGROUNDS);
// 取消设置透明状态栏，不然颜色设置不生效
activity.getWindow().clearFlags(WindowManager.LayoutParams.FLAG_TRANSLUCENT_STATUS);
// 设置状态栏颜色
activity.getWindow().setStatusBarColor(color);
```

而对于系统是4.4到5.0之前的机子，要设置状态栏的颜色就稍微要繁琐一点了。首先，需要设置页面状态栏为透明；然后，新建一个和状态栏高度一致的view，填充到DecorView上；最后，通过设置这个填充view的颜色，我们就能实现类似对状态栏颜色进行控制的效果了。

![看情况是否需要示意图]()

```
// 设置透明
activity.getWindow().addFlags(WindowManager.LayoutParams.FLAG_TRANSLUCENT_STATUS);
// 生成view填充DecorView
ViewGroup decorView = (ViewGroup) activity.getWindow().getDecorView();
View fakeStatusBarView = decorView.findViewById(FAKE_STATUS_BAR_VIEW_ID);
if (fakeStatusBarView != null) {
    if (fakeStatusBarView.getVisibility() == View.GONE) {
        fakeStatusBarView.setVisibility(View.VISIBLE);
    }
    fakeStatusBarView.setBackgroundColor(calculateStatusColor(color, statusBarAlpha));
} else {
    decorView.addView(createStatusBarView(activity, color, statusBarAlpha));
}
// 设置fitsSystemWindows属性
setRootView(activity);
```

#### 3.3 着色原理
在文章的前面部分，在系统4.4～5.0的机子上，如何设置状态栏颜色的原理其实已经做了说明。就是在透明的状态栏下放置一个和状态栏大小一致的view，通过更改view的颜色来实现更改状态栏颜色的效果。那么，当在5.0及以上的机子，当我们通过如下代码设置状态栏颜色:

```
getWindow().setStatusBarColor(RED);
```
其实，这里调用的是PhoneWindow的setStatusBarColor方法，具体实现如下：

```
@Override
public void setStatusBarColor(int color) {
    mStatusBarColor = color;
    mForcedStatusBarColor = true;
    if (mDecor != null) {
        mDecor.updateColorViews(null, false /* animate */);
    }
}
```

最终调用的是DecorView的updateColorViews函数，DecorView是属于Activity的PhoneWindow的内部对象，也就说，更新的对象从所谓的Window进入到了Activity自身的布局视图中，接着看DecorView，这里只关注更改颜色。在方法updateColorViews中，调用了如下代码，代码实现的功能就是修改状态栏颜色：

```
updateColorViewInt(mNavigationColorViewState, sysUiVisibility,
    mWindow.mNavigationBarColor, navBarSize, navBarToRightEdge || navBarToLeftEdge,
    navBarToLeftEdge,
    0 /* sideInset */, animate && !disallowAnimate, false /* force */);

```

这里mStatusColorViewState其实就代表StatusBar的背景颜色对象，主要属性包括显示的条件以及颜色值：

```
 private final ColorViewState mStatusColorViewState = new ColorViewState(
        SYSTEM_UI_FLAG_FULLSCREEN, FLAG_TRANSLUCENT_STATUS,
        Gravity.TOP,
        Gravity.LEFT,
        STATUS_BAR_BACKGROUND_TRANSITION_NAME,
        com.android.internal.R.id.statusBarBackground,
        FLAG_FULLSCREEN);
```

再来看updateColorViewInt方法：

```
private void updateColorViewInt(final ColorViewState state, int sysUiVis, int color,
        int size, boolean verticalBar, boolean seascape, int sideMargin,
        boolean animate, boolean force) {
    // 关键点1  
    state.present = (sysUiVis & state.systemUiHideFlag) == 0
            && (mWindow.getAttributes().flags & state.hideWindowFlag) == 0
            && ((mWindow.getAttributes().flags & FLAG_DRAWS_SYSTEM_BAR_BACKGROUNDS) != 0
                    || force);
    // 关键点2
    boolean show = state.present
            && (color & Color.BLACK) != 0
            && ((mWindow.getAttributes().flags & state.translucentFlag) == 0  || force);
    boolean showView = show && !isResizing() && size > 0;

    ...

    if (view == null) {
        if (showView) {
        		// 关键3 设置view的颜色
            state.view = view = new View(mContext);
            view.setBackgroundColor(color);
            view.setTransitionName(state.transitionName);
            view.setId(state.id);
            visibilityChanged = true;
            view.setVisibility(INVISIBLE);
            state.targetVisibility = VISIBLE;

            LayoutParams lp = new LayoutParams(resolvedWidth, resolvedHeight,
                    resolvedGravity);
            if (seascape) {
                lp.leftMargin = sideMargin;
            } else {
                lp.rightMargin = sideMargin;
            }
            addView(view, lp);
            updateColorViewTranslations();
        }
    } else {
        int vis = showView ? VISIBLE : INVISIBLE;
        visibilityChanged = state.targetVisibility != vis;
        state.targetVisibility = vis;
        LayoutParams lp = (LayoutParams) view.getLayoutParams();
        int rightMargin = seascape ? 0 : sideMargin;
        int leftMargin = seascape ? sideMargin : 0;
        if (lp.height != resolvedHeight || lp.width != resolvedWidth
                || lp.gravity != resolvedGravity || lp.rightMargin != rightMargin
                || lp.leftMargin != leftMargin) {
            lp.height = resolvedHeight;
            lp.width = resolvedWidth;
            lp.gravity = resolvedGravity;
            lp.rightMargin = rightMargin;
            lp.leftMargin = leftMargin;
            view.setLayoutParams(lp);
        }
        if (showView) {
            view.setBackgroundColor(color);
        }
    }
    ...
}
```
先看下关键点1，这里是根据SystemUI的配置决定是否显示状态栏背景颜色。如果状态栏都不显示，那就没必要显示背景色了。再看关键点2，如果状态栏显示，但背景是透明色，也没必要添加背景颜色，即不满足(color & Color.BLACK) != 0。然后，看一下translucentFlag，默认情况下，状态栏背景色与translucent半透明效果互斥，半透明就统一用半透明颜色，不会再添加额外颜色。最后，再来看关键点3，其实很简单，就是往DecorView上添加一个View，理论上DecorView也是一个FrameLayout，所以最终的实现就是在FrameLayout添加一个有背景色的View。

### 四、100分状态栏着色实践
目前网上封装好的状态栏设置工具有不少，用得比较多的是[StatusBarUtil](https://github.com/laobie/StatusBarUtil)，还有已经废弃的[SystemBarTint](https://github.com/jgilfelt/SystemBarTint)。但是有个问题，无论哪个工具库，都不敢保证能够百分百实现状态栏着色的效果。因此，兼容性问题是一直存在的。通过实践分析总结，兼容性问题主要有两类：

+ 部分机子（大多数是4.4~5.0的机子）本身就是不支持状态栏着色效果
+ 5.0以上的机子，不支持常规操作方式，需要采用特殊方式处理（4.4～5.0的处理方式）




### 五、那些坑
+ 设置不成功回滚（oppo不成功的例子）
+ 尽量覆盖机型的兼容性测试
+ 为什么要设置systemuivisibility
+ windowTranslucentStatus与statusBarColor不能同时生效

### 六、总结
在Android项目开发过程中，免不了要和系统栏打交道。以上是作者根据平时项目开发经验、并结合网上查阅的资料对状态栏相关设置进行的总结。希望对大家有帮助，欢迎大家交流讨论～


### 参考文章
1. [Android App 沉浸式状态栏解决方案](http://jaeger.itscoder.com/android/2016/02/15/status-bar-demo.html)
2. [StatusBarUtil 状态栏工具类（实现沉浸式状态栏/变色状态栏）](http://jaeger.itscoder.com/android/2016/03/27/statusbar-util.html)
3. [令人困惑的fitsSystemWindows属性](http://www.jianshu.com/p/5cc3bd23be7b)
4. [what-are-windowinsets](https://stackoverflow.com/questions/33585225/what-are-windowinsets)
5. [Android沉浸式状态栏实现](http://ks.netease.com/blog?id=6650)
6. [沉浸式管理：让你的APP更优雅](https://mp.weixin.qq.com/s?__biz=MzIwMzYwMTk1NA==&mid=2247484516&idx=1&sn=1f31642399dd90bda5afa8664150a2d5&chksm=96cda529a1ba2c3fa47d0fb0e8328a28480c7496e551363d05306c56aadfca8df49baf109db4&mpshare=1&scene=23&srcid=0519RTiUoD1BCbAeAJaUajL5#rd)
7. [使用setSystemUiVisibility适配statusbar和navigationbar](http://www.jianshu.com/p/08ff70c15667)
8. [StatusBarAdapt](https://github.com/CoolThink/StatusBarAdapt)
9. [全屏、沉浸式、fitSystemWindow使用及原理分析：全方位控制“沉浸式”的实现](https://juejin.im/post/5948ff8fa0bb9f006bf5da29)
10. [Android SystemBar](http://ltlovezh.com/2016/05/31/Android-SystemBar/)
11. [探索Android半透明状态栏](http://chuansong.me/n/1818688651815)
12. [沉浸式状态栏](http://www.jianshu.com/p/d147608dc27b)
13. [由沉浸式状态栏引发的血案](http://www.jianshu.com/p/140be70b84cd?utm_source=tuicool&utm_medium=referral)
14. [Android开发：Translucent System Bar 的最佳实践](http://www.jianshu.com/p/0acc12c29c1b)