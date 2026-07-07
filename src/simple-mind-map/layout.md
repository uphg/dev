根据代码库中的常量定义，当前库共有 **14 种布局**类型 [1](#0-0) 。

## 所有布局列表

| 布局名称 | 常量值 | 实现类 |
|---------|--------|--------|
| 逻辑结构图 | `logicalStructure` | `LogicalStructure` [2](#0-1)  |
| 向左逻辑结构图 | `logicalStructureLeft` | `LogicalStructure` [3](#0-2)  |
| 思维导图 | `mindMap` | `MindMap` [4](#0-3)  |
| 组织结构图 | `organizationStructure` | `OrganizationStructure` [5](#0-4)  |
| 目录组织图 | `catalogOrganization` | `CatalogOrganization` [6](#0-5)  |
| 时间轴 | `timeline` | `Timeline` [7](#0-6)  |
| 时间轴2 | `timeline2` | `Timeline` [8](#0-7)  |
| 竖向时间轴 | `verticalTimeline` | `VerticalTimeline` [9](#0-8)  |
| 竖向时间轴2 | `verticalTimeline2` | `VerticalTimeline` [10](#0-9)  |
| 竖向时间轴3 | `verticalTimeline3` | `VerticalTimeline` [11](#0-10)  |
| 鱼骨图 | `fishbone` | `Fishbone` [12](#0-11)  |
| 鱼骨图2 | `fishbone2` | `Fishbone` [13](#0-12)  |
| 向右鱼骨图 | `rightFishbone` | - |
| 向右鱼骨图2 | `rightFishbone2` | - |

## 布局分组

在 Web 界面中，这些布局被分为以下 6 个组 [14](#0-13) ：

1. **逻辑结构图** - `logicalStructure`, `logicalStructureLeft`
2. **思维导图** - `mindMap`
3. **组织结构图** - `organizationStructure`
4. **目录组织图** - `catalogOrganization`
5. **时间轴** - `timeline`, `timeline2`, `verticalTimeline`
6. **鱼骨图** - `fishbone`

## Notes

- `rightFishbone` 和 `rightFishbone2` 在常量中定义，但在 `Render.js` 的 `layouts` 对象中没有映射到实现类，可能未完全实现或作为保留选项 [15](#0-14) 。
- Web 界面的 `Structure.vue` 组件会过滤掉 `rightFishbone` 和 `rightFishbone2`，不在 UI 中显示 [16](#0-15) 。
- 部分布局共享同一个实现类，通过构造函数参数区分变体（如 `Timeline` 处理 `timeline` 和 `timeline2`） [17](#0-16) 。

Wiki pages you might want to explore:
- [Layout System (wanglin2/mind-map)](/wiki/wanglin2/mind-map#3)

### Citations

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

**File:** web/src/config/zh.js (L754-783)
```javascript
export const layoutGroupList = [
  {
    name: '逻辑结构图',
    list: ['logicalStructure', 'logicalStructureLeft']
  },
  {
    name: '思维导图',
    list: ['mindMap']
  },
  {
    name: '组织结构图',
    list: ['organizationStructure']
  },
  {
    name: '目录组织图',
    list: ['catalogOrganization']
  },
  {
    name: '时间轴',
    list: [
      'timeline',
      'timeline2',
      'verticalTimeline'
    ]
  },
  {
    name: '鱼骨图',
    list: ['fishbone']
  }
]
```

**File:** web/src/pages/Edit/components/Structure.vue (L58-60)
```vue
        let list = [...group.list].filter(item => {
          return !['rightFishbone', 'rightFishbone2'].includes(item)
        })
```

**File:** simple-mind-map/src/layouts/Timeline.js (L8-11)
```javascript
  constructor(opt = {}, layout) {
    super(opt)
    this.layout = layout
  }
```
