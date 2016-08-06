#参照组件概述#

# 整体设计 #

## 依赖环境 ##

组件采用Maven进行编译和打包发布，引入了iUAP平台的一些基础组件如iuap-log和iuap-cache，其对外提供的依赖方式如下：

	<dependency>

		<groupId>com.yonyou.iuap</groupId>

		<artifactId>uitemplate_ref</artifactId>

		<version>3.0.0-RC001</version>

		<classifier>classes</classifier>

	</dependency>


# 使用说明 #

##参照组件配置##


**1:在属性文件中，配置rest服务所 需要的服务器地址端口等信息**



	配置示例：

	

	refctx=http://127.0.0.1:8080




**2:工程中引入对uitemplate_ref组件的依赖**



	<dependency>

		<groupId>com.yonyou.iuap</groupId>

		<artifactId>uitemplate_ref</artifactId>

		<version>3.0.0-RC001</version>

		<classifier>classes</classifier>

	</dependency>	



**3:前端代码中引用组件提供的API，将参照相关的js文件和css文件引入**


	1）参照基于uui的版本需要引入的js和css文件有：
	require.config({
		paths: {
			'jquery': "/uui/libs/jquery/jquery-1.11.2",
			'knockout': "/uui/libs/knockout/knockout-3.2.0.debug",
			'uui': "/uui/libs/uui/js/u",
			'css': "/uui/libs/requirejs/css",
			'bootstrap':"/uui/libs/bootstrap/js/bootstrap",
			'ztree':"/uui/libs/uui/js/u-tree",
			'reflib': "/uitemplate_web/static/js/uiref/reflib",
			'refer': "/uitemplate_web/static/js/uiref/refer",
			'refGrid': "/uitemplate_web/static/js/uiref/refGrid",
			'refGridtree': "/uitemplate_web/static/js/uiref/refGridtree",
			'refTree': "/uitemplate_web/static/js/uiref/refTree",
			'refcommon': "/uitemplate_web/static/js/uiref/refcommon",
			'uiReferComp' : "/uitemplate_web/static/js/uiref/uiReferComp"
		},
		
		shim: {
			bootstrap:{
				deps:["jquery"]
			},
			'uui': {
				deps: ["jquery", "bootstrap", "knockout"]
			},
			'ztree': {
				deps: ["jquery","uui","css!/uui/libs/uui/css/tree.css","css!/uui/libs/uui/fonts/font-awesome/css/font-awesome.css"]
			},
			'zTree.exhide':{
				deps: ["zTree"]
			},
			'reflib':{
	       	 	deps: ["jquery"]
	        },
			'refer':{
				deps:["jquery","reflib","ztree"]
			},
			'sco.message':{
				deps: ["jquery"]
			},
			'uiReferComp':{
				deps:["uui"]
			},
			'refGridtree':{
				deps: ["refer"]
			},
			'refGrid':{
				deps: ["refer"]
			},
			'refTree':{
				deps: ["refer"]
			},
			'refcommon':{
				deps: ["refer"]
			}
		},
		waitSeconds:60
	});
	<link href="/uitemplate_web/static/css/ref/ref.css" rel="stylesheet">
	<link href="/uitemplate_web/static/css/ref/jquery.scrollbar.css" rel="stylesheet">
	<link href="/uui/libs/uui/css/tree.css" rel="stylesheet">
	<link href="/uitemplate_web/static/trd/bootstrap-table/src/bootstrap-table.css" rel="stylesheet">

	2）参照不基于uui的版本需要引入的js和css文件有：
	require.config({
		baseUrl: "../",
		paths: {
			'jquery': "/uui/libs/jquery/jquery-1.11.2",
			'knockout': "/uui/libs/knockout/knockout-3.2.0.debug",
			'ztree':"/uui/libs/uui/js/u-tree",
			'reflib': "/uitemplate_web/static/js/uiref/reflib",
			'refer': "/uitemplate_web/static/js/uiref/refer",
			'refGrid': "/uitemplate_web/static/js/uiref/refGrid",
			'refGridtree': "/uitemplate_web/static/js/uiref/refGridtree",
			'refTree': "/uitemplate_web/static/js/uiref/refTree",
			'refcommon': "/uitemplate_web/static/js/uiref/refcommon"
		},
		shim: {
			 'jquery' :{
	     		 exports: '$'
	         },
	         'reflib':{
	        	 deps: ["jquery"]
	         },
	         'ztree': {
					deps: ["jquery","css!/uui/libs/uui/css/tree.css","css!/uui/libs/uui/fonts/font-awesome/css/font-awesome.css"]
			 },
			'refer':{
				deps:["jquery","reflib","ztree"]
			},
			'refGridtree':{
				deps: ["refer"]
			},
			'refGrid':{
				deps: ["refer"]
			},
			'refTree':{
				deps: ["refer"]
			},
			'refcommon':{
				deps: ["refer"]
			}
		}
	});
	<link href="/uitemplate_web/static/css/ref/ref.css" rel="stylesheet">
	<link href="/uitemplate_web/static/css/ref/jquery.scrollbar.css" rel="stylesheet">
	<link href="/uui/libs/uui/css/tree.css" rel="stylesheet">
	<link href="/uitemplate_web/static/trd/bootstrap-table/src/bootstrap-table.css" rel="stylesheet">


