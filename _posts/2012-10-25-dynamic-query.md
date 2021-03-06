---
title: 网页动态查询条件的实现
author: Dozer
layout: post
permalink: /2012/10/dynamic-query.html
categories:
  - 编程技术
tags:
  - AspDotNet
  - html
  - javascript
---

### 场景

最近有一个需求，会在 mongodb 中插入各种类型的数据，算是记录业务日志的数据库吧。

因为业务对象类型都不同，所以插入的数据格式也完全不同。

除此之外，还需要提供一个查询界面，可以搜索数据。

插入数据没任何问题，但是查询就…

<!--more-->

### 查询设计方案

首先想到的是让用户直接输入 mongodb 查询语法，类似 json 格式。但是使用者虽然也是开发，可都不熟悉这个语法，所以放弃了。

第二个想法是让用户输入 SQL 语句，然后转换… 结果以失败而告终。

最后，看到了 iTunes 智能播放列表的交互设计：

[<img class="alignnone size-medium wp-image-894" title="itunes" alt="itunes" src="/uploads/2012/10/itunes-300x179.png" width="300" height="179" />][1]

这里，你可以插入一个条件，也可以插入一组条件（相当于插入了一个括号，括号内是许多条件）。

图中的表达式可以认为是： Score > 3 && Type == "Music" && Author == "" && ( Author == "" && Author == "" && Author == "")

也就是说，这样的交互完全可以实现各种嵌套逻辑。

&nbsp;

### 数据结构

为了设计出这样的结构，肯定要先好好想一下数据结构。

分析后感觉，这里其实就两种类型，一个可以认为是 QueryGroup，一个可以认为是 QueryItem。

代码如下：

    public class QueryGroup
        {
            public GroupType GroupType { get; set; }
            public List<QueryItem> Items { get; set; }
            public List<QueryGroup> Groups { get; set; }
        }

        public class QueryItem
        {
            public string Name { get; set; }
            public QuerySymbol OperatorType { get; set; }
            public string Value { get; set; }
            public DataType ValueType { get; set; }
        }

`QueryGroup` 包含了一组查询条件，也包含了一组子 `QueryGroup`，另外还有一个重要的属性 `GroupType` ，代表这组数据的逻辑关系是 And 还是 Or。也就是上述界面中的“任何”和“任意”选项。

`QueryItem` 内部属性分别是字段名、逻辑操作类型（等于、不等于、大于…）、和属性类型（整数、文本…）。

&nbsp;

设计完数据结构后会有几个难点：

1.  前端交互怎么设计？
2.  如何传给后端？
3.  后端得到数据后如何转换成查询表达式？

那下面就一个个来攻克吧！

&nbsp;

### 前端设计交互

[<img class="alignnone size-medium wp-image-895" title="ui" alt="ui" src="/uploads/2012/10/ui-300x138.png" width="300" height="138" />][2]

这里用的是 bootstrap ，界面非常好看！

先来看看前端设计方案吧，上面是动态条件，下面是一些固定的条件。

这里的结构和上面的数据结构一致，把 html 分两类，`QueryGroup` 和 `QueryItem`。

分别放在两个隐藏的 `div` 中，当做模版使用。

代码如下：

&nbsp;

    <div style="display: none;">
        <div class="query-group-template">
            <div class="query-group well">
                <div class="query-title">
                    <span class="help-inline">匹配以下</span>
                    <select class="input-small group-type">
                        <option value="1">全部</option>
                        <option value="2">任何</option>
                    </select>
                    <span class="help-inline">规则：</span>
                    <button type="button" class="btn btn-mini btn-success add-query-item" title="增加一个条件">
                        <i class="icon-plus icon-white"></i>
                    </button>
                    <button type="button" class="btn btn-mini btn-info add-query-group" title="增加一组条件">
                        <i class="icon-th-list icon-white"></i>
                    </button>
                    <button type="button" class="btn btn-mini btn-danger delete-query-group" title="删除这组条件">
                        <i class="icon-minus icon-white"></i>
                    </button>
                </div>
            </div>
        </div>
        <div class="query-item-template">
            <div class="query-item">
                <input type="text" value="" placeholder="字段名" title="字段名" class="property-name" />
                <select class="input-mini operate-type" title="条件">
                    <option value="1">==</option>
                    <option value="2">!=</option>
                    <option value="3">></option>
                    <option value="4">>=</option>
                    <option value="5"><</option>
                    <option value="6"><=</option>
                    <option value="7">LK</option>
                </select>
                <input type="text" class="query-value" value="" placeholder="值" title="值" />
                <select class="input-medium value-type">
                    <option value="3">String</option>
                    <option value="1">Int</option>
                    <option value="2">Double</option>
                    <option value="4">DateTime</option>
                </select>
                <button type="button" class="btn btn-mini btn-danger delete-query-item" title="删除条件">
                    <i class="icon-minus icon-white"></i>
                </button>
            </div>
        </div>
    </div>

这里其实不难，最关键的地方其实是各个按钮的事件了。

仔细看一下，一共有4个按钮：

上面三个分别是：增加一行条件、增加一组条件、删除本组条件。

单个条件右边一个是：删除此条件。

