# 首先说说为什么我要创建这个项目
在这个星期，我要做一个视频播放的功能，当然不仅仅包括视频播放这个功能，还有后台增删查改的一些功能。这个任务的重点也在界面而不在后台。经历了一年的js基础学习，我对自己的前端基础还是十分自信的，谁知，我做了两天还没做出这个功能，抱着我要做完的心态，我用下班的时候也去做，发现自己越做越慢，这时我回想了一下，自己的犯的错。
* 1.做前没有先定好要的界面模式，边做边想，自己给自己加大了开发难度
* 2.事先没有学习常做界面的逻辑，盲目去抄代码。

所以我现在对常用界面模式做一些分析，同时以后我也会先定好界面再去写代码。

## 界面
* 1.下面是常用展示列表的界面，左边是树，右边是列表。
![](https://github.com/tsmairc/ztree/blob/master/img/list.png?raw=true)

* 1.1 介绍左边树的用法
* 1.1.1 树的加载
首先介绍各文件位置
xxx.html<br/>
xxx.js<br/>
xxxCatalog.js<br/>
xxx.html同时加载后面两个js，目录代码全放xxxCatalog.js代码。
树的html代码
```html
<div class="panel panel-default">
	<div class="panel-heading">
		<h3 class="panel-title">操作说明目录</h3>
		<div class="panel-options">
			<a href="javascript:void(0)" data-toggle="panel"> <span
				class="collapse-icon">&ndash;</span> <span class="expand-icon">+</span>
			</a> <a href="javascript:void(0)" data-toggle="reload"> <i
				class="fa-rotate-right reload"></i> </a>
		</div>
	</div>
	<div class="panel-body" id="treeDiv">
		<div class="zTreeDemoBackground left">
			<ul id="tree" class="ztree" style="overflow:auto;"></ul>
			<div id="right_menu"></div>
		</div>
	</div>
</div>
```
树的加载js代码，下面这些初始化的代码都是参考ztree的官方文档做的。有兴趣的可以去zTree官网查看。
```javascript
var setting = {
	view: {
		nameIsHTML : true
	},
	async: {
		enable : true,
    //这里url，我们对ztree的有改造过
		url : "OperDecController.getCatalog",
		autoParam : ["catalog_id", "catalog_name", "catalog_type"]
	},
	data: {
		key: {
			name :"catalog_name"
		},
		simpleData :{
			enable : true,
			idKey : "catalog_id"
		}
	},
	callback: {
		onClick : function(event, treeId, treeNode) {
			var params = {catalog_id: treeNode.catalog_id};
			me.catalog_id = treeNode.catalog_id;
			me.catalog_name = treeNode.catalog_name;
      //这里代码的意思是点击选中，加载右边列表的内容，后面我会介绍这个
			window.operDec.queryOperDecByCond(params);
			
			//停止冒泡，这里的用处是点击空白地方可以取消选中，所以里面的选中项目不让冒泡，以免触发取消选中效果。
			if(event && event.stopPropagation){
				event.stopPropagation(); 
			}
		},
		onRightClick: function(event, treeId, treeNode){
      //下面的menu变量我主要用于右键菜单的显示隐藏控制
			if($.isEmptyObject(treeNode)){
				me.selNode = "";
				me.menu1 = true;me.menu2 = false;me.menu3 = false;me.menu4 = false;
			}
			if(treeNode != null){
				if(treeNode.catalog_type == "C"){
					me.menu1 = false;me.menu2 = true;me.menu3 = true;me.menu4 = true;
				}
				else if(treeNode.catalog_type == "A"){
					me.menu1 = false;me.menu2 = false;me.menu3 = false;me.menu4 = false;
				}
				else if(treeNode.catalog_type == "V"){
					me.menu1 = false;me.menu2 = false;me.menu3 = false;me.menu4 = false;
				}
				me.selNode = treeNode;
			}
			$("#right_menu").contextMenu({x: event.pageX, y: event.pageY});
		}
	}
};

$.fn.zTree.init($("#tree"), setting);
```