## 参照注册 ##
首先需要将参照的基本信息注册到ref_refinfo数据库表里，拿人员类别参照举例：
参照编码refcode为psncl_ref，参照名称refname为人员类别，rest服务url为
/uitemplate_web/psncl_ref/ ，业务对象pk为42c223a8，其他字段可以为空，调用接口类IUIrefService.java里面的addSaveRefModel方法，将此条数据插入到ref_refinfo表中，如下表所示：

<table>
  <tr>
    <th>pk_ref</th>
    <th>refname</th>
    <th>refcode</th>
    <th>refclass</th>
    <th>refurl</th>
    <th>md_entitypk</th>
    <th>productType</th>
  </tr>
  <tr>
    <td>b6779b</td>
    <td>人员类别</td>
    <td>psncl_ref</td>
    <td></td>
    <td>/uitemplate_web/psncl_ref/</td>
    <td>42c223a8</td>
    <td>hr</td>
  </tr>
</table>

此表里面的数据会根据业务对象的pk去查询（md_entitypk字段），如果md_entitypk字段没有数据，那么会查询此表里所有的参照注册数据，然后会在
ui模板设计器里列出查询出来的数据供选择，最后点击保存按钮生成模板。


#### 参照注册表ref_refinfo ####

<table>
  <tr>
    <th>编码</th>
    <th>名称</th>
    <th>说明</th>
  </tr>
  <tr>
    <td>pk_ref</td>
    <td>参照主键</td>
    <td>参照唯一性pk标识</td>
  </tr>
  <tr>
    <td>refname</td>
    <td>参照名称</td>
    <td>参照名称标识</td>
  </tr>
  <tr>
    <td>refcode</td>
    <td>参照编码</td>
    <td>参照编码标识，用来查询参照基本信息</td>
  </tr>
  <tr>
    <td>refclass</td>
    <td>参照反射类</td>
    <td>参照反射类标识</td>
  </tr>
   <tr>
    <td>refurl</td>
    <td>rest服务url地址</td>
    <td>rest服务url地址标识</td>
  </tr>
  <tr>
    <td>md_entitypk</td>
    <td>业务对象的主键</td>
    <td>业务对象的主键标识</td>
  </tr>
  <tr>
    <td>productType</td>
    <td>不同的产品类型</td>
    <td>不同的产品类型标识，比如平台，hr，nc等</td>
  </tr>
</table>

#### 参照注册后台接口类是IUIrefService.java 里面方法有： ####

