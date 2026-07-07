# catalogOrganization 布局源码分析

## 源码位置

`catalogOrganization` 布局的实现源码位于 `simple-mind-map/src/layouts/CatalogOrganization.js`，该类继承自 `Base` 布局基类 [1](#0-0) 。

在 `Render.js` 中，该布局被注册到布局列表中，通过常量 `CONSTANTS.LAYOUT.CATALOG_ORGANIZATION` 进行映射 [2](#0-1) 。

## 布局实现机制

### 布局流程

`CatalogOrganization` 类通过 `doLayout` 方法执行四步布局流程 [3](#0-2) ：

1. `computedBaseValue()` - 计算节点基本尺寸
2. `computedLeftTopValue()` - 计算节点位置
3. `adjustLeftTopValue()` - 调整节点位置避免重叠
4. 执行回调返回根节点

### 子节点布局差异实现

关键在于 `computedLeftTopValue` 方法中对根节点和非根节点的不同处理 [4](#0-3) ：

**根节点的子节点（水平排列）**：
```javascript
if (isRoot) {
  let left = node.left + node.width / 2 - node.childrenAreaWidth / 2
  let totalLeft = left + marginX
  node.children.forEach(cur => {
    cur.left = totalLeft
    totalLeft += cur.width + marginX
  })
}
```

**非根节点的子节点（垂直排列）**：
```javascript
else {
  let totalTop = node.top + this.getNodeHeightWithGeneralization(node) + marginY + ...
  node.children.forEach(cur => {
    cur.left = node.left + node.width * 0.5
    cur.top = totalTop
    totalTop += this.getNodeHeightWithGeneralization(cur) + marginY + ...
  })
}
```

## 如何调整子节点布局

要调整子节点布局与父节点不同，可以修改 `computedLeftTopValue` 方法中的逻辑：

1. **修改根节点子节点布局**：调整第 78-84 行的水平排列逻辑
2. **修改非根节点子节点布局**：调整第 85-99 行的垂直排列逻辑
3. **添加层级判断**：通过 `layerIndex` 参数为不同层级实现不同的布局策略

例如，如果想让第二层节点也水平排列，可以在 `computedLeftTopValue` 中添加对 `layerIndex === 1` 的判断，复用根节点的水平排列逻辑。

## Notes

- 目录组织图布局的特点是根节点子节点水平排列，后续层级子节点垂直排列 [5](#0-4) 
- 布局常量定义在 `simple-mind-map/src/constants/constant.js` 第 14 行 [6](#0-5) 
- 拖拽插件中也包含了对 `CATALOG_ORGANIZATION` 布局的特殊处理逻辑 [7](#0-6) 

Wiki pages you might want to explore:
- [Core Architecture (wanglin2/mind-map)](/wiki/wanglin2/mind-map#2)

### Citations

**File:** simple-mind-map/src/layouts/CatalogOrganization.js (L1-9)
```javascript
import Base from './Base'
import { walk, asyncRun, getNodeIndexInNodeList } from '../utils'

//  目录组织图
class CatalogOrganization extends Base {
  //  构造函数
  constructor(opt = {}) {
    super(opt)
  }
```

**File:** simple-mind-map/src/layouts/CatalogOrganization.js (L12-28)
```javascript
  doLayout(callback) {
    let task = [
      () => {
        this.computedBaseValue()
      },
      () => {
        this.computedLeftTopValue()
      },
      () => {
        this.adjustLeftTopValue()
      },
      () => {
        callback(this.root)
      }
    ]
    asyncRun(task)
  }
```

**File:** simple-mind-map/src/layouts/CatalogOrganization.js (L70-105)
```javascript
  computedLeftTopValue() {
    walk(
      this.root,
      null,
      (node, parent, isRoot, layerIndex) => {
        if (node.getData('expand') && node.children && node.children.length) {
          let marginX = this.getMarginX(layerIndex + 1)
          let marginY = this.getMarginY(layerIndex + 1)
          if (isRoot) {
            let left = node.left + node.width / 2 - node.childrenAreaWidth / 2
            let totalLeft = left + marginX
            node.children.forEach(cur => {
              cur.left = totalLeft
              totalLeft += cur.width + marginX
            })
          } else {
            let totalTop =
              node.top +
              this.getNodeHeightWithGeneralization(node) +
              marginY +
              (this.getNodeActChildrenLength(node) > 0 ? node.expandBtnSize : 0)
            node.children.forEach(cur => {
              cur.left = node.left + node.width * 0.5
              cur.top = totalTop
              totalTop +=
                this.getNodeHeightWithGeneralization(cur) +
                marginY +
                (this.getNodeActChildrenLength(cur) > 0 ? cur.expandBtnSize : 0)
            })
          }
        }
      },
      null,
      true
    )
  }
```

**File:** simple-mind-map/src/core/render/Render.js (L46-71)
```javascript
const layouts = {
  // 逻辑结构图
  [CONSTANTS.LAYOUT.LOGICAL_STRUCTURE]: LogicalStructure,
  // 向左逻辑结构图
  [CONSTANTS.LAYOUT.LOGICAL_STRUCTURE_LEFT]: LogicalStructure,
  // 思维导图
  [CONSTANTS.LAYOUT.MIND_MAP]: MindMap,
  // 目录组织图
  [CONSTANTS.LAYOUT.CATALOG_ORGANIZATION]: CatalogOrganization,
  // 组织结构图
  [CONSTANTS.LAYOUT.ORGANIZATION_STRUCTURE]: OrganizationStructure,
  // 时间轴
  [CONSTANTS.LAYOUT.TIMELINE]: Timeline,
  // 时间轴2
  [CONSTANTS.LAYOUT.TIMELINE2]: Timeline,
  // 竖向时间轴
  [CONSTANTS.LAYOUT.VERTICAL_TIMELINE]: VerticalTimeline,
  // 竖向时间轴2
  [CONSTANTS.LAYOUT.VERTICAL_TIMELINE2]: VerticalTimeline,
  // 竖向时间轴3
  [CONSTANTS.LAYOUT.VERTICAL_TIMELINE3]: VerticalTimeline,
  // 鱼骨图
  [CONSTANTS.LAYOUT.FISHBONE]: Fishbone,
  // 鱼骨图2
  [CONSTANTS.LAYOUT.FISHBONE2]: Fishbone
}
```

**File:** simple-mind-map/src/constants/constant.js (L9-24)
```javascript
  LAYOUT: {
    LOGICAL_STRUCTURE: 'logicalStructure',
    LOGICAL_STRUCTURE_LEFT: 'logicalStructureLeft',
    MIND_MAP: 'mindMap',
    ORGANIZATION_STRUCTURE: 'organizationStructure',
    CATALOG_ORGANIZATION: 'catalogOrganization',
    TIMELINE: 'timeline',
    TIMELINE2: 'timeline2',
    FISHBONE: 'fishbone',
    FISHBONE2: 'fishbone2',
    RIGHT_FISHBONE: 'rightFishbone',
    RIGHT_FISHBONE2: 'rightFishbone2',
    VERTICAL_TIMELINE: 'verticalTimeline',
    VERTICAL_TIMELINE2: 'verticalTimeline2',
    VERTICAL_TIMELINE3: 'verticalTimeline3'
  },
```

**File:** simple-mind-map/src/plugins/Drag.js (L1170-1178)
```javascript
  // 处理目录组织图
  handleCatalogOrganization(node) {
    const checkList = this.commonGetNodeCheckList(node)
    if (node.layerIndex === 1) {
      this.handleHorizontalCheck(node, checkList)
    } else {
      this.handleVerticalCheck(node, checkList)
    }
  }
```
