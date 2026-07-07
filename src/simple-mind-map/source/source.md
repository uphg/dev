# simple-mind-map Library Analysis Document

## Table of Contents
1. [Directory Structure](#1-directory-structure)
2. [Layout Types](#2-all-layout-types)
3. [Basic Creation](#3-basic-creation)
4. [Layout Direction (Horizontal vs Vertical)](#4-layout-direction-horizontal-vs-vertical)
5. [Readonly / Disabling Editing](#5-readonly--disabling-editing)
6. [Level-Based / Node-Based Layout Customization](#6-level-based--node-based-layout-customization)
7. [Complete Configuration Options Reference](#7-complete-configuration-options-reference)
8. [Public API Methods](#8-public-api-methods)
9. [Key File Paths Reference](#9-key-file-paths-for-reference)

---

## 1. Directory Structure

```
example/mind-map/
├── .git/
├── .gitignore
├── assets/
│   ├── client/          (12 PNG screenshots: client1-6.png, clienten1-6.png)
│   └── ob/              (10 PNG screenshots: ob1-5.png, oben1-5.png)
├── copy.js
├── dist/                (Built web app: js/, img/, fonts/)
├── Dockerfile
├── index.html
├── LICENSE              (MIT)
├── nginx.conf
├── README.md            (Main README, Chinese)
├── README_EN.md         (English README)
├── README_MORE_ZH.md    (Extended Chinese docs)
├── README_MORE_EN.md    (Extended English docs)
├── simple-mind-map/     (THE CORE LIBRARY - v0.14.0-fix.2)
│   ├── .eslintrc.js
│   ├── .prettierrc / .prettierignore
│   ├── bin/             (createPluginsTypeFiles.js, wsServer.mjs)
│   ├── example/         (exampleData.js, exportFullData.json)
│   ├── full.js          (All-in-one import with all plugins)
│   ├── index.js         (MAIN ENTRY POINT - MindMap class)
│   ├── package.json
│   ├── README.md        (Minimal - points to GitHub)
│   ├── scripts/         (walkJsFiles.js)
│   └── src/
│       ├── constants/
│       │   ├── constant.js          (Layout names, DIR constants, shape types)
│       │   └── defaultOptions.js    (ALL configuration options ~523 lines)
│       ├── core/
│       │   ├── command/Command.js   (History, undo/redo)
│       │   ├── command/KeyCommand.js
│       │   ├── event/Event.js
│       │   ├── render/Render.js     (Main renderer, layout switching, commands)
│       │   ├── render/TextEdit.js
│       │   ├── render/node/MindMapNode.js  (Node behaviors, readonly checks)
│       │   ├── render/node/Shape.js
│       │   ├── render/node/Style.js
│       │   └── view/View.js
│       ├── layouts/
│       │   ├── Base.js              (Abstract base class for all layouts)
│       │   ├── LogicalStructure.js   (Horizontal tree: right or left)
│       │   ├── MindMap.js           (Dual-direction: left+right branches)
│       │   ├── CatalogOrganization.js
│       │   ├── OrganizationStructure.js (Vertical top-down hierarchy)
│       │   ├── Timeline.js          (Horizontal timeline)
│       │   ├── VerticalTimeline.js   (Vertical timeline)
│       │   ├── Fishbone.js          (Fishbone diagram)
│       │   └── fishboneUtils.js
│       ├── plugins/                 (19+ plugins)
│       │   ├── MindMapLayoutPro.js  (**Per-node direction control**)
│       │   ├── Drag.js, Select.js, Export.js, ...
│       ├── parse/                   (xmind.js, markdown.js, markdownTo.js, toTxt.js)
│       ├── theme/                   (default.js, index.js)
│       ├── svg/                     (btns.js, icons.js)
│       └── utils/                   (Various utilities)
└── web/                 (The Vue2 demo app)
    ├── package.json
    ├── babel.config.js / vue.config.js
    ├── .env.library
    ├── src/
    │   ├── main.js
    │   ├── router.js
    │   ├── store.js
    │   └── pages/Edit/components/
    │       ├── Edit.vue               (**Creates MindMap instance**)
    │       ├── NavigatorToolbar.vue    (**Readonly toggle UI**)
    │       ├── Structure.vue           (**Layout switching UI**)
    │       └── ...
    └── public/
```

### Option at Construction

Basic configuration when creating a MindMap instance:

```javascript
const mindMap = new MindMap({
  el: document.getElementById('container'),
  data: mindMapData,
  readonly: false,  // Default is false (editable)
  layout: 'logicalStructure'
});
```

### How the Demo App Toggles Readonly

The demo app uses Vuex state management to control readonly mode:

```javascript
// NavigatorToolbar.vue (lines 179-182)
readonlyChange() {
  this.setIsReadonly(!this.isReadonly)           // Vuex state
  this.mindMap.setMode(this.isReadonly ? 'readonly' : 'edit')  // MindMap API
}

// store.js (line 26)
isReadonly: false   // Default Vuex state
```

### Real-world Creation from Demo App

```javascript
// Edit.vue - Creates MindMap instance with full configuration
const mindMap = new MindMap({
  el: this.$refs.mindMapContainer,
  data: this.mindMapData,
  layout: this.currentLayout,
  theme: this.currentTheme,
  readonly: this.isReadonly,
  // ... additional options
});
```

---

## 2. All Layout Types

### Growth Directions (from constant.js)

The `LAYOUT_GROW_DIR` constants define how nodes expand:

| Constant | Value | Description |
|----------|-------|-------------|
| `RIGHT` | 'right' | Grow to the right |
| `LEFT` | 'left' | Grow to the left |
| `DOWN` | 'down' | Grow downward |
| `UP` | 'up' | Grow upward |

### Layout-to-Class Mapping (in Render.js, lines 46-71)

```javascript
const layouts = {
  [CONSTANTS.LAYOUT.LOGICAL_STRUCTURE]: LogicalStructure,
  [CONSTANTS.LAYOUT.LOGICAL_STRUCTURE_LEFT]: LogicalStructure,  // Same class, isUseLeft=true
  [CONSTANTS.LAYOUT.MIND_MAP]: MindMap,                        // Own class
  [CONSTANTS.LAYOUT.CATALOG_ORGANIZATION]: CatalogOrganization,
  [CONSTANTS.LAYOUT.ORGANIZATION_STRUCTURE]: OrganizationStructure,
  [CONSTANTS.LAYOUT.TIMELINE]: Timeline,
  [CONSTANTS.LAYOUT.TIMELINE2]: Timeline,
  [CONSTANTS.LAYOUT.VERTICAL_TIMELINE]: VerticalTimeline,
  [CONSTANTS.LAYOUT.VERTICAL_TIMELINE2]: VerticalTimeline,    // Same class, differs in dir logic
  [CONSTANTS.LAYOUT.VERTICAL_TIMELINE3]: VerticalTimeline,
  [CONSTANTS.LAYOUT.FISHBONE]: Fishbone,
  [CONSTANTS.LAYOUT.FISHBONE2]: Fishbone
}
```

---

## 3. Basic Creation

### From README_MORE_ZH.md and index.js

```javascript
import MindMap from 'simple-mind-map'

const mindMap = new MindMap({
  el: document.getElementById('mindMapContainer'),
  data: {
    data: {
      text: 'Root Node'
    },
    children: []
  }
})
```

---

## 4. Layout Direction (Horizontal vs Vertical)

### Layout Summary Table

| Layout Name | Direction | Description |
|-------------|-----------|-------------|
| `logicalStructure` | Horizontal (rightward) | Single tree growing right |
| `logicalStructureLeft` | Horizontal (leftward) | Single tree growing left |
| `mindMap` | Horizontal (dual) | Branches alternate left/right |
| `organizationStructure` | Vertical (downward) | Org chart, top-down |
| `catalogOrganization` | Vertical (downward) | File tree style |
| `timeline` | Horizontal | Timeline, downward growth |
| `timeline2` | Horizontal (alternating) | Timeline, top/bottom alternating |
| `verticalTimeline` | Vertical | Timeline, left/right alternating |
| `verticalTimeline2` | Vertical (all left) | All branches go left |
| `verticalTimeline3` | Vertical (all right) | All branches go right |
| `fishbone` / `fishbone2` | Horizontal (alternating) | Fishbone diagram |

### Changing Layout Programmatically

```javascript
// Get current layout
const currentLayout = mindMap.getLayout()

// Set new layout
mindMap.setLayout('mindMap')

// With optional notRender parameter (prevents re-render)
mindMap.setLayout('logicalStructure', true)
```

### How the Demo App Switches Layout

```javascript
// Structure.vue (lines 79-85)
useLayout(layout) {
  this.layout = layout
  this.mindMap.setLayout(layout)
}
```

**Events:** `layout_change` is emitted when layout changes.

---

## 5. Readonly / Disabling Editing

### Default Configuration

```javascript
// defaultOptions.js (line 13)
readonly: false   // Default is false (editable)
```

### Set Readonly Mode Programmatically

```javascript
// index.js (lines 541-562)
mindMap.setMode('readonly')   // Enable read-only mode
mindMap.setMode('edit')       // Switch back to edit mode
```

### What `setMode()` Does:

- Hides the text edit box if open
- Clears the active node
- Prevents history operations (`addHistory`, `back`, `forward`)
- Emits `mode_change` event with `'readonly'` or `'edit'`

### Effects of Readonly Mode

The `readonly` flag is checked throughout the codebase:

| File | Line(s) | Blocked Operation |
|------|---------|-------------------|
| Command.js | 106 | History recording |
| Command.js | 137, 157 | Undo/redo operations |
| MindMapNode.js | 364, 378, 395, 422, 435, 461 | Node interactions |
| Render.js | 708 | Select all |
| Drag.js | 89, 107 | Drag operations |
| NodeImgAdjust.js | 66 | Image adjustment |
| Painter.js | 26 | Format painter |
| Select.js | 45, 68, 123 | Node selection |
| Search.js | 97, 197, 210 | Search replace |

---

## 6. Level-Based / Node-Based Layout Customization

### Per-Node Direction Control (`dir` property)

Each node can have a `dir` property in its data to control the growth direction of its children.

**Example: MindMap layout (MindMap.js line 57):**

```javascript
newNode.dir = newNode.getData('dir') ||
  (index % 2 === 0
    ? CONSTANTS.LAYOUT_GROW_DIR.RIGHT
    : CONSTANTS.LAYOUT_GROW_DIR.LEFT)
```

**Example: VerticalTimeline layout (VerticalTimeline.js lines 53-65):**

```javascript
if (this.layout === CONSTANTS.LAYOUT.VERTICAL_TIMELINE2) {
  newNode.dir = CONSTANTS.LAYOUT_GROW_DIR.LEFT   // All left
} else if (this.layout === CONSTANTS.LAYOUT.VERTICAL_TIMELINE3) {
  newNode.dir = CONSTANTS.LAYOUT_GROW_DIR.RIGHT  // All right
} else {
  // Alternating based on index
  newNode.dir = index % 2 === 0
    ? CONSTANTS.LAYOUT_GROW_DIR.RIGHT
    : CONSTANTS.LAYOUT_GROW_DIR.LEFT
}
```

### Important: Children Inherit Parent's Direction

```javascript
// MindMap.js lines 52-53
if (parent._node.dir) {
  newNode.dir = parent._node.dir   // Children inherit parent's direction
}
```

### Example: Force Node Direction

```javascript
// Force a node and all its descendants to grow only rightward
{
  data: {
    text: 'Section A',
    dir: 'right'   // CONSTANTS.LAYOUT_GROW_DIR.RIGHT
  },
  children: [...]
}
```

### MindMapLayoutPro Plugin (Automatic Direction Assignment)

File: `src/plugins/MindMapLayoutPro.js`

This plugin automatically assigns `dir` to second-level nodes in `mindMap` layout to balance left/right distribution:

```javascript
// Lines 84-97: Balances children around the center
updateNodeTree(tree) {
  const childrenLength = root.children.length
  const center = Math.ceil(childrenLength / 2)
  root.children.forEach((item, index) => {
    if (index + 1 <= center) {
      item.data.dir = CONSTANTS.LAYOUT_GROW_DIR.RIGHT  // First half goes right
    } else {
      item.data.dir = CONSTANTS.LAYOUT_GROW_DIR.LEFT   // Second half goes left
    }
  })
}
```

**Plugin listeners:**
- `layout_change` - rebalances when layout changes
- `afterExecCommand` - rebalances after insert/delete/move operations
- `before_update_data` / `before_set_data` - rebalances on data import

### Node Data Properties for Custom Positioning

From `nodeDataNoStylePropList` in constant.js (lines 174-210):

```javascript
// Non-style data properties each node can have:
'text', 'image', 'imageTitle', 'imageSize', 'icon', 'tag',
'hyperlink', 'hyperlinkTitle', 'note', 'expand', 'isActive',
'generalization', 'richText', 'resetRichText', 'uid', 'activeStyle',
'associativeLineTargets', 'associativeLineTargetControlOffsets',
'associativeLinePoint', 'associativeLineText', 'attachmentUrl',
'attachmentName', 'notation', 'outerFrame', 'number', 'range',
'customLeft', 'customTop',    // <-- CUSTOM POSITIONING
'customTextWidth',
'checkbox', 'dir',            // <-- DIRECTION CONTROL
'needUpdate', 'imgMap', 'nodeLink'
```

> **Note:** `customLeft` and `customTop` are internal fields set by drag operations for manual positioning.

---

## 7. Complete Configuration Options Reference

Full options documented in: `src/constants/defaultOptions.js` (523 lines)

### Basic Options (lines 5-19)

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `el` | DOM Element | `null` | **Required.** Container element |
| `data` | Object | `null` | Mind map tree data |
| `readonly` | Boolean | `false` | Read-only mode |
| `layout` | String | `'logicalStructure'` | Layout type |
| `theme` | String | `'default'` | Theme name |
| `themeConfig` | Object | `{}` | Theme overrides |

### View Options (lines 20-35)

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `scaleRatio` | Number | `0.2` | Zoom scale step |
| `translateRatio` | Number | `1` | Pan step |
| `minZoomRatio` | Number | `20` | Minimum zoom (%) |
| `maxZoomRatio` | Number | `400` | Maximum zoom (%) |
| `fit` | Boolean | `false` | Auto-fit on init |
| `initRootNodePosition` | Array | `['center','center']` | Root position |

### Node Options (lines 40-80)

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `maxTag` | Number | `5` | Maximum tags shown |
| `expandBtnSize` | Number | `20` | Expand button size |
| `alwaysShowExpandBtn` | Boolean | `false` | Always show expand button |
| `notShowExpandBtn` | Boolean | `false` | Never show expand button |
| `textAutoWrapWidth` | Number | `500` | Auto-wrap threshold (pixels) |

### Interaction Options (lines 83-130)

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `enableCtrlKeyNodeSelection` | Boolean | `true` | Ctrl+click multi-select |
| `useLeftKeySelectionRightKeyDrag` | Boolean | `false` | Swap mouse buttons |
| `isDisableDrag` | Boolean | `false` | Disable panning |
| `disableMouseWheelZoom` | Boolean | `false` | Disable wheel zoom |
| `mousewheelAction` | String | `'move'` | `'zoom'` or `'move'` |
| `enableAutoEnterTextEditWhenKeydown` | Boolean | `false` | Auto-edit on keypress |
| `disabledClipboard` | Boolean | `false` | Disable clipboard |

---

## 8. Public API Methods

### Instance Creation

```javascript
new MindMap(opt)               // Constructor
```

### Data Management

```javascript
mindMap.setData(data)          // Replace all node data
mindMap.updateData(data)       // Update from modified data
mindMap.setFullData({root, layout, theme, view})  // Set everything
mindMap.getData(withConfig)    // Get data (optionally with config)
```

### Layout

```javascript
mindMap.getLayout()            // Get current layout name
mindMap.setLayout(layout, notRender)  // Set layout
```

**Events:** `'layout_change'`

### Theme

```javascript
mindMap.setTheme(theme, notRender)
mindMap.getTheme()
mindMap.setThemeConfig(config, notRender)
mindMap.getCustomThemeConfig()
mindMap.getThemeConfig(prop)
```

### Readonly / Edit Mode

```javascript
mindMap.setMode('readonly' | 'edit')  // Toggle read-only
```

**Events:** `'mode_change'`

### View

```javascript
mindMap.view.fit()             // Fit to container
mindMap.view.narrow()          // Zoom out
mindMap.view.enlarge()         // Zoom in
mindMap.view.reset()           // Reset zoom/position
mindMap.view.moveNodeToCenter(node)
```

### Configuration

```javascript
mindMap.getConfig(prop)        // Get config value
mindMap.updateConfig(opt)      // Merge new config
```

### Commands

```javascript
mindMap.execCommand(...args)   // Execute registered command
mindMap.command.addHistory()   // Manually add history
mindMap.command.clearHistory()
mindMap.command.pause()
mindMap.command.recovery()
```

### Events

```javascript
mindMap.on(event, fn)
mindMap.emit(event, ...args)
mindMap.off(event, fn)
```

### Export

```javascript
await mindMap.export(type, options)  // Requires Export plugin
```

### Lifecycle

```javascript
mindMap.destroy()              // Clean up
```

### Static Methods

```javascript
MindMap.usePlugin(plugin, opt)
MindMap.hasPlugin(plugin)
MindMap.defineTheme(name, config)
MindMap.removeTheme(name)
MindMap.extendNodeDataNoStylePropList(list)
MindMap.resetNodeDataNoStylePropList()
```

### Plugins

```javascript
mindMap.addPlugin(plugin, opt)
mindMap.removePlugin(plugin)
```

---

## 9. Key File Paths for Reference

| Purpose | Absolute Path |
|---------|---------------|
| MindMap class (main entry) | `/simple-mind-map/index.js` |
| All default options | `/simple-mind-map/src/constants/defaultOptions.js` |
| Layout constants & names | `/simple-mind-map/src/constants/constant.js` |
| Renderer (layout switching) | `/simple-mind-map/src/core/render/Render.js` |
| Base layout class | `/simple-mind-map/src/layouts/Base.js` |
| MindMap layout (dual-direction) | `/simple-mind-map/src/layouts/MindMap.js` |
| LogicalStructure (horizontal) | `/simple-mind-map/src/layouts/LogicalStructure.js` |
| VerticalTimeline layout | `/simple-mind-map/src/layouts/VerticalTimeline.js` |
| OrganizationStructure (vertical) | `/simple-mind-map/src/layouts/OrganizationStructure.js` |
| Timeline layout | `/simple-mind-map/src/layouts/Timeline.js` |
| MindMapLayoutPro plugin | `/simple-mind-map/src/plugins/MindMapLayoutPro.js` |
| All-in-one import (with plugins) | `/simple-mind-map/full.js` |
| Example data | `/simple-mind-map/example/exampleData.js` |
| Demo Edit.vue (instance creation) | `/web/src/pages/Edit/components/Edit.vue` |
| Demo NavigatorToolbar (readonly UI) | `/web/src/pages/Edit/components/NavigatorToolbar.vue` |
| Demo Structure.vue (layout UI) | `/web/src/pages/Edit/components/Structure.vue` |
| Vuex store (readonly state) | `/web/src/store.js` |
| Chinese extended docs | `/README_MORE_ZH.md` |