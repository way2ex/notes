---

layout: post

title: "日常笔记"

date: 2017-10-09

categories: [React]

---

* content
{:toc}

## react-router
- ``exact``属性
 ``exact``是``Route``组件的一个属性，用于指示是否使用严格匹配。值为``true``时，使用严格匹配。是否使用严格匹配的区别如下:
 ```js
  <Route path='/' component={Home} />
  <Route path='/page' component={Page}>
  //这种情况下，如果匹配路由path='/page'，那么会把Home也会展示出来。
 ```

## react
- 列表渲染
 只需要返回一个含有组件的数组即可
 ```js
  const routeList = routes.map((route, index) => (
      <Route key={index} path={`${match.url}${route.path}`} component={route.component}></Route>
  ))
 ```

## ant design
### 表单的使用
- 引入需要的组件
 ```js
  import { Form, DatePicker, TimePicker, Button } from 'antd';
  const { FormItem } = Form  // 重新定义变量名
 ```
### 注意事项
``Select``的``defaultValue``是固定不变的，如果异步获取其中的``option``，那么再次``render``的时候，不会更改``defaultValue``，此时应该设置``value``属性来更改默认选中的``opytion``值。


