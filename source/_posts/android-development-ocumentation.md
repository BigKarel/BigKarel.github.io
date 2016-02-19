title: Android客户端开发文档
date: 2015-11-25 
tags: [开发文档]
categories: [规范文档]
toc: true
---

> version：1.0
> 2015.11.12 
> author：zhongshan

## 开发环境规范
- 开发工具：Android Studio
- 编码：UTF-8
- 代码格式化：Code Style –>  Right marging:120，选中Wrap when typing reaches right margin ；java Wrapping and Braces Keep when reformatting选项中全部取消勾选。其他默认。
- 常用插件安装：Setting –>pluginsAndroid BufferKnife Zelezny ：BufferKnife依赖注入代码自动生成。Android Parcelable code generator : 自动生成实现Parceable接口的代码。
- JDK版本：jdk1.7

<!--more-->

## 项目规范：

### 项目结构：

	- 主工程（app）
	com.xxxx.  ——公司名+项目名（xxxx 公司项目同名）
				common. ——公共组件模块
						api. ——公共API接口相关
							XXXConstants.java ——XXX常量
						base. ——公共基类（Activity等）
						http. ——网络请求基础
						model. ——公有数据结构
						wedigets. ——自定义控件
						utils ——工具包
						presenters ——MVP 表现层公共接口
						view	—MVP 视图处理层公共接口
						interactor  ——MVP 数据处理层公共接口
				XXXXX.	——子模块名（n+）						api. ——模块API接口相关（请求、解析）						model. ——模块相关数据结构						activitys. ——模块相关页面						db. ——数据库
						presenters. ——MVP 控制处理层
							impl. ——实现类 
						view. ——MVP 视图处理层
						interactor. ——MVP 数据处理层
							impl. ——实现类 
							
	- module工程（library）
	  		com.android.volley ——Volley源码
	 		com.xxxx.library ——项目基础架构
- Activity/Fragment继承结构：	- 所有普通Activity继承BaseActivity	- 所有Fragment继承BaseFragment	- 所有需要边缘滑动退出的Activity继承BaseSwipeBackActivity
	---
### 命名规范：
> 遵循驼峰命名法 . 严禁使用拼音。命名要描述清楚，尽量不要简写。

1. 类（class）

		名词，采用大驼峰命名法，尽量避免缩写

	类|描述|例如
	---|---|---
	activity(Fragment) 类|Activity为后缀标识|知识详情页：KnowledgeDetialActivity
	Adapter类|Adapter 为后缀标识|搜索返回数据适配器：SearchResultAdapter
	解析类 	 |RespFactory为后缀标识	| 知识模块解析类KnowledgeRespFactory	公共方法类	 | Utils或Manager为后缀标识	| 线程池管理类：ThreadPoolManager 日志工具类：	StringUtils	数据库类	| 以DBHelper后缀标识	 |新闻数据库：NewDBHelper	Service类	 |以Service为后缀标识	 |时间服务TimeService	BroadcastReceive类	|  以Broadcast为后缀标识| 时间通知TimeBroadcast	ContentProvider  	| 以Provider为后缀标识	基础类	 |以Base开头	|BaseActivity,BaseFragment
	
2. 接口（interface）

		与类相同，大驼峰命名法
3. 方法（methods）

		动词或动名词，采用小驼峰命名法		
4. 变量（variables）采用小驼峰命名法。
   
	   	- 	类中控件名称必须与xml布局id保持一致			控件及其id命名：功能描述+控件单词首字母简写 
			eg：用户头像控件：userAvatarIv		- 	类中字段命名:
			(对象描述+集合名称)			List<MeetingTopicQuery> meetingTopicQueryList			String  username			Boolean   isChanged
			(首字母小写，尽量避免缩写)
			MeetingTopicQuery meetingTopicQuery
5. 常量（Constants

		全部大写，下划线命名法，如：IMAGE_LOADER_CACHE_PATH
6. 资源文件命名
	
		-  全部小写，采用下划线命名法，加前缀区分		-  命名模式：activity名称_逻辑名称/common_逻辑名称		-  如果有多种形态如按钮等除外如btn_xx.xml（selector）
	
