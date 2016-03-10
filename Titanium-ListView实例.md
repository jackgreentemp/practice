
# Titanium-ListView数据绑定、下拉刷新等常用功能的实现
- 在使用ListView时，使用Collections和Model可以便捷实现本地数据管理以及与服务器同步数据后的UI更新；
- 下拉刷新也是在开发listview时常见的需求，实现了下拉刷新可以在当前页面直接请求新数据、保存至本地、更新UI；
- Listview分页功能也是必需的，如果一次性将所有的数据加载到ListView中，在有较多图片资源时容易导致内存泄露；

## 理清思路
在controller的开发中，常见的代码逻辑为：
- 获取Collections对象；
- 查询本地数据（一般保存在sqlite中），一般是用Collections.fetch()，或者sql查询方式；
- 一旦Collections实例化，自动加载并渲染ListView（这是使用Collections的好处）；
- 请求服务器数据（这一步主要依赖于服务器API，API最好能够提供增量查询，可以每次在请求时将某些app本地已有的数据信息，例如最后一条数据的时间戳等，作为参数传给服务器，服务器返回APP没有的数据，这样可以节省手机流量，否则app这边数据处理压力山大，尤其是在app使用一段时间之后，业务数据变多，如果每次都全量查询，流量会爆）。
- 如果需求是新增数据：将本地没有的数据创建model，add至Collections中，然后save，这样就完成了数据同步和ListView刷新。
- 如果需求是更新数据：已服务器返回的数据部分作为关键字，查询Collections，返回该数据对应的model，然后使用model.set(newDataObj).save()，这样就实现了数据更新和ListView刷新。

## 实例功能
实例包括两个页面，首页面主要控件为ListView，每一个ListItem包含一个Label和一个ImageView，点击Item，跳转到次页面（将model传给此页面），除了上述的Label和ImageView，此页面包括一个按钮，点击该按钮修改model中image地址，保存后关闭次页面，主页面中对应的Item的图片已更新。
+ Navigtor导航
+ ListView 下拉刷新
+ ListView 分页显示
+ ListView 绑定 Collection
+ ListView基于model自动更新

