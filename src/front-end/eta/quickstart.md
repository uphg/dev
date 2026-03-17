# 快速开始

安装 Eta

```bash
npm install eta
```

在你的项目根目录中，创建 `templates/simple.eta`

```js
Hi <%= it.name %>!
```

然后，在你的 JS 文件中：

```js
import { Eta } from "eta"
import path from "node:path"

const eta = new Eta({ views: path.join(import.meta.dirname, "templates") })

// 渲染模板
const res = eta.render("./simple", { name: "Ben" })
console.log(res) // Hi Ben!
```

注意：`import.meta.dirname` 需要 Node 20.11+ 版本。

Eta v4 仅支持 ESM 模块。在浏览器中，导入核心构建：

```html
<script type="module">
  import { Eta } from "eta/core"
  const eta = new Eta()
  document.body.innerHTML = eta.renderString("Hi <%= it.name %>!", { name: "Ben" })
</script>
```