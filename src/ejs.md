# EJS 文档

## 开始使用

### 安装

使用 NPM 轻松安装 EJS。

```bash
$ npm install ejs
```

### 使用

向 EJS 传递模板字符串和一些数据，即可生成 HTML。

```javascript
let ejs = require('ejs');
let people = ['geddy', 'neil', 'alex'];
let html = ejs.render('<%= people.join(", "); %>', {people: people});
```

### 命令行工具

提供模板文件和数据文件，并指定输出文件。

```bash
ejs ./template_file.ejs -f data_file.json -o ./output.html
```

### 浏览器支持

从[最新版本](https://github.com/mde/ejs/releases/latest)下载浏览器构建版本，并在 script 标签中使用。

```markup
<script src="ejs.js"></script>
<script>
  let people = ['geddy', 'neil', 'alex'];
  let html = ejs.render('<%= people.join(", "); %>', {people: people});
</script>
```

## 文档

### 示例

```markup
<% if (user) { %>
  <h2><%= user.name %></h2>
<% } %>
```

### 用法

```javascript
let template = ejs.compile(str, options);
template(data);
// => 渲染后的 HTML 字符串

ejs.render(str, data, options);
// => 渲染后的 HTML 字符串

ejs.renderFile(filename, data, options, function(err, str){
    // str => 渲染后的 HTML 字符串
});
```

### 选项

- `cache` 编译的函数会被缓存，需要提供 `filename`
- `filename` 被 `cache` 用于缓存键值，也用于包含文件
- `root` 设置项目的根目录，用于通过绝对路径包含文件（例如 /file.ejs）。可以是数组，以尝试从多个目录解析包含文件。
- `views` 一个路径数组，用于在解析相对路径的包含文件时使用。
- `context` 函数执行上下文
- `compileDebug` 当为 `false` 时，不编译调试代码
- `client` 返回独立的编译函数
- `delimiter` 用于内部分隔符的字符，默认为 '%'
- `openDelimiter` 用于开始分隔符的字符，默认为 '<'
- `closeDelimiter` 用于结束分隔符的字符，默认为 '>'
- `debug` 输出生成的函数体
- `strict` 当设置为 `true` 时，生成的函数处于严格模式
- `_with` 是否使用 `with() {}` 结构。如果为 `false`，则局部变量将存储在 `locals` 对象中。（意味着 `--strict`）
- `localsName` 当不使用 `with` 时，用于存储局部变量的对象名称，默认为 `locals`
- `rmWhitespace` 移除所有可安全移除的空白字符，包括前导和尾随空白。它还对所有小脚本标签启用更安全版本的 `-%>` 行吞噬模式（不会移除行中标签的换行符）。
- `escape` 与 `<%=` 结构一起使用的转义函数。它在渲染时使用，并在生成客户端函数时被 `.toString()`。（默认转义 XML）。
- `outputFunctionName` 设置为一个字符串（例如 `'echo'` 或 `'print'`），作为在小脚本标签内部打印输出的函数。
- `async` 当为 `true` 时，EJS 将使用异步函数进行渲染。（依赖于 JS 运行时的 async/await 支持）。

### 标签

- `<%` '小脚本' 标签，用于控制流，无输出
- `<%_` '空白吞噬' 小脚本标签，移除其前的所有空白
- `<%=` 将值输出到模板中（HTML 转义）
- `<%-` 将未转义的值输出到模板中
- `<%#` 注释标签，不执行，无输出
- `<%%` 输出字面量 '<%'
- `%>` 普通结束标签
- `-%>` 修剪模式（'换行吞噬'）标签，修剪后续换行
- `_%>` '空白吞噬' 结束标签，移除其后的所有空白

### 包含文件

包含文件相对于调用 `include` 的模板。（这需要 'filename' 选项。）例如，如果你有 "./views/users.ejs" 和 "./views/user/show.ejs"，你可以使用 `<%- include('user/show'); %>`。

你可能希望使用原始输出标签（`<%-`）与包含文件一起使用，以避免对 HTML 输出进行双重转义。

```markup
<ul>
  <% users.forEach(function(user){ %>
    <%- include('user/show', {user: user}); %>
  <% }); %>
</ul>
```

### 命令行工具

EJS 附带了一个功能齐全的命令行界面。选项与 JavaScript 代码中使用的类似：

- `cache` 编译的函数会被缓存，需要提供 `filename`
- `-o / --output-file FILE` 将渲染的输出写入 FILE 而不是标准输出。
- `-f / --data-file FILE` 必须是 JSON 格式。使用从 FILE 解析的输入作为渲染数据。
- `-i / --data-input STRING` 必须是 JSON 格式且经过 URI 编码。使用从 STRING 解析的输入作为渲染数据。
- `-m / --delimiter CHARACTER` 使用 CHARACTER 与尖括号一起作为开始/结束分隔符（默认为 %）。
- `-p / --open-delimiter CHARACTER` 使用 CHARACTER 代替左尖括号来打开。
- `-c / --close-delimiter CHARACTER` 使用 CHARACTER 代替右尖括号来关闭。
- `-s / --strict` 当设置为 `true` 时，生成的函数处于严格模式
- `-n / --no-with` 使用 'locals' 对象存储变量而不是使用 `with`（意味着 --strict）。
- `-l / --locals-name` 当不使用 `with` 时，用于存储局部变量的对象名称。
- `-w / --rm-whitespace` 移除所有可安全移除的空白字符，包括前导和尾随空白。
- `-d / --debug` 输出生成的函数体
- `-h / --help` 显示此帮助消息。
- `-V/v / --version` 显示 EJS 版本。

一些使用示例：

```bash
$ ejs -p [ -c ] ./template_file.ejs -o ./output.html
$ ejs ./test/fixtures/user.ejs name=Lerxst
$ ejs -n -l _ ./some_template.ejs -f ./data_file.json
```

### 自定义分隔符

可以基于每个模板或全局应用自定义分隔符：

```javascript
let ejs = require('ejs'),
    users = ['geddy', 'neil', 'alex'];

// 仅一个模板
ejs.render('<?= users.join(" | "); ?>', {users: users},
    {delimiter: '?'});
// => 'geddy | neil | alex'

// 或全局
ejs.delimiter = '$';
ejs.render('<$= users.join(" | "); $>', {users: users});
// => 'geddy | neil | alex'
```

### 缓存

EJS 附带了一个基本的内存缓存，用于缓存用于渲染模板的中间 JavaScript 函数。使用 Node 的 `lru-cache` 库可以轻松插入 LRU 缓存：

```javascript
let ejs = require('ejs'),
    LRU = require('lru-cache');
ejs.cache = LRU(100); // 100 项限制的 LRU 缓存
```

如果你想清除 EJS 缓存，调用 `ejs.clearCache`。如果你正在使用 LRU 缓存并且需要不同的限制，只需将 `ejs.cache` 重置为 LRU 的新实例。

### 自定义文件加载器

默认的文件加载器是 `fs.readFileSync`，如果你想自定义它，可以设置 `ejs.fileLoader`。

```javascript
let ejs = require('ejs');
let myFileLoader = function (filePath) {
  return 'myFileLoader: ' + fs.readFileSync(filePath);
};

ejs.fileLoader = myFileLoader;
```

通过此功能，你可以在读取模板之前对其进行预处理。

### 布局

EJS 不专门支持块，但可以通过包含页眉和页脚来实现布局，如下所示：

```javascript
<%- include('header'); -%>
<h1>
  标题
</h1>
<p>
  我的页面
</p>
<%- include('footer'); -%>
```

### 客户端支持

转到[最新版本](https://github.com/mde/ejs/releases/latest)，下载 `./ejs.js` 或 `./ejs.min.js`。或者，你可以通过克隆仓库并运行 `jake build`（或者如果 jake 没有全局安装，则运行 `$(npm bin)/jake build`）来自行编译。

在你的页面上包含其中一个文件，`ejs` 应该在全局可用。

#### 示例

```javascript
<div id="output"></div>
<script src="ejs.min.js"></script>
<script>
  let people = ['geddy', 'neil', 'alex'],
      html = ejs.render('<%= people.join(", "); %>', {people: people});
  // 使用 jQuery:
  $('#output').html(html);
  // 原生 JS:
  document.getElementById('output').innerHTML = html;
</script>
```

#### 注意事项

EJS 的大部分功能将按预期工作；但是，有几点需要注意：

1. 显然，由于你无法访问文件系统，`ejs.renderFile` 将无法工作。
2. 出于同样的原因，包含文件将无法工作，除非你使用包含回调。这是一个示例：

```javascript
let str = "Hello <%= include('file', {person: 'John'}); %>",
      fn = ejs.compile(str, {client: true});

fn(data, null, function(path, d){ // 包含回调
  // path -> 'file'
  // d -> {person: 'John'}
  // 在此处放置你的代码
  // 以字符串形式返回文件内容
}); // 返回渲染后的字符串
```

### 在 Express 中使用 EJS

[此 GitHub Wiki 页面](https://github.com/mde/ejs/wiki/Using-EJS-with-Express)解释了向 Express 传递 EJS 选项的各种方法。