![list](https://github.com/jackgreentemp/practice/blob/master/list.gif)

##步骤
+ 新建工程
+ 在Terminal中输入gittio install nl.fokkezb.pullToRefresh，安装下拉刷新插件;
+ 创建导航栏
  + 修改index.xml
  ``` xml
  <Alloy>
	<Window id="win" class="container" platform="android">
		<Button id="label" onClick="doClick">Home</Button>
	</Window>
	<NavigationWindow id="nav" platform="ios" class="container">
		<Window id="win" class="container">
			<Button id="label" onClick="doClick">Home</Button>
		</Window>
	</NavigationWindow>
  </Alloy>
  ```
  + 修改index.js
  ```javascript
 
    Alloy.Globals.Navigator = {
	
    	open: function(controller, payload){
    		var win = Alloy.createController(controller, payload || {}).getView();
    		
    		if(OS_IOS){
    			$.nav.openWindow(win);
    		}
    		else if(OS_MOBILEWEB){
    			$.nav.open(win);
    		}
    		else {
    			
    			// added this property to the payload to know if the window is a child
    			if (payload.displayHomeAsUp){
    				
    				win.addEventListener('open',function(evt){
    					var activity=win.activity;
    					activity.actionBar.displayHomeAsUp=payload.displayHomeAsUp;
    					activity.actionBar.onHomeIconItemSelected=function(){
    						evt.source.close();
    					};
    				});
    			}
    			win.open();
    		}
    	}
    };
    
    function doClick(e) {
       Alloy.Globals.Navigator.open("detail", {displayHomeAsUp:true});
    }
    if(OS_ANDROID) {
    	$.win.open();
    } else {
    	$.nav.open();
    }

  ```
  + 创建detail.xml
  ``` xml
  <Alloy>
	<Window id="win" class="container" title="index">
		<ActionBar platfor="android" displayHomeAsUp="true" onHomeIconItemSelected="closeWindow" />
	</Window>
  </Alloy>
  ```
  + 创建detail.js
  ``` javascript
    function closeWindow(){
      $.win.close();
    }
  ```
+ 创建ListView
    + 修改index.xml，使用require方式引入list.js
  ``` xml
  <Alloy>
	<Require id="index" src="list" platform="android" /> 
	<NavigationWindow id="nav" platform="ios" class="container">
		<Require src="list"/>
	</NavigationWindow>
  </Alloy>
  ```    
    + 修改inde.js
  ```javascript
    /**
     * Global Navigation Handler
     */
    Alloy.Globals.Navigator = {
    	
    	open: function(controller, payload){
    		var win = Alloy.createController(controller, payload || {}).getView();
    		
    		if(OS_IOS){
    			$.nav.openWindow(win);
    		}
    		else if(OS_MOBILEWEB){
    			$.nav.open(win);
    		}
    		else {
    			
    			// added this property to the payload to know if the window is a child
    			if (payload.displayHomeAsUp){
    				
    				win.addEventListener('open',function(evt){
    					var activity=win.activity;
    					activity.actionBar.displayHomeAsUp=payload.displayHomeAsUp;
    					activity.actionBar.onHomeIconItemSelected=function(){
    						evt.source.close();
    					};
    				});
    			}
    			win.open();
    		}
    	}
    };
    
    function doClick(e) {
       Alloy.Globals.Navigator.open("detail", {displayHomeAsUp:true});
    }
    
    if(OS_ANDROID) {
    	$.index.getView().open();
    } else {
    	$.nav.open();
    }
    
  ```
    + 创建list.xml，标签方式实例化Collection，引入下拉刷新widget
  ``` xml
  <Alloy>
	<Collection src="myCollection"/>
	<Window id="win" class="container">
		<Widget id="ptr" src="nl.fokkezb.pullToRefresh" onRelease="myRefresher">
			<ListView id="list" defaultItemTemplate="defaultItem" onMarker="onMarkerEvent">
				<Templates>
					<ItemTemplate name="defaultItem" height="Titanium.UI.SIZE">
						<View class='v_0'>
							<Label bindId="testDate" class="testDate" />
							<ImageView bindId="webView" class="w_1"></ImageView>
						</View>
					</ItemTemplate>
				</Templates>
				<ListSection id="section" dataCollection="myCollection" dataTransform="doTransform">
					<ListItem class="item" dataid:text="{id}" testDate:text="{testDate}" webView:image="{image}"/>
				</ListSection>
			</ListView>
		</Widget>
	</Window>
  </Alloy>
  ```    
    + 创建list.js，这是本例的核心文件，由于没有服务器接口，所以使用本地创建样例数据，思路就是查本地数据，如果本地数据为空，则新建10条数据；注意ListView的更新都是依赖于Collection的变化；并且使用onMarkerEvent来进行分页显示，具体的方式参见代码；
  ``` javascript
    var collection = Alloy.Collections.myCollection;
    var moment = require('alloy/moment');
    var table = collection.config.adapter.collection_name;
    Ti.API.info("table=", table);
    var uid = 1;//不知道为什么不能是string格式，否则ios query时会出问题
    
    //listview滑动到底部，查询数据，更新listview
    function onMarkerEvent(e) {
    	var oldLength = _.size(collection);
    	Ti.API.info("oldLength = ", oldLength);
    	collection.fetch({query: {statement: 'SELECT * from ' + table + ' where uid = ?' + ' ORDER BY testDate DESC limit 0, ' + (oldLength+10), params: [uid]}});
    	var newLength = _.size(collection);
    	Ti.API.info("newLength = ", newLength);
    	
    	if(newLength == oldLength){//sql中无新数据了
    		addDatasToCollection();//TODO 使用网络请求获取新参数
    	}
    	
    	//更新marker索引，如果是网络请求中的，需要在网络请求callback中更新marker索引
    	$.list.setMarker({
    		sectionIndex:0,
    		itemIndex:$.list.sections[0].items.length - 1
    	});
    }
    
    //整理template中的参数
    function doTransform(model){
    	var obj = model.toJSON();
    	obj.testDate = "id=" + obj.id + ", " + obj.testDate;
    	return obj;
    }
    
    //为Collection新增数据
    function addDatasToCollection(){
    	var i;
    	for(i=1;i<=10;i++){
    		var obj = {
    			uid: 1,
    		    testDate: ''+moment().format('YYYY-MM-DD HH:mm:ss'),
    		    image: 'http://www.photography-match.com/views/images/gallery/Uluru_Kata_Tjuta_National_Park_Australia.jpg',
    		};
    		
    		var model = Alloy.createModel('myCollection', obj);
    		
    		collection.add(model);
    		
    		model.save();
    	}
    }
    
    //初始化
    function init(){
    	//查询数据
    	collection.fetch({query: { statement: 'SELECT * from ' + table + ' where uid = ?' + ' ORDER BY testDate DESC limit 0,10', params: [uid]}});
    	
    	//如果数据库无数据，新增一些数据
    	if(_.size(collection) === 0){
    		addDatasToCollection();
    	}
    	
    	//更新marker索引
        $.list.setMarker({
    		sectionIndex:0,
    		itemIndex:$.list.sections[0].items.length - 1
    	});
    }
    
    //先open window，再执行初始化
    $.win.addEventListener("open", function(){
        init();
    });
    
    //销毁，避免内存溢出
    $.win.addEventListener("close", function(){
        $.destroy();
    });
    
    //下拉刷新执行的函数
    function myRefresher(e) {
    	init();
    	e.hide();
    }
    
    /*
     * listView 点击监听 
     */
    $.list.addEventListener('itemclick', function(e){
    	
    	//取消 listView点击后选中状态
        var item = e.section.getItemAt(e.itemIndex);
        e.section.updateItemAt(e.itemIndex, item);
        
    	var dataid = e.section.items[e.itemIndex].dataid.text;//根据item索引查找dataid
    
    	var data = collection.get(dataid);//根据dataid获取数据模型
    	
    	// Ti.API.info("item data =", data);
    	
    	Alloy.Globals.Navigator.open("detail", {displayHomeAsUp:true, data: data});
    		
    });

  ```
    
    + 创建detail.xml
  ``` xml
  <Alloy>
	<Collection src="myCollection"/>
	<Window id="win" class="container" title="index" layout="vertical">
		<ActionBar platfor="android" displayHomeAsUp="true" onHomeIconItemSelected="closeWindow" />
			<View class='v_0'>
				<Label id="data_id"></Label>
				<Label class="testDate" id="testDate"/>
				<ImageView class="w_1" id="image"></ImageView>
			</View>
			<Button id="btn_edit" onClick="edit" top="50">修改</Button>
	</Window>
  </Alloy>
  ```
    + 创建detail.js，接收list.js传递进来的model，并获取相应的数据赋值给UI，在“修改”按钮的监听中，修改model中image地址，保存model后，关闭detail.js，发现ListView的图片也自动更新了。
  ```javascript
    var args = arguments[0] || {};
    
    function init() {
    	Ti.API.info("args = ", args.data.toJSON());
    	$.data_id.text = "id=" + args.data.toJSON().id;
    	$.testDate.text = args.data.toJSON().testDate;
    	$.image.image = args.data.toJSON().image;
    }
    
    function closeWindow(){
    	$.win.close();
    }
    
    function edit() {
    	var model = args.data;//获取model
    	var obj = model.toJSON();//model转为object
    	obj.image = 'http://image.tianjimedia.com/uploadImages/2013/105/414UWER6711T.jpg';//修改image地址
    	$.image.image = obj.image;//更新UI
    	model.set(obj).save();//更新model并保存，自动刷新list.js的UI
    }
    
    init();    
  ```
+ 创建Collection
  + 新建myCollection.js
  ``` javascript
    exports.definition = {
    	config: {
    		columns: {
    			id: 'INTEGER PRIMARY KEY AUTOINCREMENT',
    			uid: 'TEXT',
    			testDate: 'TEXT',
    			image: 'TEXT',
    		},
    		defaults: {
    		},
    		adapter: {
    			type: 'sql',
    			collection_name: 'myCollection',
    			idAttribute: 'id'
    		}
    	},
    
    	extendCollection : function(Collection) {
    		_.extend(Collection.prototype, {
    
    			// For Backbone v1.1.2, uncomment this to override the fetch method
    			/*
    			fetch: function(options) {
    				options = options ? _.clone(options) : {};
    				options.reset = true;
    				return Backbone.Collection.prototype.fetch.call(this, options);
    			}
    			*/
    			initialize: function () {
                    //*** Default sort field.  Replace with your own default.
                    this.sortField = "testDate";
                    //*** Default sort direction
                    this.sortDirection = "DESC";
                },
                //*** Use setSortField to specify field and direction before calling sort method
                setSortField: function (field, direction) {
                    this.sortField = field;
                    this.sortDirection = direction;
                },
     
                comparator: function(collection) {
                    return collection.get(this.sortField);
                },
     
                //*** Override sortBy to allow sort on any field, either direction 
                sortBy: function (iterator, context) {
                    var obj = this.models;
                    var direction = this.sortDirection;
     
                    return _.pluck(_.map(obj, function (value, index, list) {
                        return {
                            value: value,
                            index: index,
                            criteria: iterator.call(context, value, index, list)
                        };
                    }).sort(function (left, right) {
                        // swap a and b for reverse sort
                        var a = direction === "ASC" ? left.criteria : right.criteria;
                        var b = direction === "ASC" ? right.criteria : left.criteria;
     
                        if (a !== b) {
                            if (a > b || a === void 0) return 1;
                            if (a < b || b === void 0) return -1;
                        }
                        return left.index < right.index ? -1 : 1;
                    }), 'value');
                }
    		});
    
    		return Collection;
    	}
    };
  
  ```

## 使用到的组件以及实现的功能
- ListView
- collections and model
- pull to refresh（基于[nl.fokkezb.pullToRefresh][1]）
- 分页


  [1]: https://github.com/FokkeZB/nl.fokkezb.pullToRefresh