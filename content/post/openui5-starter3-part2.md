+++
title = "UI5入门系列之三：MVC （下） - 视图与控制器"
date = 2015-09-15
tags = ["SAPUI5"]
categories = ["编程"]
draft = false
+++

继续来学习UI5的MVC模型吧，这次我们来探讨视图与控制器。
<!--more-->


## 视图 {#视图}

在MVC中，视图用来定义和渲染UI。在UI5中，视图的类型是可以自定义的，除了以下预定义的四种视图类型之外，你也可以定制自己的视图类型。
预定的四种视图类型如下：<br />

-   XML view
-   JSON view
-   JS view
-   HTML view

如果你想定义自己的视图类型，可以通过扩展 `sap.ui.core.mvc.View` 这个基类来实现。


### 视图的加载 {#视图的加载}

   视图可以通过异步或者同步的方式加载，默认是同步的方式。视图的工厂函数通过同步的方式请求并传入视图定义的源代码并返回一个视图的实例。但是这种方式会导致在加载视图的时候UI界面卡住，而且也有可能会导致视图在初始化期间一些函数不能够被正常调用。
所以，为了避免这种情况，可以通过异步（asynchronous）的方式来加载视图，所有的视图类型都支持异步的方式。

以下是一个同步加载视图，视图实例被创建之后放到id为uiArea的DOM元素中，随后视图会被渲染。

```javascript
var oView = sap.ui.xmlview({ viewName: “sample.view” });
// the instance is available now
oView.placeAt(“uiArea”);
```

以下的代码片段是一个异步模式加载视图的例子，请求视图工厂函数创建视图实例，但是此时由于这个请求是一个异步的模式，视图实例还没有ready，所以我们不能马上用 `placeAt` 来把它放到DOM元素中，我们必须等待View.prototype.loaded()执行完毕之后发出的Promise之后，在 `then()` 中才能真正操作已经实例化完毕的视图。
如果对于 `jQuery` 的递延、回调等等异步概念不太了解的，可以阅读这篇文章[^fn:1]，如果英文不错的可以直接看jQuery的官方API[^fn:2]。


### 视图的实例化 {#视图的实例化}

其实前面也已经提到了，UI5通过 `sap.ui.view` 这个工厂函数来实例化视图。<br />
这个工厂函数可以接受如下参数：<br />

-   `type` : <br />
    类型可以是预定义的 `JSON`, `JS`, `XML`, 或者 `HTML` ， 这些类型都被定义成了枚举类型，所以在引用的时候可以直接用 `sap.ui.core.mvc.ViewType` 下面的这四个类型的大写字符串就可以，比如 `sap.ui.core.mvc.ViewType.XML` 就是代表XML视图，当然直接用字符串 "XML" 也是可以的，规范化起见最好还是使用枚举类型而不是直接使用字符串字面量。
-   `viewName` : <br />
    视图的名字，工厂函数通过这个名字去找到视图的源码。
-   `viewContent` : <br />
    仅仅和XML视图以及JSON视图相关，如果一个视图很简单（比如就一两个控件），可以通过这个属性来定义视图的内容，当 `viewName` 和 `viewContent` 同时定义的话，只有viewName会起作用，而viewContent会被忽略。
-   `Controller` :<br />
    可以是任意控制器的实例，如果这里给定了控制器，则视图中定义的控制器会被覆盖。
-   `viewData` : <br />
    仅JS视图可以使用这个属性，viewData可以用来保存一些用户自定义的数据，并且在整个视图及对应的控制器的生命周期中这些数据都是可用的。


### 预定义的视图类型 {#预定义的视图类型}


#### JS View {#js-view}

定一个JS view文件只需要把文件名命为类似xxx.view.js的形式就可以了，其中xxx就是视图的名字，在实例化视图的时候，这个名字就是需要传给工厂函数的viewName，当然，记得加上命名空间。

我们需要实现以下两个方法来完成JS视图的定义：

-   `getControllerName()` : <br />
    指定属于该视图的控制器，如果该方法没有实现，或者返回空，则这个视图就没有控制器
-   `createContent()` : <br />
    当对应的控制器被初始化之后，这个方法会被而且仅会被调用一次来初始化UI。

<!--listend-->

```javascript
   sap.ui.jsview("sap.hcm.Address", {  // this View file is called Address.view.js

   getControllerName: function() {
      return "sap.hcm.Address";     // the Controller lives in Address.controller.js
   },

   createContent: function(oController) {
      var oButton = new sap.ui.commons.Button({text:"Hello JS View"});
      oButton.attachPress(oController.handleButtonClicked);
      return oButton;
   }

});
```

这里要提到一个要注意的地方，就是当我们在JS view中定义一个控件的时候，有时候我们需要同时定义一个id以方便之后引用，这时需要注意的是，我们不能直接给一个字符串字面量，我们通过类的实例方法 `View.createId('idstring')` 来实现，通过这种方式返回的id会在我们定义的idstring之前加上所在的类实例id作为前缀以保证唯一性。
但是如果是申明式的视图类型，就无需使用这个createId方法了，视图解析器会自动来帮我们做这个事情。


