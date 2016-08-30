---
layout: post
title: Android 新特性
author: YangLing
---

Google在2015的IO大会上，发布了Android5.0及Material Design设计规范，同时，带来了全新的Android Design Support Library（向下兼容到Android 2.2）。安卓5.0是有史以来最重要的安卓版本之一，这其中有很大部分要归功于Material Design的引入，这种新的设计语言让整个安卓的用户体验焕然一新。

## Android 新特性(5.0/6.0)
-----
 

====================================
* MD-CoordinatorLayout (协调者布局)
* MD-FloatingActionButton (悬浮操作按钮)
* MD-Toolbar 工具栏(替代之前的ActionBar)
* MD-AppBarLayout (应用标题栏容器)
* MD-CollapsingToolbarLayout 折叠效果的布局容器
* MD-CoordinatorLayout Behaviors(变化反应)的使用

* MD-TabLayout 标签导航

* MD-TextInputLayout 输入框控件的悬浮标签
* MD-NavigationView 抽屉导航
* RecyclerView 代替ListView/GridView

* 运行时权限(6.0)

------------------------------------
## 概述
* Google在2015的IO大会上，发布了Android5.0及Material Design设计规范，同时，带来了全新的Android Design Support Library（向下兼容到Android 2.2）。安卓5.0是有史以来最重要的安卓版本之一，这其中有很大部分要归功于Material Design的引入，这种新的设计语言让整个安卓的用户体验焕然一新。

* 配置方式:  
在build.gradle的dependencies节点下添加
	
	> compile 'com.android.support:design:23.1.1'  
	> 或  
	> compile 'com.android.support:design:22.2.0'

* AppCompatActivity
	> 界面不再继承 Activity, FragmentActivity 或ActionBarActivity, 而是继承AppCompatActivity, 目的是为了将MD的风格,及Toolbar等新的特效兼容到低版本

* Theme.AppCompat
	> Activity的主题配置为Theme.AppCompat, 在主题中可以配置许多系统自带属性值. 目的同样是为了将新的风格兼容到API 7

## MD-CoordinatorLayout
* CoordinatorLayout (协调者布局) 实现了多种Material Design中提到的滚动效果.
把CoordinatorLayout作为根布局容器，其子控件可以不用写动画相关的代码就能产生动画

* MD提供的主要子控件有:
	- FloatingActionButton 浮动操作按钮
	- AppBarLayout 应用标题栏容器 
	- NestedScrollView 类似于ScrollView, 配合AppBarLayout执行动画使用
	- Toolbar 工具栏(替代之前的ActionBar)
	- Snackbar 反馈提示栏
	- RecyclerView 自带回收的ListView

* `android:fitsSystemWindows="true"` 适应系统, 是否把内容显示到状态栏	

* 示意图如下:  
> ![CoordinatorLayout](img/CoordinatorLayout.png)


## MD-FloatingActionButton 悬浮操作按钮
* 在CoordinatorLayout的子控件中, 通过`app:layout_anchor`(布局锚) 和 `app:layout_anchorGravity`属性创造出悬浮效果
	- `app:layout_anchor`: 把指定的`锚点`控件作为当前Button的目标视图。
	- `app:layout_anchorGravity`: 在目标视图中的位置。值由bottom、center、right、left、top等
> ![fab_snackbar](img/fab_snackbar.gif)

* 其他常用属性:

		app:backgroundTint - 设置FAB的背景颜色。
		app:rippleColor - 设置FAB点击时的背景颜色。
		app:borderWidth - FAB边缘宽度, 在有的4.1的sdk上FAB会显示为正方形，或在5.0以后的sdk没有阴影效果。可以设置为borderWidth="0dp"解决此问题。
		app:elevation - 默认状态下FAB的阴影大小。
		app:pressedTranslationZ - 点击时候FAB的阴影大小。
		app:fabSize - 设置FAB的大小，该属性有两个值，分别为normal和mini，对应的FAB大小分别为56dp和40dp。
		src - 设置FAB的图标，Google建议符合Design设计的该图标大小为24dp。

