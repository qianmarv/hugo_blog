+++
title = "UI5入门系列之三：MVC （上） - 模型"
date = 2015-09-04
tags = ["SAPUI5"]
categories = ["编程"]
draft = false
+++

这次我们来一起学习MVC，这个专题分为两个小节，本次主要是总览以及模型，下一次着重会介绍视图以及控制器，因为控制器其实没有太多可以讲的，所以和视图合并在一块。
<!--more-->


## Model View Controller （MVC）的基本概念 {#model-view-controller-mvc-的基本概念}

MVC，对于大多数人说，这是一个讲烂了的概念。不过，既然这是一个入门系列，还是要稍微讲一讲。

-   M 代表Model - 模型 <br />
    一般用来管理数据层，比如绑定后台数据。
-   V 代表View - 视图 <br />
    一般用来处理展示层，比如具体前端UI的展示。
-   C 代表Controller - 控制器 <br />
    一般用来处理逻辑，可是页面逻辑也可以是协调处理数据的逻辑

不同的程序设计语言，只要涉及到有前端交互的，一般都会有MVC的概念，这个概念是相通的，但是具体
到细节层面还是会有差别，所以下面的内容都是针对UI5的MVC而言。

{{< figure src="/ox-hugo/starter_3_1.png" >}}

MVC最主要的目的是把展示与逻辑、数据分离开来，使得程序更容易阅读、容易理解，从而也更可以维护，
同时，也增加了可扩展性。

视图和控制器一般是1:1对应的，但是也可以创建一个没有视图的控制器，这样的控制器叫做应用控制器， `Application Controller` ；同时，也可以创建一个没有控制器的视图。


## UI5中的模型概念 {#ui5中的模型概念}

前面讲到，模型的主要作用是提供数据，比如如何从后台数据库获取数据，如何更新后台数据等等。

UI5提供了以下预定义的模型：

