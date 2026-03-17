# API 概述

## 设置 Eta

Eta 以类的形式导出，因此在使用前必须先实例化：

```js
import { Eta } from "eta"
const eta = new Eta(options)
```

在 Node ESM 中，使用 `import.meta.dirname` 解析路径（Node 20.11+）：

```js
import path from "node:path"

const eta = new Eta({ views: path.join(import.meta.dirname, "templates") })
```

注意：`import.meta.dirname` 需要 Node 20.11+ 版本。

传入选项是可选的。你可以在[这里](api/configuration)找到所有选项的列表。大多数用户需要传入 `views` 选项，这是指向模板目录的路径。

```js
const eta = new Eta({ views: path.join(import.meta.dirname, "templates") })
```

其他常用选项包括：

* `debug`：启用运行时错误的漂亮打印。默认为 `false`。
* `cache`：是否缓存模板。默认为 `false`。
* `autoEscape`：是否自动转义模板中的 HTML。默认为 `true`。

## 渲染模板文件

### 同步渲染

要渲染模板，使用 `render` 方法：

```js
const res = eta.render("templateName", { name: "Ben" })
```

第一个参数是模板名称，第二个参数是传递给模板的数据。模板名称相对于实例化 Eta 时传入的 `views` 选项。

如果你想使用命名模板而不从文件系统解析，请为模板名称添加 `@` 前缀。Eta 将不会尝试从文件系统解析这些模板，而是会在缓存中查找它们。

## 在浏览器中使用 Eta

导入浏览器友好的核心构建：

```html
<script type="module">
  import { Eta } from "eta/core"
  const eta = new Eta()
  document.body.innerHTML = eta.renderString("Hello <%= it.name %>", { name: "Ben" })
</script>
```

### 异步渲染

要异步渲染模板，使用 `renderAsync` 方法：

```js
const res = await eta.renderAsync("templateName", { name: "Ben" })
```

`renderAsync` 方法返回一个 Promise，因此你必须使用 `await` 或 `.then` 来获取结果。

## 渲染字符串

你可以使用 `renderString` 方法将字符串作为模板渲染：

```js
const res = eta.renderString("Hello <%= it.name %>", { name: "Ben" })
```

或者使用 `renderStringAsync` 方法异步渲染字符串：

```js
const res = eta.renderStringAsync("Hello <%= await it.someFunction() %>", {
  someFunction: () => Promise.resolve("Ben")
})
```

## 以编程方式定义模板

要以编程方式定义模板，使用 `loadTemplate`：

```js
const headerPartial = `
  <header>
    <h1><%= it.title %></h1>
  </header>
`

eta.loadTemplate("@header", headerPartial)
```

如果你的模板不是视图目录中的文件，必须为其名称添加 `@` 前缀，这样 Eta 就知道不要从文件系统解析它。

`loadTemplate` 的第三个参数是一个 `{async: boolean}` 类型的对象，用于描述模板是否是异步的。默认情况下，Eta 会假定模板是同步的。

## 常见使用场景

### 自定义标签

你可以使用 `tags` 选项更改 Eta 的默认标签：

```js
const eta = new Eta({ tags: ["{{", "}}"] })
```

### 自动过滤数据

你可以通过将所有值传递给你自己的过滤函数来自动过滤它们：

```js
const eta = new Eta({
  autoFilter: true,
  filterFunction: (val) => {
    if (typeof val === "string") {
      return val.toUpperCase()
    }
    return val
  }
})
```

### 摆脱 `it` 变量

默认情况下，Eta 会将所有数据存储在 `it` 变量中。你可以使用 `varName` 选项自定义此变量的名称：

```js
const eta = new Eta({ varName: "data" })

// "Hi <%= data.name %>"
```

如果你想完全摆脱 `it`，可以使用 `useWith` 选项：

```js
const eta = new Eta({ useWith: true })
// "Hi <%= name %>"
```

这通常被认为是不良实践，因为它可能导致命名冲突/性能下降。

更好的方法是使用 `functionHeader` 配置选项：

```js
const eta = new Eta({
  functionHeader: "const name=it.name, age=it.age"
})
// "Hi <%= name %>, our records show you are <%= age %> years old"
```

或者如果你想要类似于 `useWith` 的行为但具有更好的性能（仍然存在命名冲突），可以这样做：

```js
const eta = new Eta({
  functionHeader: "Object.entries({...it}||{}).forEach(([k,v])=>globalThis[k]=v)"
})
// "Hi <%= name %>"
```

### 自定义文件处理

你可以通过扩展 Eta 类并重写 `readFile` 和 `resolvePath` 方法来自定义 Eta 读取文件的方式：

```js
class CustomEta extends Eta {
  readFile = function (...) {...}

  resolvePath = function (...) {...}
}
```