<table style="border-collapse:collapse">
	<tr>
		<th>方法名</th>
		<th>参数</th>
		<th>返回值</th>
		<th>功能说明</th>
	</tr>
	<tr>
		<td>queryRefModelByPK</td>
		<td>String md_entitypk（此参数是业务对象pk）</td>
		<td>List &ltUIrefEntity&gt（返回参照注册信息的列表）</td>
		<td>根据业务对象pk来获取参照注册信息列表</td>
	</tr>
	<tr>
		<td>queryRefModelByCode</td>
		<td>String refCode（此参数是参照的编码）</td>
		<td>UIrefEntity（返回参照注册信息的实体）</td>
		<td>根据refCode来获取参照注册信息实体</td>
	</tr>
	<tr>
		<td>addSaveRefModel</td>
		<td>UIrefEntity uiRefEntity（对应参照注册的实体信息）</td>
		<td>String（返回保存参照注册信息后的主键）</td>
		<td>用来返回保存参照注册信息后的主键</td>
	</tr>
	<tr>
		<td>updateSaveRefModel</td>
		<td>UIrefEntity uiRefEntity（对应参照注册的实体信息）</td>
		<td>void</td>
		<td>用来更新参照注册信息</td>
	</tr>
	<tr>
		<td>deleteRefModel</td>
		<td>String refCode（对应参照编码信息）</td>
		<td>void</td>
		<td>根据refCode用来删除参照注册信息</td>
	</tr>
</table>

## 开发步骤 ##

#### 1.基于uui开发参照 ####

- 前端需要引入reflib.js, refer.js, refGrid.js, refGridtree.js, refTree.js,refcommon.js, uiReferComp.js,还需要引入uui里面的u-tree.js, u.js, knockout-3.2.0.debug.js, jquery-1.11.2.js，其中uiReferComp.js是与uui的datatable做数据交互用的接口文件。
			



		首先需要将参照的基本信息注册到ref_refinfo数据库表里，此表里会有这些字段：参照名称refname，参照编码refcode，
		参照反射类refclass，rest服务url地址refurl，元数据的主键md_entitypk，不同的产品类型productType

		如果用的是ui模板设计器，那就通过设计器界面选择不同参照后拿到refcode再去调用查询参照信息接口来获取此参照的基本信息，
		而后将参照的这些基本信息放到datatable的u-meta里面，
		再通过uiReferComp.js解析u-meta将参照基本信息放到参照前台dom结构data-refmodel里面，到此参照的基本信息初始化完毕；

		如果用的不是ui模板设计器，那么需要使用参照的人自己传refcode然后去调用查询参照信息接口来获取此参照的基本信息，
		而后的初始化过程与上面是一样的。

		一些datatable初始化代码片段如下：
			var app = window.app;
			if (app ===undefined) {
				app = u.createApp();
			}
			//初始化数据
			viewModel.headform.createEmptyRow(); 
			viewModel.headform.setRowSelect(0); 
			
			try{
				app.init(viewModel,null,false);
			}catch(e){
				console.log(e.stack);
			}

		uiReferComp.js初始化参照init方法：
		init: function() {
			element = this.element;
			options = this.options;
			viewModel = this.viewModel;
			//增加锚点
			this.meta_translations = options.meta_translations;
			this.meta_childmeta = options.meta_childmeta;
			if(this.meta_childmeta)
				this.childmetaArr = this.meta_childmeta.split('.');
			this.childIndex = options.childIndex;
			if(this.childIndex)
				this.childIndexArr = this.childIndex.split('-');
			this.hasDataTable = true;
			var self = this;
			var refmodel = '', refparam = '',refcfg = '';
			this.fieldId = '';
			this.showField = this.options['showField'];
			this.validType = 'string';
			if (this.hasDataTable) {
				refcfg = this.dataModel.getMeta(this.field, 'refcfg');
				if(this.meta_childmeta){
					var nowChildMeta = this.dataModel;
					for(var i = 0; i < this.childmetaArr.length; i++){
						nowChildMeta = nowChildMeta.meta[this.childmetaArr[i]];
					}
					refmodel = nowChildMeta.meta[this.field].refmodel;
				}else{
					refmodel = this.dataModel.getMeta(this.field, 'refmodel')
				}
				refparam = this.dataModel.getMeta(this.field, 'refparam');
				var els = $(this.element);
				var $inputed = els.find("input");
				var inputid = $inputed.attr('id');

				this.fieldId = inputid || this.field;
				this.dataModel.refMeta(this.field, 'refparam').subscribe(function(value){
					$(element).attr('data-refparam', value);
				})
				this.dataModel.refMeta(this.field, 'refcfg').subscribe(function(value){
					$(element).attr('data-refcfg', value);
				})
			}
			$(element).attr('data-refmodel', refmodel);
			$(element).attr('data-refcfg', refcfg);
			$(element).attr('data-refparam', refparam);
			var pageUrl = ''; 
			var extendUrl = '/static/js/uiref/refDList.js';
			var rbrace = /^(?:\{[\w\W]*\}|\[[\w\W]*\])$/;
			if(rbrace.test( refcfg )){
				refcfg=jQuery.parseJSON(refcfg);
			}
			if (refcfg && refcfg.pageUrl) {
				pageUrl ="/"+refcfg.pageUrl + extendUrl;
			}
			
			
			var refInitFunc = pageUrl.substr(pageUrl.lastIndexOf('/') +1).replace('.js','');
			
			var contentId = 'refContainer' + this.fieldId;
			var refContainerID=contentId.replace(/[^\w\s]/gi, '\\$&');
			if ($('#' + refContainerID).length > 0)
				$('#' + refContainerID).remove();

				
			if(!window[refInitFunc]){
				var scriptStr = '';	
				$.ajax({
					url:pageUrl,
					dataType:'script',
					async : false,
					cache: true,
					success : function(data){
						scriptStr  = data
					}
				});
				(new Function(scriptStr))();
			}
			
			
			window[refInitFunc]({contentId: contentId,
				dom: $(this.element),
				pageUrl: pageUrl,
				setVal: function(data, noFocus) {
					if (data) {
						var options = $('#' + refContainerID).Refer('getOptions');
						var pk = $.map(data, function(val, index) {
							return val.refpk
						}).join(',');
						var name = $.map(data, function(val, index) {
							return val.refname
						}).join(',');
						var code = $.map(data, function(val, index) {
							return val.refcode
						}).join(',');
						var oldVal = self.trueValue,
						showValue = options.isReturnCode ? code : name,
						valObj = {trueValue:pk,showValue:showValue}
						if (pk == '') {
							if (oldVal != null && oldVal != '') {
								self.setValue(valObj);
							}
						} else {
							self.setValue(valObj);
						}
						if (!noFocus) {
							$(self.element).find("input").focus();
						}
					}
				},
				onOk: function(data) {
					this.setVal(data);
					this.onCancel();
				},
				onCancel: function() {
					$('#' + refContainerID).Refer('hide');
				}
			})
			if(this.meta_childmeta){
				var nowRow = this.dataModel.getCurrentRow();
				if(nowRow && this.childmetaArr && this.childIndexArr && this.childmetaArr.length == this.childIndexArr.length){ 
					for(var i = 0; i < this.childmetaArr.length; i++){
						nowRow = nowRow.getValue(this.childmetaArr[i]).getRow(this.childIndexArr[i]);
					}
					nowRow.ref(this.field).subscribe(function(value){
						self.modelValueChange(value)
					});

					var v = nowRow.getValue(this.field);
					this.modelValueChange(v);
				}
			}else{
				this.dataModel.ref(this.field).subscribe(function(value) {
					self.modelValueChange(value)
				});
				this.modelValueChange(this.dataModel.getValue(this.field))
			}

		},
		

		


