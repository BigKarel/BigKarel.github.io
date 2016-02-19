title: MVP模式
date: 2015-11-24 
tags: 
	- MVP
categories: 
	- MVP
toc: true
---
## MVP模式

* 项目按照MVP模式的设计思想，实现View和Model的解耦，弱化Activity/Fragment的功能。

* Presenter负责逻辑的处理，Model负责提供数据，View负责显示数据。View和Model完全分离，其交互在Presenter中进行。
* Presenter是通过View的接口与View进行交互（给View提供数据），保证在View改变时，Presenter不变。
* Presenter也是通过Model的接口与Model进行交互（获取提供的数据）。

<!--more-->

---

#### MVP 的使用流程

> MVP模式目前还没有特定的规范，凡是符合上述思想的均可称为MVP。Demo可参考项目中DemoActivity。

0. 定义BasePresenter（还可以细分为BaseListPresenter等），BaseView，BaseModel接口。这些接口为M/V/P各模块所共有的特性，比如BasePresenter中定义方法getData()(获取数据)，BaseView中定义showError等。

1. Presenter层：定义Presenter类。Presenter类主要完成业务逻辑。比如，当前Activity功能是列表展示数据，item的选择，删除，点击item跳转等，就可以为Presenter提供以上一些方法。
2. Model层：定义Model类。职责是获取数据，处理数据，提供数据。所有的数据任务都由这一层完成。
3. View层：定义View类。View类主要实现View的基本功能，如列表，功能是刷新数据，加载数据，操作数据等，其中的数据不用View关心，只需方法参数中传递数据给View提供就行。View得到数据后的具体操作还是交由Activity处理（比如这里用Adapter处理列表数据，其他情况也可以交由第三方处理，这样避免Presenter和View、Model的耦合性过高）。

---
### Demo中的示例如下：
1. Presenter：
	
	SearchResultListPresenterImpl 为 SearchResultListPresenter的实现类

	SearchResultListPresenterImpl(Context mContext, SearchResultListView resultListView)
	
	需要在Activity中获取Context，并实现SearchResultListView接口。
	
2. Model：
	
	SerchRresultListInteratorImpl 为 CommenListInterator的实现类，主要是获取数据。
	
	获取数据所需要的参数，通过Presenter提供。
	
3. View：
	
	SearchResultListView 只是个接口，由Activity调用回调方法。
	
	提供数据的展示，只有等数据获取到之后才能拿到数据，这一交互过程在Presenter中进行，由	Presenter控制。