## MD-ToolBar 工具栏
* Toolbar是应用的内容的标准工具栏
* Toolbar可以根据需要配置在布局中, 用来替代以往的ActionBar
* 使用Toolbar必须在Activity配置的theme中去除ActionBar, 如使用`Theme.AppCompat.Light.NoActionBar`或在主题style中添加如下配置:
	
        <item name="windowActionBar">false</item>
        <item name="windowNoTitle">true</item>
* 需要在代码中配置, 把Toolbar作为ActionBar

        Toolbar toolbar = (Toolbar) findViewById(R.id.toolbar);
        setSupportActionBar(toolbar);
* 配置Toolbar内容:

        // 设置标题
        getSupportActionBar().setTitle("Title");
        // 显示返回按钮
        getSupportActionBar().setDisplayHomeAsUpEnabled(true);
		...

* 一个Toolbar如下:
            
             <android.support.v7.widget.Toolbar
                android:id="@+id/toolbar"
                android:layout_width="match_parent"
                android:layout_height="?attr/actionBarSize"
                app:popupTheme="@style/ThemeOverlay.AppCompat.Light" />

> ![toolbar](img/toolbar.png)

## MD-AppBarLayout (应用标题栏容器)
1. AppBarLayout只能作为CoordinatorLayout里的第一个子view。
2. 在RecyclerView或者任意支持`嵌套滚动`的view如NestedScrollView上添加`app:layout_behavior`属性
3. `app:layout_behavior`
	> 属性值配置为`@string/appbar_scrolling_view_behavior`,该值在com.android.support的design包下, 用来通知AppBarLayout, 界面内容发生了滚动事件.
4. 在AppBarLayout中的子View中配置`app:layout_scrollFlags`属性，就可以在滚动事件发生的时候自动执行自己动画.

5. `app:layout_scrollFlags`

	> 属性里面必须至少添加`scroll`这个flag，这样这个View及其子控件才会滚动出屏幕，否则它最终将一直固定在顶部。

* `app:layout_scrollFlags`的几个属性值及其意义
	
	- `scroll` 
		> 列表在顶端时,可以跟着滑动方向滑动(只有配置了scroll, 其他属性才能生效)
		
	- `enterAlways` 
		> 不管滑动列表到哪里, 只要往下拉, 当前控件就跟着往下滑出. 
		
	- `enterAlwaysCollapsed` 
		> 跟enterAlways类似, 但是最大只会滑出折叠后的大小.要配合`android:minHeight="50dp"`属性使用.最小高度即为折叠后的高度 (配置方式:`"scroll|enterAlways|enterAlwaysCollapsed"`)
		
	- `exitUntilCollapsed` 
		> 不管滑动列表到哪里, 只要向上拉，这个View会跟着滑动直到折叠。配合`android:minHeight="50dp"`属性使用.最小高度即为折叠后的高度(配置方式:`"scroll|exitUntilCollapsed"`)
		
	- `snap` 
		> 滑动结束的时候，如果这个View部分显示，它就会滑动到离它最近的上边缘或下边缘。

* 布局结构如下:
	
		<CoordinatorLayout>
		    <AppbarLayout>
				<ImageView/>
				<Toolbar/>
			</AppbarLayout>
		    <NestedScrollView/>
		</CoordinatorLayout>

## CollapsingToolbarLayout 折叠效果的布局容器
* 如果需要Toolbar有折叠效果, 可以让CollapsingToolbarLayout包裹Toolbar
* 相当于让CollapsingToolbarLayout临时托管了Toolbar的标题内容, 执行平移和缩放动画
* Toolbar可以随着CollapsingToolbarLayout向上滑动
* 给Toolbar添加`app:layout_collapseMode`属性可以配置向上滑动时的效果
	
		app:layout_collapseMode="none" 默认效果,随父控件一起滑动
		app:layout_collapseMode="pin"  图钉效果,开始滑动时自己不动, 到顶端才离开
		app:layout_collapseMode="parallax" 视察效果,随父控件进行高度缩放