#### 2.不基于uui开发参照 ####
- 前端需要引入reflib.js, refer.js, refGrid.js, refGridtree.js, refTree.js,refcommon.js, refCompAdaptor.js,knockout-3.2.0.debug.js, jquery-1.11.2.js，其中refCompAdaptor.js是参照初始化的接口文件。


	使用参照的人员需要传入refcode，然后根据refcode去查询参照基本信息，最后设到
	参照前端的dom上面，代码片段如下：

		define(['jquery','knockout','refer','refGrid','refGridtree','refTree','refcommon'],function($, ko,refer){
			var refComp = {
				initRefComp : function(dom, options) {
				var $input=dom.find('input');
				var refCode = options.refCode;
				var selectedVals = options.selectedVals;
				$.ajax({
					type: "get",
					url: '/uitemplate_web/iref_ctr/refInfo/',
					data: options,
					traditional: true,
					async: false,
					dataType: "json",
					success: function (refmodel) {
						dom.attr('data-refmodel',JSON.stringify(refmodel));
						options = JSON.parse(dom.attr('data-refmodel'));
						options.refCode = refCode;
						options.dom = dom;
						options.contentId = 'refContainer' + dom.attr('id');
						options.sPOPMode=true;
						options.selectedVals = selectedVals;
						
						options.setVal = function(data) {
							 if (data) {
								var  optionsTemp=$('#'+options.contentId).Refer('getOptions');
								var pk = $.map(data,function(val, index) {
										return val.refpk
									}).join(',');
								var name = $.map(data,function(val, index) {
										return val.refname
									}).join(',');
								var code=$.map(data,function(val, index) {
										return val.refcode
									}).join(',');
								$input.val(options.isReturnCode?code:name);
								
								
								var msg = {};
								msg.type = "ok";
								msg.data = data;
								var str = JSON.stringify(msg);
								top.postMessage(str, '*');	
							}

						};
						options.onCancel = function() {
							var msg = {};
							msg.type = "cancel";
							msg.data = data;
							var str = JSON.stringify(msg);
							top.postMessage(str, '*');	
						}
						
						options.refInput = dom.find("input");
						
						if (!options.pageURL) {
							options.pageURL = '/uitemplate_web/static/js/uiref/refDList.js';
						}
						
						
						var	pageURL = options.pageURL;
						
						var refInitFunc = pageURL.substr(pageURL.lastIndexOf('/') +1).replace('.js','');
						if(!window[refInitFunc]){
							var scriptStr = '';	
							$.ajax({
								url:pageURL,
								dataType:'script',
								async : false,
								success : function(data){
									scriptStr  = data
								}
							});
							eval(scriptStr);	
						}
						window[refInitFunc](options);
					}
				});
				
				}
			};
			
			return refComp;
		});