#### XML View {#xml-view}

XML view文件以xxx.view.xml作为文件名。一个XML view定义的示例如下：

```xml
<mvc:View controllerName="sap.hcm.Address" xmlns="sap.ui.commons" xmlns:mvc="sap.ui.core.mvc">
   <Panel>
      <Image src="http://www.sap.com/global/ui/images/global/sap-logo.png"/>
      <Button text="Press Me!"/>
   </Panel>
</mvc:View>
```

XML view中需要注意的一个问题是命名空间，在XML中定义任何一个控件的时候，都需要加上命名空间。


#### JSON View {#json-view}

JSON view以xxx.view.json作为文件名。一个JSON view定义的示例如下：

```javascript
{
   "Type":"sap.ui.core.mvc.JSONView",
   "controllerName":"sap.hcm.Address",
   "content": [{
      "Type":"sap.ui.commons.Image",
      "id":"MyImage",
      "src":"http://www.sap.com/global/ui/images/global/sap-logo.png"
   },
   {
      "Type":"sap.ui.commons.Button",
      "id":"MyButton",
      "text":"Press Me"

   }]

}
```

在JSON view中可以使用字符串、布尔值以及null。


#### HTML View {#html-view}

HTML view就是通过HTML标签来定义的视图，文件名类似 xxx.view.html。样例如下：

```HTML
  <template data-controller-name="example.mvc.test">
   Hello
  <h1>Title</h1>
  <div>Embedded HTML</div>
  <div class="test test2 test3" data-sap-ui-type="sap.ui.commons.Panel" id="myPanel">
  <div class="test test2 test3" data-sap-ui-type="sap.ui.commons.Button" id="Button1" data-text="Hello World" data-press="doIt"></div>
  <div data-sap-ui-type="sap.ui.commons.Button" id="Button2" data-text="Hello"></div>
  <div data-sap-ui-type="sap.ui.core.mvc.HTMLView" id="MyHTMLView" data-view-name="example.mvc.test2"></div>
  <div data-sap-ui-type="sap.ui.core.mvc.JSView" id="MyJSView" data-view-name="example.mvc.test2"></div>
  <div data-sap-ui-type="sap.ui.core.mvc.JSONView" id="MyJSONView" data-view-name="example.mvc.test2"></div>
  <div data-sap-ui-type="sap.ui.core.mvc.XMLView" id="MyXMLView" data-view-name="example.mvc.test2"></div>
  </div>
```

所有视图相关的属性都可以通过用data-<property name>的方式在tag <template>中定义。


## 控制器 {#控制器}

控制器用来定义业务或者页面的逻辑。控制器文件必须命名为xxx.controller.js。定义一个控制器的样例如下：

```javascript
	sap.ui.controller("sap.hcm.Address", {
   // controller logic goes here
});
```


### 生命周期 (Lifecyle Hooks) {#生命周期--lifecyle-hooks}

对于视图的不同生命周期状态，在控制器中有对应的钩子 （Hooks） 可以让我们对应于不同的状态实现一些特定的功能。
UI5提供了下面这些Hooks：

-   `onInit()` <br />
    视图被实例化时，当所有的控件已经被创建完成的时候被触发。<br />
    可以在视图显示之前做一些修改，或者做一些其他的一次性初始化工作。
-   `onExit()` <br />
    当视图被销毁的时候被触发。<br />
    一般可以用来释放资源或者最终确定一些活动的状态。
-   `onAfterRendering()` <br />
    当视图被完全渲染的时候被触发。<br />
    此时视图已经出现在DOM中了，可以用来操作DOM元素，修改一些HTML。
-   `onBeforeRendering()` <br />
    当视图被重新渲染的时候被调用，注意在视图第一次渲染的时候是不会被调用的。<br />


## 总结 {#总结}

MVC是UI5的基础开发模型，不管你有没有去刻意的了解它，但是只要你开发UI5，你肯定已经在使用它了。我个人觉得M和C应该还是比较容易理解的，按照API去定义或者用系统生成的文件框架基本上都不会有什么问题。
而问题一般都会出现在V，因为UI5里的控件太多，我们不可能掌握所有控件的用法，即使对于一些熟悉的控件，也没有机会在所有四种预定义的视图模型中都去实践一遍用法，所以很多时候，我们不知道一个控件该怎么用。
这个时候，UI5的帮助文档中的Explorer会是我们的好朋友，里面列了绝大多数常用控件的用法示例。但也有问题，这就是Explorer中基本上所有的视图都是用的XML类型定义的，所以我们还需要掌握API的阅读方法，如何转换成对应于不同视图的用法，这就需要
多多实践了。

[^fn:1]: [jQuery回调、递延对象总结（上篇）—— jQuery.Callbacks](http://www.cnblogs.com/yangjunhua/p/3508502.html)
[^fn:2]: [Category: Deferred Object](https://api.jquery.com/category/deferred-object/)