* 布局结构如下:
	
		<CoordinatorLayout>
		    <AppbarLayout>
				<CollapsingToolbarLayout>
					<ImageView/>
					<Toolbar/>
				</CollapsingToolbarLayout>
				...
			</AppbarLayout>
		    <NestedScrollView/>
		</CoordinatorLayout>

## MD-TabLayout 标签导航
* Tab 是在 Android 应用程序中用户体验(UX)最佳实践的一部分。以前，如果需要使用新的材料设计风格的 Tab，需要自己去Github下载 SlidingTabLayout 和 SlidingTabStrip.

* 布局配置

        <android.support.design.widget.TabLayout
            android:id="@+id/tabs"
            android:layout_width="match_parent"
            android:layout_height="wrap_content" />

* 代码配置:

        TabLayout tabLayout = (TabLayout) findViewById(R.id.tabs);
		// 关联ViewPager
        tabLayout.setupWithViewPager(mViewPager);

> ![TabLayout](img/TabLayout.png)

## MD-TextInputLayout 输入框控件的悬浮标签
* 使用场景: 用来显示一个提示或一个错误信息, 默认会把Hint放置在上方
* 使用方式: 用 TextInputLayout 包裹 EditText 即可, 如下

		<android.support.design.widget.TextInputLayout
		    android:layout_width="match_parent"
		    android:layout_height="wrap_content">
		    <EditText
		        android:layout_width="match_parent"
		        android:layout_height="wrap_content"
		        android:hint="Username" />
		</android.support.design.widget.TextInputLayout>

> ![TextInputLayout](img/TextInputLayout.png)

## MD-NavigationView 抽屉导航
* NavigationView是android-support-design包下的一个控件, 用来配合`android.support.v4.widget.DrawerLayout`便捷地配置抽屉内容

		<android.support.v4.widget.DrawerLayout
		    xmlns:app="http://schemas.android.com/apk/res-auto">
		    <include layout="@layout/activity_main"/>

		    <android.support.design.widget.NavigationView>
		    ...
		    </android.support.design.widget.NavigationView>

		</android.support.v4.widget.DrawerLayout>
* 重要的属性值

	
        android:layout_gravity="start"  抽屉在左侧滑出
        android:fitsSystemWindows="true"  适配系统窗体
        app:headerLayout="@layout/nav_header_main_navi" 设置头布局文件
        app:menu="@menu/activity_main_navi_drawer"	设置下边菜单文件

* 给导航条目添加监听:
		
		navigationView.setNavigationItemSelectedListener(new OnNavigationItemSelectedListener(){...})

> ![NavigationView](img/NavigationView.png)


## RecyclerView
* RecyclerView 是Android L (5.0)版本中新添加的一个用来取代ListView/GridView的SDK，它的灵活性与可替代性比listview更好。

* 核心类
	- Adapter：使用RecyclerView之前，你需要一个继承自RecyclerView.Adapter的适配器，作用是将数据与每一个item的界面进行绑定。
	- LayoutManager：用来确定每一个item如何进行排列摆放，何时展示和隐藏。目前SDK中提供了三种自带的LayoutManager:

			LinearLayoutManager
			GridLayoutManager
			StaggeredGridLayoutManager

* 引用库会自动添加到工程, 如果sdk版本较低,没有自动添加,可在build.gradle中手动添加

			compile 'com.android.support:recyclerview-v7:21.0.+'

> ![RecyclerView](img/RecyclerView.png)

## 运行时权限
* 核心方法 (需`Android 6.0 (API level 23)`及以上)
	- 检查是否已获得权限
		> `context.checkPermission(permission, android.os.Process.myPid(), Process.myUid())`
	- 检查是否需要弹出本地提示			
		> `activity.shouldShowRequestPermissionRational	e(permission);`  
		> `fragment.shouldShowRequestPermissionRationale(perm);`  
	- 请求指定权限
		> `activity.requestPermissions(permissions, requestCode);`  
		> `fragment.requestPermissions(perms, requestCode);`
* 权限申请官方文档
	> https://developer.android.com/training/permissions/index.html  

* 开源项目
	> https://github.com/googlesamples/easypermissions  
	  https://github.com/Karumi/Dexter  
	  https://github.com/hotchemi/PermissionsDispatcher