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
xxx.html同时加载后面两个js，目录代码全放xxxCatalog.js代码,如下图所示导入js
```html
<script type="text/javascript" src="js/operDescCatalog.js"></script>
<script type="text/javascript" src="js/operDesc.js"></script>
```
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
			<!-- 右键菜单位置 -->
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
下面初始化右键菜单,右键菜单如下图所示：<br/>
![](https://github.com/tsmairc/ztree/blob/master/img/treeRight.png?raw=true)

```javascript
$.contextMenu({
  selector：'#right_menu',//上面html中定义的菜单div
  items: {},//菜单项
  callback： function(key, option){}//委托菜单事件
});
```

新增&修改目录--这里新增跟修改都通过弹窗的方式显示界面，有些界面是通过在树的右方显示，这里先介绍通过弹窗的方式显示界面,如下面所示效果图：
![](https://github.com/tsmairc/ztree/blob/master/img/modifyTree2.png?raw=true)

```javascript
/**
 * arg1 要显示的html内容或者dom对象
 * arg2 {title: "标题", width: "宽度", height: "高度", autoClose: "点击确定是否自动关闭", yes: "确定按钮方法", cancel: "取消按钮方法",    success: "层弹出后的成功回调方法"}
 **/
window.top.Utils.showDialog($("#addStairCatalog").html(), {title: "新增根目录", autoClose: false, width: "550px", height: "230px",
  yes: function(index){
    //获取表格内容
    var data = $("#addStairCatalogForm", window.top.document).xform("getData");
    //校验
    if($.isEmptyObject(data.catalog_name)){
      window.top.Utils.alert("目录名称不能为空");
      return false;
    }
    //组装入参报文
    var params = {catalog_id: "-1", catalog_name: data.catalog_name, catalog_desc: data.catalog_desc};
    //请求后台
    me.insertStairCatalog(params);
    //关闭显示层
    window.top.layer.close(index);
  },
  success: function(){
    //清空数据
    $("#addStairCatalogForm", window.top.document).xform("clear");
  },
  cancel:function(){}
});
```

下面介绍插入操作
```javascript
insertStairCatalog: function(params){
  Invoker.async("OperDescController", "insertCatalog", params, function(data){
    if(data.res_code != "00000"){
      //请求不成功，显示错误信息
      window.top.Utils.alert(data.res_message);
    }
    else{
      //获取树
      var ztree = $.fn.zTree.getZTreeObj("tree");
      if(window.operDescCatalog.selNode){
        //selNode是右键的时候添加上operDescCatalog的变量
        if(window.operDescCatalog.selNode.isParent == false){
          var catalog_child = {
              catalog_id: data.result.catalog_id,
              catalog_name: data.result.catalog_name,
              catalog_type: "C",
              catalog_desc: data.result.catalog_desc,
              isParent: true
          };
	  //为selNode增加子树
          ztree.addNodes(window.operDescCatalog.selNode, catalog_child);
        }
        //强行异步加载父节点的子节点
	//reloadType = "refresh" 表示清空后重新加载
        ztree.reAsyncChildNodes(window.operDescCatalog.selNode, "refresh");
      }
      else{
        var catalog = {
          catalog_id:data.result.catalog_id,
          catalog_name:data.result.catalog_name,
          catalog_type:"C",
          catalog_desc: data.result.catalog_desc,
          isParent : false
        };
	//增加节点
        ztree.addNodes(null, catalog);
      }
    }
  });
},
```
修改树，修改树比较简单，直接修改selNode变量，再把变量更新到树时面。
```javascriipt
Invoker.async("OperDescController","modifyCatalog", params, function(data){
  if(data.res_code != "00000"){
    window.top.Utils.alert(data.res_message);
  }
  else{
    var ztree = $.fn.zTree.getZTreeObj("tree");
    window.operDescCatalog.selNode.catalog_name = data.result.catalog_name;
    window.operDescCatalog.selNode.catalog_desc = data.result.catalog_desc;
    ztree.updateNode(window.operDescCatalog.selNode);
  }
});
```
下面介绍直接在右边显示的情况：
![](https://github.com/tsmairc/ztree/blob/master/img/tree2.png?raw=true)
main.html -- main.js 主界面事件
          -- catalog.js 主界面树事件（点击和右击和右击出现的菜单事件）
	  -- main.js会load catalogOper.html和catalogOper.js，这两个文件用于右边界面保存的前端事件(我不喜欢用load方法，因为无法在直接dubugr代码，要在代码中加debugger。)<br/>
修改和保存使用同一个界面，后台判断是否有目录id区分当前是新增还是修改。



2.主页面新增行：（界面上方的绿色新增按钮）
```javascript
$("#operDesc_ins_btn").on("click", function(){
  //是否已经选中目录
  if(!window.operDescCatalog.catalog_id){
    window.top.Utils.alert("请先选择目录");
    return;
  }
  //TODO: 这里的详情和新增界面可以做在一起，前期没做分析，不知道可以做一起。
  //隐藏列表界面
  $("#listInfo").hide();
  //隐藏详情界面
  $("#detailInfo").hide();
  //显示新增界面
  $("#addOperDescDiv").show();
  
  var url = "operDescAdd.html";
  var catalog_id = window.operDescCatalog.catalog_id;
  var catalog_name = window.operDescCatalog.catalog_name;
  //组装请求url
  if(catalog_id){
    url += "?catalog_id=" + catalog_id + "&catalog_name=" + encodeURIComponent(catalog_name);
  }
  //iframe加载页面
  Utils.loadFrame($("#addOperDescDiv iframe"), url, function(){
    Utils.refreshFrameHeight(this);
  });
});
```

```html
<!-- 新增操作说明 与列表在同一位置-->
<div class="col-sm-12" id="addOperDescDiv" style="display:none;">
	<div class="panel panel-default">
		<div class="panel-heading">
			<h3 class="panel-title">新增操作说明</h3>
			<div class="panel-options">
				<a href="javascript:void(0)">
					<i class="fa-reply"></i>
				</a>&nbsp;
				<a href="javascript:void(0)" data-toggle="panel">
					<span class="collapse-icon">&ndash;</span>
					<span class="expand-icon">+</span>
				</a>
			</div>
		</div>		
		<div class="panel-body">
	                <!-- 通加这个iframe加载对应的页面 -->
			<iframe id="addOperDescFrame" scr="" width="100%" height="100%" scrolling="no" frameborder="0" allowfullscreen="true" webkitallowfullscreen="true" mozallowfullscreen="true"></iframe>
		</div>
	</div>
</div>
```
详情页面跟上面新增页面类似，上面也提到详情跟新增页面可以做在一起的，详情与新增的区别只是在
~ 1.详情需要在加载页面的时候，向后台请求数据，并显示在界面中
~ 2.详情需要在加载页面的时候，调整下方的保存、取消、编辑按钮，显示适合的按钮
~ 3.后台需要做调整，把新增跟修改方法做在一起，根据前台传参有没主键id去区分哪个是新增，哪个是修改。

