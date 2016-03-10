
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
+ 创建Model、Collection

## 使用到的组件以及实现的功能
- ListView
- collections and model
- pull to refresh（基于[nl.fokkezb.pullToRefresh][1]）
- 分页


  [1]: https://github.com/FokkeZB/nl.fokkezb.pullToRefresh