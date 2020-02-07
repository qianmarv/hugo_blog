+++
title = "UI5入门系列之二： 最佳实践练习（下）"
date = 2015-08-25
tags = ["SAPUI5"]
categories = ["编程"]
draft = false
+++

上期我们完成了一个简单的主从页面，但是页面是静态的，不能交互，功能也很简单，只有一个销售订单的列表。
我们今天就一鼓作气把代码全都写完，由于本次的代码量较大，所以只对重点代码部分进行讲解。
具体每个文件和代码就不一一贴出来了，代码都放在github中，需要的自行[下载](https://github.com/qianmarv/ui5tutorial/tree/master/ui5bp%5F2)吧。
<!--more-->


## 页面导航 {#页面导航}

可以先把代码下载到本地并跑起来，这样可以对这个最佳实践的程序有一个直观的了解。

页面导航如下： <br />
销售订单列表(Master) -> 销售订单明细(Detail) -> 行项目明细(LineItem),在每个明细页面都可以返回到上一层。

具体页面之间的导航是如何实现的呢？ <br />
我们从页面的入口 `index.html` 开始

```javascript
         var oView = sap.ui.view({
             id : "app",
             viewName : "ui5.tutorial.bp.view.App",
             type : "JS",
         });
         //...
         oView.placeAt('content');
```

这一段代码初始化了一个叫做App的JS view，那我们就来看 `App.view.js`

```javascript
	// create app
	this.app = new sap.m.SplitApp();

	// load the master page
	var master = sap.ui.xmlview("Master", "ui5.tutorial.bp.view.Master");
	master.getController().nav = this.getController();
	this.app.addPage(master, true);

	// load the empty page
	var empty = sap.ui.xmlview("Empty", "ui5.tutorial.bp.view.Empty");
	this.app.addPage(empty, false);
```

App view中在 `createContent` 中创建了一个SplitApp，稍微说下SplitApp这个控件，当宿主是PC或者平台的时候这个控件默认包含两个页面容器，而当宿主是手机的时候又可以只包含一个页面，所以一般主从结构的页面可以用这个控件。<br />
随后，分别创建了两个页面，一个是Master页面，另一个页面是空白页——作为没有选中任何Master页面中的数据时的默认页面，最后把两个页面都加入到了SplitApp中。<br />
到目前为止都是在上一篇中已经做过的，到这里，页面已经可以展示了。但是我们今天要研究的是导航，所以接着往下。<br />
SplitApp本身带有 `to()` 这个函数，可以在已经加入其容器的页面之间导航，但是我们现在的功能稍稍有点复杂，当点击Master页面的时候，要求导航到详细页面，
点击详细页面的行项目可以进入到行项目的详细页面，对应每个详细页面可以返回至上一级页面，同时可能还需要做一些其他逻辑上的处理，比如判断和绑定数据，因此我们需要在原有的 `to()` 上增强一些功能并封装给其他的控制器使用。<br />

这些功能都集中定义在 `App.controller.js` 这个控制器中<br />

```javascript
    to : function (pageId, context) {

	var app = this.getView().app;

	// load page on demand
	var master = ("Master" === pageId);
	if (app.getPage(pageId, master) === null) {
	    var page = sap.ui.view({
		id : pageId,
		viewName : "ui5.tutorial.bp.view." + pageId,
		type : "XML"
	    });
	    page.getController().nav = this;
	    app.addPage(page, master);
	    jQuery.sap.log.info("app controller > loaded page: " + pageId);
	}

	// show the page
	app.to(pageId);

	// set data context on the page
	if (context) {
	    var page = app.getPage(pageId);
	    page.setBindingContext(context);
	}
    },

    /**
     * Navigates back to a previous page
     * @param {string} pageId The id of the next page
     */
    back : function (pageId) {
	this.getView().app.backToPage(pageId);
    }
```

重新封装后的 `to()` 有两个参数，一个是页面id，另一个是上下文，当传入的页面不存在的时候，系统会首先初始化这个页面并加入到当前的SplitApp中，随后直接调用SplitApp的 `to()` 完成导航动作。
最后，如果上下文不为空的话，把这个上下文绑定到新的页面中，这对于一个新页面来说是非常有意义的。<br />
back则直接复用SplitApp的 `backToPage()` 函数。

接下来我们再看其他的页面如何复用定义在这里的导航函数的。<br />
我们回过头来再看 `App.view.js` , 其中在初始化Master页面之后，有这么一条语句 <br />
`master.getController().nav = this.getController();` <br />
这条语句把当前的控制器赋给Master页面的控制器的nav，这个nav只是用来存放App控制器的一个key，叫什么名字都行，这样在Master页面中就可以通过nav来调用App控制器的所有函数了。

同样的，再回到 `App.controller.js` 中，看到这条语句 `page.getController().nav = this;` 也是类似的作用。

在首页面刚刚初始化时，Detail页面是没有加载的，当点击Master页面中的某个SalesOrder的时候，Master页面中的 `handleListItemPress` 被调用：

```javascript
    handleListItemPress : function (evt) {
	var context = evt.getSource().getBindingContext();
	this.nav.to("Detail", context);
    }
```

首先获得被点击的item上绑定的上下文信息（数据），然后调用App的 `to()` 方法并传入 `Detail` 告诉页面要加载Detail这个页面，同时把上下文数据传递过去，
接下来又回到了 `to()` ， Detail页面被初始化，绑定上下文数据，加载到SplitApp中，然后导航成为当前显示页面。

以上，就完成了这个App的导航，可以看到有些繁琐，并且需要显式的初始化子页面的时候获得并共享App的控制器，另外，页面之间的跳转不会修改URL，无法将某个中间页面存为bookmark，所以这种方式并不是UI5所推荐的，
但是作为初学者了解UI5的页面导航机制还是非常的直观，另外对于简单的应用来说，如果页面较少也未尝不可以考虑。<br />
作为稍大型的web应用，UI5在早期的版本中推荐使用EventBus通过Event的传递来实现复杂的页面导航，从1.6开始引入了新的导航机制，就是Routing，可以将页面之间的导航关系定义在component中，在最新的1.30版本中，导航定义则可以直接写在App的说明文件 `manifest.jso` 中。<br />
导航就介绍到这里，Component和Routing是一个比较复杂但是非常强大的工具，我们可以在后续接着探讨。


## 数据绑定 {#数据绑定}

在我们的代码中，数据绑定也是做了简化处理，都直接写在 `index.html` 中了。

一共绑定了三个模型： <br />

-   业务数据模型：<br />
    因为我们使用的是离线的json格式数据，所以可以直接把相对路径传递给 `sap.ui.model.json.JSONModel` 来初始化这个模型，并绑定到App这个根视图上。

    ```javascript
               var oModel = new sap.ui.model.json.JSONModel("model/mock.json");
               oView.setModel(oModel);
    ```

    随后在这个视图及其子视图中，都可以直接通过类似 `{SoId}` 这种语法格式来使用这个模型的数据字段，需要注意的是，如果需要绑定的字段是这个模型的根节点，需要在前面加一个 `/` ，譬如在Master视图中绑定到列表的aggregate字段 `items` ，是这样的语法格式：items="{/SalesOrderCollection}" 。

-   多语言模型：<br />
    UI5中使用了 `i18n` 机制来处理多语言问题。i18n是 internationalization的简称，在首位两个字母之间有18个字母……

    具体如何使用非常的简单，<br />
    首先创建一个资源文件 `messageBundle.properties` ，这里我们在根目录创建了一个 `i18n` 目录，在这里目录中集中存放相关的i18n文件。<br />
    在这个资源文件里我们定义如下：

    ```text
          MasterTitle=Sales Orders \\
          DetailTitle=Sales Order \\
          StatusTextN=New \\
          StatusTextP=In Process \\
          ApproveButtonText=Approve \\
          ...
    ```

    左边的是KEY，右边的是对应的语言描述，如果我们需要定义一个中文的语言文件，那么只需要拷贝这个文件并重命名为 `messageBundle_zh-CN.properties` ，并将对应的描述改为中文如下：<br />

    ```text
          MasterTitle=销售订单列表
          DetailTitle=销售订单
          StatusTextN=新建
          StatusTextP=处理中
          ApproveButtonText=批准
          ...
    ```

    系统会根据用户设定的浏览器语言顺序依次查找对应的语言资源文件，如果都找不到的话，就会找默认的 `messageBundle.properties` 。

    定义好了资源文件，我们接下来就在 `index.html` 中通过 `sap.ui.model.resource.ResourceModel` 来初始化这个资源模型，接着就可以把它绑定到视图或者控件中使用了。<br />
    怎么使用呢？在视图中通过类似 `{i18n>MasterTiel}` 这种语法格式来绑定到对应的空间的文本项上，实际使用中需要用引号把这个串包含进去。这个串中前面的 `i18n>` 指的是引用绑定到本视图的叫做i18n的模型，这里的i18n是绑定时起的名字， `oView.setModel(i18nModel, "i18n");`  可以是任意符合格式的字符串。<br />

-   设备模型
    设备模型通过查询jQuery的device来获悉宿主是否手机，并设定相应的不同显示选项，然后将结果存为Json格式并初始化为一个JSON模型，最后绑定到模型中。


## 工具方法 {#工具方法}

  大多数情况下，我们可以直接把业务数据直接绑定到控件中显示，但是在一些情况下，我们可能需要对其中的一些格式做一些调整，或者根据一些字段做一些简单的逻辑处理，
这个时候，我们就需要用到大多数控件中的某些属性的 `formatter` 方法。<br />

```XML
	  <ObjectStatus
	      text="{
		    path: 'LifecycleStatus',
		    formatter: 'ui5.tutorial.bp.util.Formatter.statusText'
		    }"
	      state="{
		     path: 'LifecycleStatus',
		     formatter: 'ui5.tutorial.bp.util.Formatter.statusState'
		     }" />
```

  上面这个例子中，我们来看 `text` 属性，如果我们希望直接把业务数据绑定到text中，我们这样定义 text="{LifecycleStatus}"，但是我们知道这个字段可能是后台定义的技术字段，我们需要把它转化的比较有业务意义。
所以这个时候，我们就需要用到 `formatter` 了，首先定义 `path` 告之需要绑定的字段，这里不需要用大括号，随后给 `formatter` 赋予一个处理方法，这个方法可以定义在任何地方，我们这里是在util下单独定义了一个 `Fomatter.js` 来集中处理这类需求。<br />
  来看 `Formatter.js` ，我们就看 `statusText` 这个方法：

```javascript
	statusText :  function (value) {
		var bundle = this.getModel("i18n").getResourceBundle();
		return bundle.getText("StatusText" + value, "?");
	},
```

`path` 中绑定的字段对应的值会作为参数传入，然后用这个值结合StatusText生成一个KEY，并在 `i18n` 中取出相应的描述。


## 总结 {#总结}

基本上这个最佳实践应用已经被剖析完成了，通过这样一个最佳实践 `Best Practice` 的练习，我们学习到了一般的UI5应用的整体结构以及大多数重要控件的使用方法。