-   JSON 模型：<br />
    属于客户端(client-side model)模型，所以比较适合小型数据集，JSON模型支持双向绑定。 <br />
    类名：[sap.ui.model.json.JSONModel](https://openui5.hana.ondemand.com/docs/api/symbols/sap.ui.model.json.JSONModel.html)

-   XML模型： <br />
    同样属于客户端模型。<br />
    类名：[sap.ui.model.xml.XMLModel](https://openui5.hana.ondemand.com/#docs/api/symbols/sap.ui.model.xml.XMLModel.html)

-   Resource模型： <br />
    这个比较特殊，它是通过资源包(Resource Bundle)的方式来处理数据，一般我们可以用它来做多语言处理。 <br />
    类名：[sap.ui.model.resource.ResourceModel](https://openui5.hana.ondemand.com/#docs/api/symbols/sap.ui.model.resource.ResourceModel.html)

-   OData模型： <br />
    属于服务端模型(server-side model)，所以必须提供服务端OData的资源URL来绑定到对应的UI5控件。 <br />
    类名：[sap.ui.model.odata.ODataModel](https://openui5.hana.ondemand.com/#docs/api/symbols/sap.ui.model.odata.ODataModel.html)


### 绑定模式 {#绑定模式}

这里有一个绑定模式(Binding Model)的概念，绑定模式定了数据资源是如何被绑定的，UI5里面绑定模式有三种：

-   One Way: 从模型到视图的单向绑定。
    我们可以简单的认为单向绑定是一种只读的绑定，如果视图里面的某个字段的值变更了，不会对模型造成影响。如果想要更新数据，就必须要通过控制器来手动控制更新数据到模型，然后这个模型所绑定到视图的所有字段都会自动更新一次。 <br />
    `View(UI: input box value)` --manually update--> `Model` --automatically--> `View`&nbsp;[^fn:1]

<!--listend-->

-   Two Way: 从模型到视图和视图到模型的双向绑定，如果视图中的某个字段的值变更了，模型会自动更新，同时，这个模型所绑定到视图的其他控件数据。 <br />
    `View(UI: input box value)` --automatically--> `Model` --automatically--> `View`

-   One Time: 一次性绑定，有些类似单向绑定，但是如果模型有变更的话，系统不会自动刷新数据，所以一般用来绑定静态文本。

<a id="table--tab:basic-data"></a>
<div class="table-caption">
  <span class="table-number"><a href="#table--tab:basic-data">Table 1</a></span>:
  模型以及相应支持的绑定模式
</div>

| Model          | One-way | Two-way | One-time |
|----------------|---------|---------|----------|
| Resource model | --      | --      | X        |
| JSON model     | X       | X       | X        |
| XML model      | X       | X       | X        |
| OData model    | X       | X       | X        |


### 绑定类型 {#绑定类型}

了解了绑定模式，我们再来看绑定类型(Binding Type)，简单的说绑定模式要处理的是怎么绑定的问题，绑定类型要处理的是绑定到哪的问题。
UI5中提供了三种绑定类型。


#### Property Binding {#property-binding}

Property Binding允许控件的某些属性直接绑定到模型的数据从而可以自动初始化以及当后台数据变动时可以自动刷新。

定义property Binding有两种方法：

-   通过控件的构造器在 `seetings` 对象中绑定
-   通过控件的 `bindProperty` 方法绑定

一般最常见的方式就是直接利用构造器的settings对象来直接绑定模型，比如：

```javascript
     var oTextField = new sap.ui.commons.TextField({
         value: "{/company/name}"
     });
```

稍稍做一点说明，当有多个数据模型绑定到当前控件以及祖先控件时，需要用在绑定的字段之前加上模型名称，比如 "{mymodel>/company/name}" 。 <br />

如果需要对这个绑定做更多的定义，可以跟进一步，用以下的扩展语法格式：

```javascript
     var oTextField = new sap.ui.commons.TextField({
     value: {
		path: "/company/name",
		mode: sap.ui.model.BindingMode.OneWay
	}
     });
```

这里显式的把绑定的模型字段赋给 `path` ，注意这里就不需要套上大括号了。 <br />

通过控件的 `bindProperty` 方法则提供了更多的选项，可以让用户在稍后而不是初始化的时候来绑定。

```javascript
     oTextField.bindProperty("value", "/company/name");
```

以及类似的

```javascript
     oTextField.bindProperty("value", {
	path: "value",
	type: new sap.ui.model.type.Integer()
     });
```

有一些控件做了进一步的封装，比如文本框，由于 `value` 是经常需要用来绑定模型的属性，所以直接提供了 `bindValue` 方法以方便使用。

```javascript
     oTextField.bindValue("/company/name");
```

当需要对绑定的字段做更多的处理，而不是简单的一对一绑定时，UI5还提供了 `formatter` 这个属性方法，用法如下：

```javascript
oTextField.bindProperty("value", "/company/title", function(sValue) {
    return sValue && sValue.toUpperCase();
});

oControl = new sap.ui.commons.TextField({
    value: {
        path:"/company/revenue",
        formatter: function(fValue) {
            if (fValue) {
                return "> " + Math.floor(fValue/1000000) + "M";
            }
            return "0";
        }
    }
})
```

例子中分别提供了用构造器的方法和用 `bindProperty` 的方法来对要绑定的字段做format的示例。


#### Aggregation Binding {#aggregation-binding}

Aggregation binding主要是用来绑定子控件，对应的模型数据的结构也必须符合一定的树状结构关系。

同样，和Property Binding类似，可以通过控件构造器的 `settings` 对象或者是 `bindAggregation` 方法来绑定模型，
但是有一点不同的是，Aggregation Binding需要指定所谓的 `template` ，这是因为Property Binding是一个数据条目绑定到一个控件的字段，属于一对一的绑定，而Aggregation Binding，则是一组数据绑定到一组控件，比如将多个销售订单绑定到一个表控件的多个item中，属于一个数组到另一个数组的绑定。
这个所谓的 `template` 其实就是我们创建一个item，然后系统在render的时候会参照我们创建的这个item，拷贝多个相同的items并绑定对应的数据。可以想象成两个数组，一个是数据数组，一个是Item控件数组，数据已经确定了，但是系统不知道需要创建哪个Item来绑定数据，需要我们帮它创建一个，接下来，系统就会创建和数据数组相同数量的Item控件，并且绑定和数据数组相同索引的数据。

```javascript
var oItemTemplate = new sap.ui.core.ListItem({text:"{name}"});
var oComboBox = new sap.ui.commons.ComboBox({
    items: {
		path: "/company/contacts",
		template: oItemTemplate
	}
});
```

或者通过方法调用来绑定：

```javascript
oComboBox.bindAggregation("items", "/company/contacts", new sap.ui.core.ListItem({text:"{name}"}));
```


#### Element Binding {#element-binding}

Element Binding可以允许我们把模型数据的某个特定的对象绑定到一个控件上（不是控件的某个属性），从而我们可以在控件的 `property` 或者 `aggregation` 中直接使用绑定到这一级的模型的下级对象，换句话说，允许我们在绑定数据的时候使用相对路径的方式。

譬如：

```javascript
oControl.bindElement("/company");
oControl.bindProperty("value", "name");
```

Element Binding使用场景比较简单，这里就不多说了。


## 分析与总结 {#分析与总结}

本次我们学习了UI5中的模型的概念、类型以及如何使用。
作为一套前端UI库，SAP的重点其实还是在于企业级的数据交互与展示，所以数据模型以及数据绑定就显得尤为重要，后面希望有机会可以一起探讨后端模型的输出，比如Netweaver以及HANA中是如何提供这些数据模型的。

[^fn:1]: Refer to <https://www.youtube.com/watch?v=vY5%5FifnvDa8> at 7:02.