这里逻辑其实非常简单：

    $('#queryContainer').append($('.query-group-template>.query-group').clone())
        $('#queryContainer>.query-group').first().find('.delete-query-group').remove();

        $('button.add-query-item').live('click', function () {
            $(this).parent().parent().append($('.query-item-template>.query-item').clone());
            return false;
        });

        $('button.add-query-group').live('click', function () {
            $(this).parent().parent().append($('.query-group-template>.query-group').clone());
            return false;
        });

        $('button.delete-query-group').live('click', function () {
            if (!$(this).parent().parent().parent().hasClass('query-group')) { return false; }
            $(this).parent().parent().remove();
            return false;
        });

        $('button.delete-query-item').live('click', function () {
            $(this).parent().remove();
            return false;
        });

另外，看代码前两行，第一次加载的时候别忘了先加一组条件，并且把默认组的“删除本组条件”这个按钮去掉吧。

&nbsp;

### 前端数据处理

界面交互真的很简单，但是怎么把这个数据传给后端呢？

把表单一个个字段取出来传过去？那后端要哭了… 完全是乱七八糟的一堆数据。

那… 既然查询条件的结构是非常清晰的，为什么不能先变成 javascript 中的对象呢？

然后，把这个对象序列化…

再然后，把 json 传给后端…

最后，后端定义同样结构的类型，然后反序列化…

也就是说，在这个交互的过程中，只需要把表单数据实例化成 javascript 中的对象即可！

&nbsp;

那我先来定义两个对象（注意字段名一定要和后端一样）：

    function QueryGroup() {
        this.GroupType = 0;
        this.Items = [];
        this.Groups = [];
    }
    function QueryItem() {
        this.Name = '';
        this.OperatorType = 0;
        this.Value = '';
        this.ValueType = 0;
    }

&nbsp;

实例化成对象的方法也非常简单，需要用到递归，基本逻辑是：

对最外层 `QueryGroup` 内部的对象循环一次，如果是 `QueryItem` 就指着取值，如果还是 `QueryGroup` 就递归调用此方法。

代码如下：

    function GetQueryGroup(group) {
        group = $(group);
        var queryGroup = new QueryGroup();
        queryGroup.GroupType = parseInt(group.find('.group-type').val());

        var queryItems = group.children('.query-item');
        for (var k = 0; k < queryItems.length; k++) {
            var queryItem = new QueryItem();
            queryItem.Name = $(queryItems[k]).find('.property-name').val();
            queryItem.OperatorType = parseInt($(queryItems[k]).find('.operate-type').val());
            queryItem.Value = $(queryItems[k]).find('.query-value').val();
            queryItem.ValueType = parseInt($(queryItems[k]).find('.value-type').val());
            queryGroup.Items.push(queryItem);
        }

        var childGroups = group.children('.query-group');
        for (var k = 0; k < childGroups.length; k++) {
            queryGroup.Groups.push(GetQueryGroup(childGroups[k]));
        }

        return queryGroup;
    }

&nbsp;

最后，表单是表单提交，最终会生成一个对象，把这个对象序列化成 json 然后编码一下：

`encodeURIComponent(JSON.stringify(item))`

&nbsp;

### 后端数据处理

后端数据处理主要分两个部分：反序列化、转换成查询条件。

数据结构在上面已经定义过了，只要字段名和 json 中的一样，就可以直接反序列化。

    var json = Uri.UnescapeDataString(Request["query"]);
    var item = JsonConvert.DeserializeObject<QueryGroup>(json);

两行代码，它就变成 .net 中的对象了！

&nbsp;

最后，生成查询条件其实也非常简单，也是一个方法，递归调用即可，基本逻辑和前段把表单数据实例化的过程很像。

我在 `QueryGroup` 中扩展了一个方法，其中 `ICriteria` 和 `IMongoQuery` 结构类似，用过 mongodb 的同学当它是 `IMongoQuery` 即可，它只是包了一层，最终也是生成 `IMongoQuery`。

    public class QueryGroup
        {
            public GroupType GroupType { get; set; }
            public List<QueryItem> Items { get; set; }
            public List<QueryGroup> Groups { get; set; }

            public ICriteria ToICriteria()
            {
                ICriteria result = null;
                foreach (var criteria in GetICriteriaList())
                {
                    if (result == null)
                    {
                        result = criteria;
                        continue;
                    }

                    if (GroupType == Model.GroupType.AndAlse)
                    {
                        result = result.Add(criteria);
                        continue;
                    }

                    if (GroupType == Model.GroupType.OrElse)
                    {
                        result = result.Or(criteria);
                        continue;
                    }
                }

                return result;
            }

            private List<ICriteria> GetICriteriaList()
            {
                var list = new List<ICriteria>();
                foreach (var item in Items)
                {
                    list.Add(new Criteria(item.Name, item.OperatorType, new QueryValue(item.ValueType, item.Value, FieldHierarchyLevel.Child)));
                }
                foreach (var group in Groups)
                {
                    list.Add(group.ToICriteria());
                }
                return list;
            }
        }

得到查询条件对象后，直接调用相关查询方法即可。

&nbsp;

### 后记

本场景中用的是 mongodb ，所以最终转换出来的是 mongodb 查询对象。其实，如果是转换 SQL 也是非常方便的。

另外，稍微复杂一点，转换成 .net 中的表达式树也是木有问题的！

最后附上 gif 的 Demo：

[<img class="alignnone size-medium wp-image-903" title="demo" alt="demo" src="/uploads/2012/10/demo-300x146.gif" width="300" height="146" />][3]

&nbsp;

 [1]: /uploads/2012/10/itunes.png
 [2]: /uploads/2012/10/ui.png
 [3]: /uploads/2012/10/demo.gif