7. 资源布局文件（XML文件/layout布局文件）

		全部小写，采用下划线命名法		-  contentview命名, Activity默认布局，以去掉后缀的Activity类进行命名。		   activity_功能模块.xml		   例如：activity_main.xml、activity_more.xml		-  Dialog命名：dialog_描述.xml
			例如：dialog _hint.xml		-  PopupWindow命名：ppw_描述.xml	       例如：ppw _info.xml		-  列表项命名listitem_描述.xml		   例如：listitem_city.xml   listitem_meeting.xml		-  包含项：include_模块.xml		   例如：include_head.xml、include_bottom.xml	---
### 注释规范
> 要求在逻辑比较复杂的地方写清必要的注释。常量及变量的命名基本都需要有注释。

1. 类（class）接口(interface)的注释

		在AndroidStudio中 按下面格式 修改File Header模板
		/**
		* Author：${USER}   
		* Date：${YEAR}/${MONTH}/${DAY} ${HOUR}:${MINUTE}      
		* Description：
		*/
		Description中写清楚对此类的描述。
2. 方法（method）的注释
	
		统一如下形式：
		/**
	    * 方法描述
	    * @param 参数1  参数1描述
	    * @param 参数2  参数2描述
	    */
	    
3. 变量及常量的注释
	
		统一如下形式：
		/**姓名*/
		String name;
		好处是在任何一个调用它的地方，使用F1查就能查看到变量/常量注释
4. 语句注释
		
		统一如下形式：
		// description

	---

### 通用组件使用说明
1. 常用类

		- 避免相同功能 的类或方法 重复添加。
		- 当需要添加某一个类时，先去对应的包下的help.txt中找，查看相同功能是否已存在。
		- utils类 统一放在common包下utils中。
		- wedigets类 应用的自定义控件均在此包下，每添加一个需要的控件时，都应该先去包下的help.text中查找，如果不存在才可添加。
		- 类似的还有上传下载，服务，广播等。

2. Fresco图片加载库
	
	> GitHub地址 [Fresco](https://github.com/facebook/fresco)
	
		- gradle中配置 compile 'com.facebook.fresco:fresco:0.8.1+'
		- 只在Application中初始化
		- 需要用到Fresco中View控件的自定义属性时需要在布局根目录添加
		  命名空间xmlns:fresco="http://schemas.android.com/apk/res-auto"
3. ButterKnife 依赖注解注入视图	
	> GitHub地址 [ButterKnife](https://github.com/JakeWharton/butterknife)
4. Material-Dialogs  Material向下兼容库
	> GitHub地址 [Material-Dialogs](https://github.com/afollestad/material-dialogs)
5. EventBus 事件总线
	> GitHub地址 [EventBus](https://github.com/greenrobot/EventBus)
	
		- 已经将EventBus集成到项目框架中
		- boolean isBindEventBusHere() ：是否使用EventBus
		- onEventComming(EventCenter eventCenter) : 订阅的数据
6. SystemBarTint 修改系统SmartBar颜色（以设置沉浸式状态栏）
	> GitHub地址 [SystemBarTint](https://github.com/jgilfelt/SystemBarTint)
	
		Activity中使用只需设置isApplyKitKatTranslucency() return true
7. Volley + OKHttp + gson 网络请求架构
	> 使用MVP的架构后，以下形式就废弃了
	
		相应模块的网络请求，写在模块对应包下api包中，如com.xxxx.home.api，		请求工具类命名为 模块名+RequestUtils.java，
		请求方法命名为do+请求功能+Request(Context context, final ResponseListener responseListener，Object params…)。		onStop中取消请求，调用.cancelAll(Tag)。	---

## MVP模式
### MVP 简介

* 按照MVP模式的设计思想，实现View和Model的解耦，弱化Activity/Fragment的功能。

* Presenter负责逻辑的处理，Model负责提供数据，View负责显示数据。View和Model完全分离，其交互在Presenter中进行。
* Presenter是通过View的接口与View进行交互（给View提供数据），保证在View改变时，Presenter不变。
* Presenter也是通过Model的接口与Model进行交互（获取提供的数据）。

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
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
