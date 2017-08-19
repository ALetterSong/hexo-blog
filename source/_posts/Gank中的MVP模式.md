---
title: Gank中的MVP模式
date: 2016-10-01 10:59:27
tags:
---


### 前言

第一次看到 Gank，还是源于 drakeet 的 [Meizhi](https://github.com/drakeet/Meizhi) 项目，后来各种干货项目层出不穷，自己的项目中也借鉴了其中不少的写法，恰好最近公司决定让我做点前端了，同时项目经理也和我一样偏爱 Material Design，在说服了老板后，我就开始边学边用 [Materialize](http://materializecss.com/) 框架重写公司的学校管理系统页面。
  
最近一段时间， Android 的任务也没那么多，终于可以有时间写自己的Gank客户端了，~~看起妹纸来也就更方便了~~。Gank项目采用 MVP 架构，用到了 RxJava，Retrofit 等最近流行的技术，同时，我最近也开了一个新的分支，将依赖注入框架 [Dagger](https://google.github.io/dagger/) 加入其中。我就先写篇文章记录下 Gank 项目中 MVP 模式的使用，下次会介绍 Gank 中 Dagger 的使用，并想在以后加入 Data Binding 技术。
  
### 什么是MVP模式
  
  说到 MVP，还是得先谈谈 MVC，MVC 模式将把软件系统分为三个基本部分：Model、View 和 Controller。原生 Android 开发应该就算是一个基础的 MVC 框架了，我们一般采用 XML 进行界面的描述，在 Activity 中读取 View 的数据，并通知 Model 更新，针对业务模型，建立数据结构相关的类，处理数据库或者网络操作。
  
  然而，XML 作为 View 层的控制能力实在是太弱了，如果你想去动态的改变一个页面的背景或者动态的隐藏显示一个按钮，这些都没办法在 XML 中做，只能把代码写在Activity中，造成了 Activity 既是 Controller，又是View的局面。随着项目的扩大，逻辑越来越复杂，导致代码会越来越难读，难以维护与测试。
  
框架模式的最终目的都是为了增加代码的可读性，维护性和方便测试，我们无论采取什么框架，都不应该背离软件设计的这些基本原则。MVP 模式应运而生

MVP 是 Model-View-Presenter 的缩写，它由三部分组成：Model 提供数据，Presenter 负责逻辑处理，View 负责显示。在 MVP 模式中，View 与 Model 不直接进行交互，而是通过 Presenter 来进行间接交互。通常来说，一个 Presenter    对应与一个 View，Presenter 与 View 的交互通过接口来实现，这也更加有利于单元测试。

![MVP](http://7xq3d5.com1.z0.glb.clouddn.com/MVC_MVP.png)

### Gank中的MVP模式

Gank 主页面主要用来展示妹纸图片，同时可以进行下拉刷新和底部加载更多，我把所有业务逻辑都写在 Presenter 中，代码如下

    public void loadPage() {
        mView.showRefreshView();
        mGank.getGirls(PAGE_SIZE, mCurrentPage)
                .subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())
                .map(new Func1<GirlData, List<GankEntity>>() {
                    @Override
                    public List<GankEntity> call(GirlData girlData) {
                        return girlData.results;
                    }
                })
                .subscribe(new Subscriber<List<GankEntity>>() {
                    @Override
                    public void onCompleted() {
                    }

                    @Override
                    public void onError(Throwable e) {
                        e.printStackTrace();
                        mView.showErrorView();
                        LLog.e(e.getMessage());
                    }

                    @Override
                    public void onNext(List<GankEntity> girlEntities) {
                        mView.hideRefreshView();

                        if (girlEntities.isEmpty()) {
                            //没有数据
                            mView.showEmptyView();
                        } else if (girlEntities.size() < PAGE_SIZE) {
                            //最后一页
                            mView.updateData(girlEntities, false);
                            mView.hasNoMoreData();
                        } else if (girlEntities.size() == PAGE_SIZE) {
                            //正常情况
                            mView.updateData(girlEntities, false);
                        }
                    }
                });
    }

    public void loadNextPage() {
        mCurrentPage++;
        loadPage();
    }

    public void refreshPage() {
        mCurrentPage = 1;
        mView.clearData();
        loadPage();
    }


可以看到，我通过`loadPage()`方法加载数据，使用了 RxJava + Retrofit 进行数据请求，在`onNext()`方法中就可以获取到妹纸数据,并通过`mView.updateData()`方法通知 View 更新界面。

那么，`mView`是什么呢?我们发现，它是`IGirlListView`接口的一个实例，代码如下

	public interface IGirlListView<T> extends ISwipeRefreshView {
	
	    void updateData(T data, boolean needClear);
	
	    void clearData();
	
	    void hasNoMoreData();
	}

我们发现其中有我们所调用的`updateData()`方法，同时`GirlListActivity`实现了`IGirlListView`这个接口，我们再来看看`GirlListActivity`是如何实现`updateData()`这个方法的

	  @Override
	    public void updateData(List<GankEntity> data, boolean needClear) {
	        if (needClear) {
	            mAdapter.updateWithClear(data);
	        } else {
	            mAdapter.update(data);
	        }
	    }

通过此方法，我们更新了 Adapter，Activity 再也没有那么多臃肿的代码了。
那么 Presenter 又是如何创建的呢？

通过`BaseActivity`中定义的抽象方法，我们在`GirlListActivity`中实现了此方法，并在构造函数中传入`Context`和`IBaseView`接口的实例，

	@Override
	    protected void initPresenter() {
	        mPresenter = new GirlListPresenter(this, this);
	    }

通过 Presenter，我们就可以方便的调用`loadPage()`, `refreshPage()`等方法了。


参考资料:

[Android官方MVP架构示例项目解析](http://mp.weixin.qq.com/s?__biz=MzA4MjA0MTc4NQ==&mid=404088059&idx=3&sn=78dafacbca09b0d7345344c3eef24aff#rd)  
[不要再给MVP中Presenter写接口了](http://blog.chengdazhi.com/index.php/205)  
[选择恐惧症的福音！教你认清MVC，MVP和MVVM](http://zjutkz.net/2016/04/13/%E9%80%89%E6%8B%A9%E6%81%90%E6%83%A7%E7%97%87%E7%9A%84%E7%A6%8F%E9%9F%B3%EF%BC%81%E6%95%99%E4%BD%A0%E8%AE%A4%E6%B8%85MVC%EF%BC%8CMVP%E5%92%8CMVVM/)  
[Android App的设计架构：MVC,MVP,MVVM与架构经验谈](https://www.tianmaying.com/tutorial/AndroidMVC)  
[一步一步实现Android的MVP框架](http://mp.weixin.qq.com/s?__biz=MzA3NTYzODYzMg==&mid=2653577546&idx=1&sn=e10be159645a3aa8f6d6f209420fb412&scene=0#wechat_redirect)  
[MVP 模式在 GankDaily 中的应用](http://gudong.name/advanced/2015/11/23/gank_mvp_introduce.html)  
[android MVP模式介绍与实战](http://dahei.me/2016/06/22/mvp/android%20MVP%E6%A8%A1%E5%BC%8F%E4%BB%8B%E7%BB%8D%E4%B8%8E%E5%AE%9E%E6%88%98/)  









 
   