# simple-mind-map 库源码分析文档

## 目录
1. [目录结构](#1-目录结构)
2. [所有布局类型](#2-所有布局类型)
3. [基本创建方式](#3-基本创建方式)
4. [布局方向（水平 vs 垂直）](#4-布局方向水平-vs-垂直)
5. [只读模式 / 禁用编辑](#5-只读模式--禁用编辑)
6. [基于层级/节点的布局自定义](#6-基于层级节点的布局自定义)
7. [完整配置选项参考](#7-完整配置选项参考)
8. [公共 API 方法](#8-公共-api-方法)
9. [关键文件路径参考](#9-关键文件路径参考)

---

## 1. 目录结构

```
example/mind-map/
├── .git/
├── .gitignore
├── assets/
│   ├── client/          (12张PNG截图: client1-6.png, clienten1-6.png)
│   └── ob/              (10张PNG截图: ob1-5.png, oben1-5.png)
├── copy.js
├── dist/                (构建后的Web应用: js/, img/, fonts/)
├── Dockerfile
├── index.html
├── LICENSE              (MIT许可证)
├── nginx.conf
├── README.md            (主README，中文)
├── README_EN.md         (英文README)
├── README_MORE_ZH.md    (中文扩展文档)
├── README_MORE_EN.md    (英文扩展文档)
├── simple-mind-map/     (核心库 - v0.14.0-fix.2)
│   ├── .eslintrc.js
│   ├── .prettierrc / .prettierignore
│   ├── bin/             (createPluginsTypeFiles.js, wsServer.mjs)
│   ├── example/         (exampleData.js, exportFullData.json)
│   ├── full.js          (包含所有插件的完整导入)
│   ├── index.js         (主入口文件 - MindMap类)
│   ├── package.json
│   ├── README.md        (精简版 - 指向GitHub)
│   ├── scripts/         (walkJsFiles.js)
│   └── src/
│       ├── constants/
│       │   ├── constant.js          (布局名称、方向常量、形状类型)
│       │   └── defaultOptions.js    (所有配置选项，约523行)
│       ├── core/
│       │   ├── command/Command.js   (历史记录、撤销/重做)
│       │   ├── command/KeyCommand.js
│       │   ├── event/Event.js
│       │   ├── render/Render.js     (主渲染器、布局切换、命令)
│       │   ├── render/TextEdit.js
│       │   ├── render/node/MindMapNode.js  (节点行为、只读检查)
│       │   ├── render/node/Shape.js
│       │   ├── render/node/Style.js
│       │   └── view/View.js
│       ├── layouts/
│       │   ├── Base.js              (所有布局的抽象基类)
│       │   ├── LogicalStructure.js   (水平树：向右或向左)
│       │   ├── MindMap.js           (双向：左右分支)
│       │   ├── CatalogOrganization.js
│       │   ├── OrganizationStructure.js (垂直自上而下的层级)
│       │   ├── Timeline.js          (水平时间轴)
│       │   ├── VerticalTimeline.js   (垂直时间轴)
│       │   ├── Fishbone.js          (鱼骨图)
│       │   └── fishboneUtils.js
│       ├── plugins/                 (19+个插件)
│       │   ├── MindMapLayoutPro.js  (**逐节点方向控制**)
│       │   ├── Drag.js, Select.js, Export.js, ...
│       ├── parse/                   (xmind.js, markdown.js, markdownTo.js, toTxt.js)
│       ├── theme/                   (default.js, index.js)
│       ├── svg/                     (btns.js, icons.js)
│       └── utils/                   (各种工具函数)
└── web/                 (Vue2演示应用)
    ├── package.json
    ├── babel.config.js / vue.config.js
    ├── .env.library
    ├── src/
    │   ├── main.js
    │   ├── router.js
    │   ├── store.js
    │   └── pages/Edit/components/
    │       ├── Edit.vue               (**创建MindMap实例**)
    │       ├── NavigatorToolbar.vue    (**只读切换UI**)
    │       ├── Structure.vue           (**布局切换UI**)
    │       └── ...
    └── public/
```

### 构造时选项

创建 MindMap 实例时的基本配置：

```javascript
const mindMap = new MindMap({
  el: document.getElementById('container'),
  data: mindMapData,
  readonly: false,  // 默认为 false（可编辑）
  layout: 'logicalStructure'
});
```

### 演示应用如何切换只读模式

演示应用使用 Vuex 状态管理来控制只读模式：

```javascript
// NavigatorToolbar.vue (第179-182行)
readonlyChange() {
  this.setIsReadonly(!this.isReadonly)           // Vuex状态
  this.mindMap.setMode(this.isReadonly ? 'readonly' : 'edit')  // MindMap API
}

// store.js (第26行)
isReadonly: false   // 默认Vuex状态
```

### 演示应用的真实创建示例

```javascript
// Edit.vue - 创建带完整配置的MindMap实例
const mindMap = new MindMap({
  el: this.$refs.mindMapContainer,
  data: this.mindMapData,
  layout: this.currentLayout,
  theme: this.currentTheme,
  readonly: this.isReadonly,
  // ... 其他选项
});
```

---

## 2. 所有布局类型

### 生长方向常量（来自 constant.js）

`LAYOUT_GROW_DIR` 常量定义节点的扩展方向：

| 常量 | 值 | 描述 |
|------|-----|------|
| `RIGHT` | 'right' | 向右生长 |
| `LEFT` | 'left' | 向左生长 |
| `DOWN` | 'down' | 向下生长 |
| `UP` | 'up' | 向上生长 |

### 布局到类的映射（Render.js 第46-71行）

```javascript
const layouts = {
  [CONSTANTS.LAYOUT.LOGICAL_STRUCTURE]: LogicalStructure,
  [CONSTANTS.LAYOUT.LOGICAL_STRUCTURE_LEFT]: LogicalStructure,  // 同一个类，isUseLeft=true
  [CONSTANTS.LAYOUT.MIND_MAP]: MindMap,                        // 独立类
  [CONSTANTS.LAYOUT.CATALOG_ORGANIZATION]: CatalogOrganization,
  [CONSTANTS.LAYOUT.ORGANIZATION_STRUCTURE]: OrganizationStructure,
  [CONSTANTS.LAYOUT.TIMELINE]: Timeline,
  [CONSTANTS.LAYOUT.TIMELINE2]: Timeline,
  [CONSTANTS.LAYOUT.VERTICAL_TIMELINE]: VerticalTimeline,
  [CONSTANTS.LAYOUT.VERTICAL_TIMELINE2]: VerticalTimeline,    // 同一个类，方向逻辑不同
  [CONSTANTS.LAYOUT.VERTICAL_TIMELINE3]: VerticalTimeline,
  [CONSTANTS.LAYOUT.FISHBONE]: Fishbone,
  [CONSTANTS.LAYOUT.FISHBONE2]: Fishbone
}
```

---

## 3. 基本创建方式

### 来自 README_MORE_ZH.md 和 index.js

```javascript
import MindMap from 'simple-mind-map'

const mindMap = new MindMap({
  el: document.getElementById('mindMapContainer'),
  data: {
    data: {
      text: '根节点'
    },
    children: []
  }
})
```

---

## 4. 布局方向（水平 vs 垂直）

### 布局汇总表

| 布局名称 | 方向 | 描述 |
|----------|------|------|
| `logicalStructure` | 水平（向右） | 单棵树向右生长 |
| `logicalStructureLeft` | 水平（向左） | 单棵树向左生长 |
| `mindMap` | 水平（双向） | 分支交替左右 |
| `organizationStructure` | 垂直（向下） | 组织结构图，自上而下 |
| `catalogOrganization` | 垂直（向下） | 目录树风格 |
| `timeline` | 水平 | 时间轴，向下生长 |
| `timeline2` | 水平（交替） | 时间轴，上下交替 |
| `verticalTimeline` | 垂直 | 时间轴，左右交替 |
| `verticalTimeline2` | 垂直（全左） | 所有分支向左 |
| `verticalTimeline3` | 垂直（全右） | 所有分支向右 |
| `fishbone` / `fishbone2` | 水平（交替） | 鱼骨图 |

### 编程方式切换布局

```javascript
// 获取当前布局
const currentLayout = mindMap.getLayout()

// 设置新布局
mindMap.setLayout('mindMap')

// 带可选参数 notRender（阻止重新渲染）
mindMap.setLayout('logicalStructure', true)
```

### 演示应用如何切换布局

```javascript
// Structure.vue (第79-85行)
useLayout(layout) {
  this.layout = layout
  this.mindMap.setLayout(layout)
}
```

**事件：** 布局变化时触发 `layout_change` 事件。

---

## 5. 只读模式 / 禁用编辑

### 默认配置

```javascript
// defaultOptions.js (第13行)
readonly: false   // 默认为 false（可编辑）
```

### 编程方式设置只读模式

```javascript
// index.js (第541-562行)
mindMap.setMode('readonly')   // 启用只读模式
mindMap.setMode('edit')       // 切换回编辑模式
```

### `setMode()` 的作用：

- 如果打开则隐藏文本编辑框
- 清除当前激活的节点
- 阻止历史操作（`addHistory`、`back`、`forward`）
- 触发 `mode_change` 事件，参数为 `'readonly'` 或 `'edit'`

### 只读模式的影响范围

`readonly` 标志在代码库的多处被检查：

| 文件 | 行号 | 被阻止的操作 |
|------|------|-------------|
| Command.js | 106 | 历史记录 |
| Command.js | 137, 157 | 撤销/重做操作 |
| MindMapNode.js | 364, 378, 395, 422, 435, 461 | 节点交互 |
| Render.js | 708 | 全选 |
| Drag.js | 89, 107 | 拖拽操作 |
| NodeImgAdjust.js | 66 | 图片调整 |
| Painter.js | 26 | 格式刷 |
| Select.js | 45, 68, 123 | 节点选择 |
| Search.js | 97, 197, 210 | 搜索替换 |

---

## 6. 基于层级/节点的布局自定义

### 逐节点方向控制（`dir` 属性）

每个节点可以在其数据中设置 `dir` 属性来控制子节点的生长方向。

**示例：MindMap 布局（MindMap.js 第57行）**

```javascript
newNode.dir = newNode.getData('dir') ||
  (index % 2 === 0
    ? CONSTANTS.LAYOUT_GROW_DIR.RIGHT
    : CONSTANTS.LAYOUT_GROW_DIR.LEFT)
```

**示例：VerticalTimeline 布局（VerticalTimeline.js 第53-65行）**

```javascript
if (this.layout === CONSTANTS.LAYOUT.VERTICAL_TIMELINE2) {
  newNode.dir = CONSTANTS.LAYOUT_GROW_DIR.LEFT   // 全部向左
} else if (this.layout === CONSTANTS.LAYOUT.VERTICAL_TIMELINE3) {
  newNode.dir = CONSTANTS.LAYOUT_GROW_DIR.RIGHT  // 全部向右
} else {
  // 基于索引交替
  newNode.dir = index % 2 === 0
    ? CONSTANTS.LAYOUT_GROW_DIR.RIGHT
    : CONSTANTS.LAYOUT_GROW_DIR.LEFT
}
```

### 重要：子节点继承父节点方向

```javascript
// MindMap.js 第52-53行
if (parent._node.dir) {
  newNode.dir = parent._node.dir   // 子节点继承父节点方向
}
```

### 示例：强制节点方向

```javascript
// 强制某个节点及其所有后代只向右生长
{
  data: {
    text: '分组 A',
    dir: 'right'   // CONSTANTS.LAYOUT_GROW_DIR.RIGHT
  },
  children: [...]
}
```

### MindMapLayoutPro 插件（自动方向分配）

文件：`src/plugins/MindMapLayoutPro.js`

该插件在 `mindMap` 布局中自动为第二级节点分配 `dir` 属性，以平衡左右分布：

```javascript
// 第84-97行：围绕中心平衡子节点
updateNodeTree(tree) {
  const childrenLength = root.children.length
  const center = Math.ceil(childrenLength / 2)
  root.children.forEach((item, index) => {
    if (index + 1 <= center) {
      item.data.dir = CONSTANTS.LAYOUT_GROW_DIR.RIGHT  // 前半部分向右
    } else {
      item.data.dir = CONSTANTS.LAYOUT_GROW_DIR.LEFT   // 后半部分向左
    }
  })
}
```

**插件监听的事件：**
- `layout_change` - 布局变化时重新平衡
- `afterExecCommand` - 插入/删除/移动操作后重新平衡
- `before_update_data` / `before_set_data` - 数据导入时重新平衡

### 用于自定义定位的节点数据属性

来自 constant.js 中的 `nodeDataNoStylePropList`（第174-210行）：

```javascript
// 每个节点可以拥有的非样式数据属性：
'text', 'image', 'imageTitle', 'imageSize', 'icon', 'tag',
'hyperlink', 'hyperlinkTitle', 'note', 'expand', 'isActive',
'generalization', 'richText', 'resetRichText', 'uid', 'activeStyle',
'associativeLineTargets', 'associativeLineTargetControlOffsets',
'associativeLinePoint', 'associativeLineText', 'attachmentUrl',
'attachmentName', 'notation', 'outerFrame', 'number', 'range',
'customLeft', 'customTop',    // <-- 自定义定位
'customTextWidth',
'checkbox', 'dir',            // <-- 方向控制
'needUpdate', 'imgMap', 'nodeLink'
```

> **注意：** `customLeft` 和 `customTop` 是由拖拽操作设置的内部字段，用于手动定位。

---

## 7. 完整配置选项参考

完整选项文档：`src/constants/defaultOptions.js`（523行）

### 基础选项（第5-19行）

| 选项 | 类型 | 默认值 | 描述 |
|------|------|--------|------|
| `el` | DOM元素 | `null` | **必需。** 容器元素 |
| `data` | 对象 | `null` | 思维导图树形数据 |
| `readonly` | 布尔值 | `false` | 只读模式 |
| `layout` | 字符串 | `'logicalStructure'` | 布局类型 |
| `theme` | 字符串 | `'default'` | 主题名称 |
| `themeConfig` | 对象 | `{}` | 主题覆盖配置 |

### 视图选项（第20-35行）

| 选项 | 类型 | 默认值 | 描述 |
|------|------|--------|------|
| `scaleRatio` | 数字 | `0.2` | 缩放步长 |
| `translateRatio` | 数字 | `1` | 平移步长 |
| `minZoomRatio` | 数字 | `20` | 最小缩放百分比 |
| `maxZoomRatio` | 数字 | `400` | 最大缩放百分比 |
| `fit` | 布尔值 | `false` | 初始化时自动适应 |
| `initRootNodePosition` | 数组 | `['center','center']` | 根节点位置 |

### 节点选项（第40-80行）

| 选项 | 类型 | 默认值 | 描述 |
|------|------|--------|------|
| `maxTag` | 数字 | `5` | 最多显示的标签数 |
| `expandBtnSize` | 数字 | `20` | 展开按钮大小 |
| `alwaysShowExpandBtn` | 布尔值 | `false` | 始终显示展开按钮 |
| `notShowExpandBtn` | 布尔值 | `false` | 从不显示展开按钮 |
| `textAutoWrapWidth` | 数字 | `500` | 自动换行阈值（像素） |

### 交互选项（第83-130行）

| 选项 | 类型 | 默认值 | 描述 |
|------|------|--------|------|
| `enableCtrlKeyNodeSelection` | 布尔值 | `true` | Ctrl+点击多选 |
| `useLeftKeySelectionRightKeyDrag` | 布尔值 | `false` | 交换鼠标按钮功能 |
| `isDisableDrag` | 布尔值 | `false` | 禁用画布平移 |
| `disableMouseWheelZoom` | 布尔值 | `false` | 禁用滚轮缩放 |
| `mousewheelAction` | 字符串 | `'move'` | `'zoom'` 或 `'move'` |
| `enableAutoEnterTextEditWhenKeydown` | 布尔值 | `false` | 按键时自动进入编辑 |
| `disabledClipboard` | 布尔值 | `false` | 禁用剪贴板 |

---

## 8. 公共 API 方法

### 实例创建

```javascript
new MindMap(opt)               // 构造函数
```

### 数据管理

```javascript
mindMap.setData(data)          // 替换所有节点数据
mindMap.updateData(data)       // 从修改后的数据更新
mindMap.setFullData({root, layout, theme, view})  // 设置全部数据
mindMap.getData(withConfig)    // 获取数据（可选择是否包含配置）
```

### 布局

```javascript
mindMap.getLayout()            // 获取当前布局名称
mindMap.setLayout(layout, notRender)  // 设置布局
```

**事件：** `'layout_change'`

### 主题

```javascript
mindMap.setTheme(theme, notRender)
mindMap.getTheme()
mindMap.setThemeConfig(config, notRender)
mindMap.getCustomThemeConfig()
mindMap.getThemeConfig(prop)
```

### 只读 / 编辑模式

```javascript
mindMap.setMode('readonly' | 'edit')  // 切换只读模式
```

**事件：** `'mode_change'`

### 视图

```javascript
mindMap.view.fit()             // 适应容器
mindMap.view.narrow()          // 缩小
mindMap.view.enlarge()         // 放大
mindMap.view.reset()           // 重置缩放/位置
mindMap.view.moveNodeToCenter(node)
```

### 配置

```javascript
mindMap.getConfig(prop)        // 获取配置值
mindMap.updateConfig(opt)      // 合并新配置
```

### 命令

```javascript
mindMap.execCommand(...args)   // 执行已注册的命令
mindMap.command.addHistory()   // 手动添加历史记录
mindMap.command.clearHistory()
mindMap.command.pause()
mindMap.command.recovery()
```

### 事件

```javascript
mindMap.on(event, fn)
mindMap.emit(event, ...args)
mindMap.off(event, fn)
```

### 导出

```javascript
await mindMap.export(type, options)  // 需要 Export 插件
```

### 生命周期

```javascript
mindMap.destroy()              // 清理销毁
```

### 静态方法

```javascript
MindMap.usePlugin(plugin, opt)
MindMap.hasPlugin(plugin)
MindMap.defineTheme(name, config)
MindMap.removeTheme(name)
MindMap.extendNodeDataNoStylePropList(list)
MindMap.resetNodeDataNoStylePropList()
```

### 插件

```javascript
mindMap.addPlugin(plugin, opt)
mindMap.removePlugin(plugin)
```

---

## 9. 关键文件路径参考

| 用途 | 绝对路径 |
|------|----------|
| MindMap 类（主入口） | `/simple-mind-map/index.js` |
| 所有默认选项 | `/simple-mind-map/src/constants/defaultOptions.js` |
| 布局常量及名称 | `/simple-mind-map/src/constants/constant.js` |
| 渲染器（布局切换） | `/simple-mind-map/src/core/render/Render.js` |
| 基础布局类 | `/simple-mind-map/src/layouts/Base.js` |
| MindMap 布局（双向） | `/simple-mind-map/src/layouts/MindMap.js` |
| LogicalStructure 布局（水平） | `/simple-mind-map/src/layouts/LogicalStructure.js` |
| VerticalTimeline 布局 | `/simple-mind-map/src/layouts/VerticalTimeline.js` |
| OrganizationStructure 布局（垂直） | `/simple-mind-map/src/layouts/OrganizationStructure.js` |
| Timeline 布局 | `/simple-mind-map/src/layouts/Timeline.js` |
| MindMapLayoutPro 插件 | `/simple-mind-map/src/plugins/MindMapLayoutPro.js` |
| 完整导入（含插件） | `/simple-mind-map/full.js` |
| 示例数据 | `/simple-mind-map/example/exampleData.js` |
| 演示 Edit.vue（实例创建） | `/web/src/pages/Edit/components/Edit.vue` |
| 演示 NavigatorToolbar（只读UI） | `/web/src/pages/Edit/components/NavigatorToolbar.vue` |
| 演示 Structure.vue（布局UI） | `/web/src/pages/Edit/components/Structure.vue` |
| Vuex store（只读状态） | `/web/src/store.js` |
| 中文扩展文档 | `/README_MORE_ZH.md` |