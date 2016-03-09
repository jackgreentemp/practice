
# Titanium-ListView数据绑定、下拉刷新等常用功能的实现
- 在使用ListView时，使用Collections和Model可以便捷实现本地数据管理以及与服务器同步数据后的UI更新；
- 下拉刷新也是在开发listview时常见的需求，实现了下拉刷新可以在当前页面直接请求新数据、保存至本地、更新UI；
- Listview分页功能也是必需的，如果一次性将所有的数据加载到ListView中，在有较多图片资源时容易导致内存泄露；

在开发时，controller常见的处理逻辑为
1. 使用本地数据（一般保存在sqlite中）初始化页面；
2. 请求服务器数据（这个一般看服务器API如何设计，最好是能够增量查询，每次在请求时将本地数据信息，例如最后一条数据的时间戳等，作为参数，服务器返回

## 使用到的组件以及实现的功能
- ListView
- collections and model
- pull to refresh（基于[nl.fokkezb.pullToRefresh][1]）
- 分页


  [1]: https://github.com/FokkeZB/nl.fokkezb.pullToRefresh