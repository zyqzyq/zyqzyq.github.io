---
layout: post
title: "kotlin 底部导航栏（tabLayout + fragment）"
date: 2017-09-02 16:25:06 -0700
comments: true
---
##准备工作
  在build.grale中添加
    ```compile "com.android.support:design:$support_version"```

#1.编辑main_activity.xml
  确定主页面布局FrameLayout +TabLayout
```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
        xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:tools="http://schemas.android.com/tools"
        xmlns:app="http://schemas.android.com/apk/res-auto"
        android:orientation="vertical"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        tools:context="com.zyqzyq.eyepetizer.activities.MainActivity">

    <FrameLayout
            android:id="@+id/main_container"
            android:layout_width="match_parent"
            android:layout_height="0dp"
            android:layout_weight="1">
    </FrameLayout>
    <View
            android:layout_width="match_parent"
            android:layout_height=".5dp"
            android:alpha=".6"
            android:background="@android:color/darker_gray"/>
    <android.support.design.widget.TabLayout
            android:id="@+id/bottom_tab_layout"
            android:layout_width="match_parent"
            android:layout_height="50dp"
            app:tabIndicatorHeight="0dp"
            app:tabSelectedTextColor="@android:color/black"
            app:tabTextColor="@android:color/darker_gray">
    </android.support.design.widget.TabLayout>
</LinearLayout>
```
#2.创建tabView.xml
  因为默认的tablayout添加icon显示后图片显得较小（选择了setCustomView方法，需要自定义view）
```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
              android:orientation="vertical"
              android:gravity="center"
              android:layout_width="match_parent"
              android:layout_height="match_parent">
    <ImageView
            android:layout_width="wrap_content"
            android:layout_height="0dp"
            android:id="@+id/tab_content_image"
            android:scaleType="fitCenter"
            android:layout_weight="1"/>
    <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:id="@+id/tab_content_text"
            android:textSize="@dimen/tabTextSize"
            android:textColor="@android:color/darker_gray"/>

</LinearLayout>
```
#3.编辑MainActivity.kt
  信息存在了object中,方便修改
 ```
object MainData {
    val mainFragmentList = arrayOf(HomeFragment(), FindFragment(), FollowFragment(), MineFragment())
    val mainTabRes = listOf(R.mipmap.ic_tab_home,R.mipmap.ic_tab_find,
            R.mipmap.ic_tab_follow,R.mipmap.ic_tab_mine)
    val mainTabResPressed = listOf(R.mipmap.ic_tab_home_selected,R.mipmap.ic_tab_find_selected,
            R.mipmap.ic_tab_follow_selected,R.mipmap.ic_tab_mine_selected)
    val mainTabStr = listOf("首页","发现","关注","我的")
}
```
默认添加了4个tab，并设置Tab 选中之后，改变各个Tab的状态
```
override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        initView()
    }
    private fun initView(){
        bottom_tab_layout.addOnTabSelectedListener(object : TabLayout.OnTabSelectedListener {
            override fun onTabSelected(tab: TabLayout.Tab) {
                onTabItemSelected(tab.position)
                // Tab 选中之后，改变各个Tab的状态
                for (i in 0 until bottom_tab_layout.tabCount ) {
                    val view = bottom_tab_layout.getTabAt(i)!!.customView

                    if (i == tab.position) { // 选中状态
                        view?.tab_content_image?.setImageResource(MainData.mainTabResPressed[i])
                        view?.tab_content_text?.setTextColor(resources.getColor(android.R.color.black))

                    } else {// 未选中状态
                        view?.tab_content_image?.setImageResource(MainData.mainTabRes[i])
                        view?.tab_content_text?.setTextColor(resources.getColor(android.R.color.darker_gray))
                    }
                }
            }

            override fun onTabUnselected(tab: TabLayout.Tab) {

            }

            override fun onTabReselected(tab: TabLayout.Tab) {

            }
        })

        for(i in 0..3) {
            bottom_tab_layout.addTab(bottom_tab_layout.newTab().setCustomView(getTabView(this, i)))
        }
    }

    private fun getTabView(context: Context,position: Int): View{
        val view = LayoutInflater.from(context).inflate(R.layout.tabview, null)
        view.tab_content_image.setImageResource(MainData.mainTabRes[position])
        view.tab_content_text.text = MainData.mainTabStr[position]
        return view
    }
```
#添加fragment
新建home_framgment.kt
代码如下
```
class HomeFragment: Fragment(){
    override fun onCreateView(inflater: LayoutInflater?, container: ViewGroup?, savedInstanceState: Bundle?): View {
        return inflater!!.inflate(R.layout.fragment_home,container,false)
    }

    override fun onActivityCreated(savedInstanceState: Bundle?) {
        super.onActivityCreated(savedInstanceState)
    }
}
```
新建fragment_home.xml
```<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
              android:orientation="vertical"
              android:layout_width="match_parent"
              android:layout_height="match_parent">

    <TextView
            android:text="home fragment"
            android:layout_width="match_parent"
            android:layout_height="wrap_content" android:id="@+id/textView"/>
</LinearLayout>
```
重复上述操作新建4个fragment。

#5.在MainActivity.kt添加fragment的更新
添加函数onTabItemSelected
```
fun onTabItemSelected(position: Int){
        val transaction = fragmentManager.beginTransaction()
        transaction.replace(R.id.main_container, MainData.mainFragmentList[position]).commit()
    }
```
并在onTabSelected中调用。

#6.最终效果图如下


![](http://upload-images.jianshu.io/upload_images/7646499-d081b53f35b04173.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)