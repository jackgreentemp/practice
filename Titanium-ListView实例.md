
# Titanium-ListView数据绑定、下拉刷新等常用功能的实现
- 在使用ListView时，使用Collections和Model可以便捷实现本地数据管理以及与服务器同步数据后的UI更新；
- 下拉刷新也是在开发listview时常见的需求，实现了下拉刷新可以在当前页面直接请求新数据、保存至本地、更新UI；
- Listview分页功能也是必需的，如果一次性将所有的数据加载到ListView中，在有较多图片资源时容易导致内存泄露；

## 理清思路
在controller的开发中，常见的代码逻辑为：
- 获取Collections对象；
- 查询本地数据（一般保存在sqlite中），一般是用Collections.fetch()，或者sql查询方式；
- 一旦Collections实例化，自动加载并渲染ListView（这是使用Collections的好处）；
- 请求服务器数据（这一步主要依赖于服务器API，API最好能够提供增量查询，可以每次在请求时将某些app本地已有的数据信息，例如最后一条数据的时间戳等，作为参数传给服务器，服务器返回APP没有的数据，这样可以节省手机流量，否则app这边数据处理压力山大，尤其是在app使用一段时间之后，业务数据变多，如果每次都全量查询，流量会爆），将本地没有的数据创建model，add至Collections中，然后save，这样就完成了数据同步和ListView刷新。

## 使用到的组件以及实现的功能
- ListView
- collections and model
- pull to refresh（基于[nl.fokkezb.pullToRefresh][1]）
- 分页


  [1]: https://github.com/FokkeZB/nl.fokkezb.pullToRefresh