## 参照前端参数说明 ##

###### 参照大概分为树表类型，表格类型，树类型，列表类型四种 ######
###### 具体参数配置有data-refmodel、data-refcfg、data-refparam三种######

	1.data-refmodel参数：

		data-refmodel里面存放参照的基本信息，例如供应商(树表类型)参照需要的基本信息有：    
		data-refmodel="{"refUIType":"RefGridTree","defaultFieldCount":4,
			"strFieldCode":["pk_org","code","name","mnecode","shortname"],
			"strFieldName":["所属组织","供应商编码","供应商名称","助记码","供应商简称"],
			"strHiddenFieldCode":["pk_supplier","pk_supplier_main","pk_supplierclass"],
			"refCodeNamePK":["code","name","pk_supplier"],
			"rootName":"供应商基本分类","refName":"供应商档案"}"
		其中refUIType是参照类型，defaultFieldCount是默认显示列数，strFieldCode是显示列的code值，
		strFieldName是显示列的name值，strHiddenFieldCode是隐藏字段，refCodeNamePK是参照编码|名称|主键，
		rootName是根节点的名称，refName是参照名称，ctrlUrl是不同产品的url接口。

	2.data-refcfg参数：

		初始化配置页面载入时 设置 data-refcfg，data-refcfg里面存放的是参照的配置信息，
		比如参照是否多选的参数可以放在data-refcfg里面，
		此时data-refcfg里面的参数信息会覆盖data-refmodel里面的相同参数信息。
		比如data-refmodel里面有个参照类型refUIType参数为“RefGridTree”，
		那么将refUIType：“RefGrid”写到data-refcfg里面就可以将默认RefGridTree类型变为RefGrid类型，
		具体写法：data-refcfg="{"refUIType":"RefGrid"}"

	3.data-refparam参数：

		data-refparam用于设置联动参数，比如选择某个参照时可以联动另外的某个或多个参照将一个组织的pk_org放到需要联动的这些参照的
		data-refparam参数里面，那么就这个参照查询数据时就可以依赖这个组织的pk_org
		例如：data-refparam="{"pk_org":"0001A310000000000OML"}"

