# 语法速查表

## 条件语句

```js
<% if (it.someval === "someothervalue") { %>
显示这个！
<% } else { %>
它们不相等
<% } %>
```

## 遍历数组

```js
<% users.forEach(function(user){ %>
  <%= user.first %> <%= user.last %>
<% }) %>
```

## 遍历对象

```js
<% Object.keys(someObject).forEach(function(prop) { %>
  <%= someObject[prop] %>
<% }) %>
```

## 输出到控制台

```js
<% console.log("it.num 的值是：" + it.num) %>
```

## 异步局部模板

```js
<%~ await includeAsync("./path-to-partial") %>
```