title: 下拉刷新ListView的实现原理
date: 2015-11-25 10:38:48
tags:
---
> 下拉刷新源代码来自Trinea的 [DropDownListView.java][1]

通过对`ListView`添加了一个刷新`layout`(`drop_down_to_refresh_list_header.xml`)作为`header`，在滚动中时不断改变header的高度和内容并记录一些状态，在用户手指离开屏幕时根据状态决定进行刷新还是放弃刷新。
 
主要是通过重写`ListView`的`onTouchEvent`和`OnScrollListener`的`onScrollStateChanged`,`onScroll`函数实现
先介绍下刷新状态共有四种，如下：
`CLICK_TO_REFRESH` 点击刷新状态，为初始状态
`DROP_DOWN_TO_REFRESH` 当刷新`layout`高度低于一定范围时，为此状态
`RELEASE_TO_REFRESH` 当刷新`layout`高度高于一定范围时，为此状态
`REFRESHING` 刷新中时，为此状态
 
`onTouchEvent`函数
`onTouchEvent(MotionEvent event)`根据用户在屏幕上的move事件，进行相应操作，如下：
`ACTION_DOWN`表示用户手指刚接触屏幕，会记录用户此时touch的点的y坐标，在下面调整高度时使用
`ACTION_MOVE`表示用户手指正在屏幕上移动，此时会不断调整header的高度，即下拉时刷新item部分高度的不断变化
`ACTION_UP`表示用户手指离开屏幕，此时会根据当前状态决定是进行刷新还是放弃刷新，若刷新调用用户设置的`OnRefreshListener`接口。
 
`onScrollStateChanged`函数
`onScrollStateChanged(AbsListView view, int scrollState)`
记录`listView`当前的滚动状态到`currentScrollState`，包括三种状态：
`SCROLL_STATE_TOUCH_SCROLL` ListView正在滚动中，并且手指尚未离开屏幕
`SCROLL_STATE_FLING` ListView仍在滚动中，但用户手指已经离开屏幕
`SCROLL_STATE_IDLE` ListView已经停止滚动
 
`onScroll`函数
`onScroll(AbsListView view, int firstVisibleItem, int visibleItemCount, int totalItemCount)`
根据listView当前的滚动状态即`currentScrollState`和当前刷新的状态不断修改`header`内容显示和刷新状态，如下：
    `ListView`为`SCROLL_STATE_TOUCH_SCROLL`状态(按着不放滚动中)并且刷新状态不为`REFRESHING`
        a. 刷新对应的`item`可见时，若刷新`layout`高度超出范围，则置刷新状态为`RELEASE_TO_REFRESH`；若刷新`layout`高度低于高度范围，则置刷新状态为`DROP_DOWN_TO_REFRESH`。
        b. 刷新对应的`item`不可见，重置`header`
 
`ListView`为`SCROLL_STATE_FLING`状态(松手滚动中)
a. 若刷新对应的item可见并且刷新状态不为`REFRESHING`，设置`position`为1的(即第二个)`item`可见
b. 若反弹回来，设置`position`为1的(即第二个)`item`可见


  [1]: https://github.com/Trinea/AndroidCommon/blob/master/src/cn/trinea/android/common/view/DropDownListView.java