参照具体参数详细列表：

		isEnable:true,
		isBillType: true,//否则是Bill
        ctx: undefined,//请求上下文
        cfg_pk_org: '',
        isOrgRefer: false,//是否是组织参照
        isMaintenanceDocAddEnable: false,//维护档案新增按钮
        isquerytpl:false,//是否是查询模板
        wrapRefer: undefined,//控件wrap
        defaultFieldCount:2,//默认显示字段中的显示字段数----表示显示前几个字段
        isClickToHide: true,//当弹出模式，点击时是否自动关闭
        refInput: undefined,//指定input时，有模糊过滤
        refUIType: 'CommonRef',//参照样式类型
        isClassDoc: false,//档案列表(分类只有一个层级,对应树只有一个可展开节点)
        isZtreeStyle:true,//单选树，ztree样式
        autoCheck: true,//是否自动检查
        focusShowCode: false,//获取焦点后是否显示Code
        refName: undefined,
        refCode: undefined,
        refModelClassName: undefined,//自定义参照模型类
        refModelHandlerClass: undefined,//参照后端业务切入处理类
        refModel: {},//参照配置集合
        selectedVals: [],//多选的值
		values: [],//参照选择结果
        rootName: undefined,//根节点名称
        isRootCheckEnabled: false,//根节点可选
        isReturnCode: false,//input框显示编码
        isNotLeafSelected: true,//非叶子节点是否可选(默认为true 父节点可以选择)
        isMultiSelectedEnabled: false, //是否多选标志,默认单选
        isCheckListEnabled: true, //焦点进入是否显示搜索的数据，默认显示
        refTempl: undefined,//参照容器模板
        hotDataSize: 20,
        pageSize: 50,
        cache: true,
        refModelUrl:{},
        data: [],
        classData: [],//分类数据集

## 参照后台接口 ##

- 参照后台使用rest服务进行调用，所有后台方法入口是WebRefController类，每种参照类型都已经封装好自己的抽象类，后台使用参照的接口方法时只需继承这些抽象类，实现抽象类里面的各种方法即可。

```
  各种参照类型抽象类有:
  1. 树表类型AbstractTreeGridRefModel.java
  2. 表格类型AbstractGridRefModel.java
  3. 树类型AbstractTreeRefModel.java
  4. 普通列表类型AbstractCommonRefModel.java
```

- 参照后台方法列表



<table style="border-collapse:collapse">

	<tr>

		<th>方法名</th>

		<th>参数</th>

		<th>返回值</th>

		<th>功能说明</th>

	</tr>



	<tr>

		<td>getRefModelInfo</td>

		<td>RefViewModelVO refViewModel（对应参照前台设置的参数详细信息）</td>

		<td>RefViewModelVO（参照基础信息的返回对象）</td>

		<td>用来获取参照基础信息</td>

	</tr>

	<tr>

		<td>getCommonRefData</td>

		<td>RefViewModelVO refViewModel（对应参照前台设置的参数详细信息）</td>

		<td>List&ltMap&ltString, String&gt&gt
		（返回参照数据对应的List集合）</td>

		<td>用来返回列表类型和表格类型和树类型的参照数据</td>

	</tr>

	<tr>

		<td>getBlobRefClassData</td>

		<td>RefViewModelVO refViewModel（对应参照前台设置的参数详细信息）</td>

		<td>List&ltMap&ltString, String&gt&gt
		（返回参照数据对应的List集合）</td>

		<td>用来返回树表类型里面树的参照数据</td>

	</tr>

	<tr>

		<td>getBlobRefData</td>

		<td>RefViewModelVO refViewModel（对应参照前台设置的参数详细信息）</td>

		<td>List&ltMap&ltString, String&gt&gt
		（返回参照数据对应的List集合）</td>

		<td>用来返回树表类型里面点击树的节点后获取的参照数据</td>

	</tr>

	<tr>

		<td>filterRefJSON</td>

		<td>RefViewModelVO refViewModel（对应参照前台设置的参数详细信息）</td>

		<td>List&ltMap&ltString, String&gt&gt
		（返回参照数据对应的List集合）</td>

		<td>用来返回参照模糊过滤的数据</td>

	</tr>

	<tr>

		<td>matchPKRefJSON</td>

		<td>RefViewModelVO refViewModel（对应参照前台设置的参数详细信息）</td>

		<td>List&ltMap&ltString, String&gt&gt
		（返回参照数据对应的List集合）</td>

		<td>用来返回参照通过pk过滤校验后的数据,比如常用数据的校验</td>

	</tr>
	<tr>

		<td>matchBlurRefJSON</td>

		<td>RefViewModelVO refViewModel（对应参照前台设置的参数详细信息）</td>

		<td>List&ltMap&ltString, String&gt&gt
		（返回参照数据对应的List集合）</td>

		<td>用来返回参照通过content属性值校验录入的数据,比如对手工录入的参照数据校验</td>

	</tr>
</table>



