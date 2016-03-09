
# Titanium-ListView数据绑定、下拉刷新等常用功能的实现
在使用ListView时，使用Collections和Model可以便捷实现本地数据管理以及与服务器同步数据后的UI更新，下拉刷新也是在开发listview时常见的需求，实现了下拉刷新可以在当前页面直接请求新数据、保存至本地、更新UI。

在开发时，controller常见的处理逻辑为
使用本地数据（一般保存在sqlite中）初始化->请求

## 使用到的组件以及实现的功能
- ListView
- collections and model
- pull to refresh（基于[nl.fokkezb.pullToRefresh][1]）


  [1]: https://github.com/FokkeZB/nl.fokkezb.pullToRefresh