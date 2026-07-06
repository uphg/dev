# Naive UI Vue 组件书写风格规范

> 基于 naive-ui `src/` 源码分析，涵盖目录结构、Props、类型、CSS、主题、测试等全链路编码约定。

---

## 1. 目录结构规范

每个组件遵循统一的目录布局：

```
src/<component>/
├── index.ts                       # 对外导出（component、props、types）
├── src/
│   ├── <Component>.tsx            # defineComponent 组件实现
│   ├── public-types.ts            # 公开类型导出（Size、Type 等枚举）
│   ├── interface.ts               # 内部类型/接口定义（可选）
│   ├── utils.ts                   # 组件专用工具函数（可选）
│   └── styles/
│       └── index.cssr.ts          # CSS-in-JS 样式（cssr）
├── styles/
│   ├── _common.ts                 # 尺寸无关的共享变量
│   ├── light.ts                   # 亮色主题变量
│   ├── dark.ts                    # 暗色主题变量
│   ├── rtl.ts                     # RTL 样式覆盖（可选）
│   └── index.ts                   # 主题重导出
├── tests/
│   ├── <Component>.spec.tsx       # Vitest 测试
│   ├── server.spec.tsx            # SSR 兼容测试
│   └── __snapshots__/             # 快照产物
└── demos/                         # 文档站点示例
    ├── enUS/
    └── zhCN/
```

**关键约定：**

| 约定 | 说明 |
|------|------|
| 文件命名 | 组件实现文件用 `PascalCase`：`Button.tsx`、`Input.tsx` |
| 目录命名 | `kebab-case`：`date-picker/`、`auto-complete/` |
| 子组件 | 放在同一 `src/` 下：`InputGroup.tsx`、`InputGroupLabel.tsx` |
| 主题文件 | 必须在 `styles/` 下包含 `light.ts`、`dark.ts`、`index.ts` |

---

## 2. Props 声明规范

Props 定义为**独立的 `const` 对象**（非 inline），以 **`as const`** 收尾，始终以 `useTheme.props` 展开开头。

```ts
// src/button/src/Button.tsx
export const buttonProps = {
  ...(useTheme.props as ThemeProps<ButtonTheme>),

  // 简单布尔：Boolean 简写
  block: Boolean,
  loading: Boolean,
  disabled: Boolean,
  circle: Boolean,

  // 带默认值：Object 形式
  focusable: { type: Boolean, default: true },
  tag: { type: String as PropType<keyof HTMLElementTagNameMap>, default: 'button' },
  type: { type: String as PropType<ButtonType>, default: 'default' },

  // 字符串联合类型
  size: String as PropType<ButtonSize>,
  iconPlacement: { type: String as PropType<'left' | 'right'>, default: 'left' },

  // 函数类型
  renderIcon: Function as PropType<() => VNodeChild>,

  // 支持数组的 handler（多回调）
  onClick: [Function, Array] as PropType<MaybeArray<(e: MouseEvent) => void>>,

  // v-model（同时提供 CamelCase 和 kebab-case 两种形式）
  onUpdateValue: [Function, Array] as PropType<MaybeArray<OnUpdateValue>>,
  'onUpdate:value': [Function, Array] as PropType<MaybeArray<OnUpdateValue>>,

  // 废弃/私有属性标记
  /** @deprecated */ oldProp: String,
  /** @private */ internalXxx: String,
} as const
```

**Props 规范速查：**

| 规则 | 示例 |
|------|------|
| 命名风格 | **camelCase**：`iconPlacement`、`maxTagCount`、`showPasswordOn` |
| 简单布尔 | `block: Boolean`（无默认值用简写） |
| 带默认值 | `{ type: Boolean, default: true }` |
| 字符串联合 | `String as PropType<'left' \| 'right'>` |
| 函数 | `Function as PropType<(...) => void>` |
| 多回调 | `[Function, Array] as PropType<MaybeArray<...>>` |
| Theme 展开 | 第一行必须 `...useTheme.props` |
| v-model | 同时提供 `onUpdateValue` 和 `'onUpdate:value'` |
| 废弃标记 | `/** @deprecated */` |
| 私有标记 | `/** @private */`，Key 以 `internal` 开头 |
| 元组断言 | 始终以 `} as const` 结尾 |

---

## 3. 组件实现规范

使用 `defineComponent`，明确分离 `setup()` 和 `render()`：

```ts
// src/button/src/Button.tsx
export default defineComponent({
  name: 'Button',              // PascalCase 组件名
  props: buttonProps,          // 引用导出的 props const
  slots: Object as SlotsType<ButtonSlots>,  // 类型化 slots

  setup(props) {
    // ===== 响应式引用 =====
    const selfElRef = ref<HTMLElement | null>(null)

    // ===== Config/Hooks =====
    const { mergedClsPrefixRef, inlineThemeDisabled } = useConfig(props)
    const themeRef = useTheme(
      'Button', '-button', style, buttonLight, props, mergedClsPrefixRef
    )
    const formItem = useFormItem(props)
    const { mergedSizeRef } = useFormItem(props)

    // ===== 计算属性 =====
    const cssVarsRef = computed(() => { /* ... */ })
    const mergedDisabledRef = computed(() => {
      // formItem?.disabled 合并逻辑
    })

    // ===== 事件处理 =====
    function handleClick(e: MouseEvent): void {
      // ...
    }

    // ===== 返回 =====
    return {
      selfElRef,
      mergedClsPrefix: mergedClsPrefixRef,
      cssVars: inlineThemeDisabled ? undefined : cssVarsRef,
      handleClick,
    }
  },

  render() {
    // 通过 this 访问 setup 返回值
    const {
      mergedClsPrefix,
      tag: Component,
      onRender,
      handleClick,
    } = this

    return (
      <Component
        class={[
          `${mergedClsPrefix}-button`,
          this.themeClass,
          { [`${mergedClsPrefix}-button--disabled`]: this.mergedDisabled },
        ]}
        style={this.cssVars}
        onClick={handleClick}
      >
        {resolveWrappedSlot(this.$slots.default, (children) => /* ... */)}
      </Component>
    )
  },
})
```

### 命名速查表

| 类别 | 命名规则 | 示例 |
|------|----------|------|
| **ref** | `xxxRef` | `selfElRef`、`hoverRef`、`focusedRef` |
| **computed** | `xxxRef` | `cssVarsRef`、`mergedSizeRef`、`mergedDisabledRef` |
| **config 合并值** | `mergedXxxRef` | `mergedClsPrefixRef`、`mergedSizeRef`、`mergedValueRef` |
| **事件处理函数** | `handleXxx` | `handleClick`、`handleMouseDown`、`handleKeydown` |
| **副作用执行** | `doXxx` | `doUpdateValue`、`doBlur`、`doClear`、`doFocus` |
| **组件名** | `PascalCase` | `'Button'`、`'Input'`、`'DatePicker'` |

### provide/inject 模式

组件向外暴露内部状态时，使用 `createInjectionKey` 创建注入键：

```ts
// src/input/src/interface.ts
export const inputInjectionKey = createInjectionKey<{
  mergedValueRef: Ref<string | null>
  maxlengthRef: Ref<number | undefined>
  // ...
}>('n-input')

// 在父组件 setup 中 provide
provide(inputInjectionKey, {
  mergedValueRef,
  maxlengthRef,
})
```

---

## 4. 类型命名规范

### 完整类型表格

| 类型模式 | 命名规则 | 示例 |
|----------|----------|------|
| **Props** | `{Component}Props` | `ButtonProps`、`InputProps`、`SelectProps` |
| **Slots** | `{Component}Slots` | `ButtonSlots`、`InputSlots`、`SelectSlots` |
| **尺寸** | `{Component}Size` | `ButtonSize`、`InputSize`、`SelectSize` |
| **类型枚举** | `{Component}Type` | `ButtonType` |
| **主题** | `{Component}Theme` | `ButtonTheme`、`InputTheme`、`SelectTheme` |
| **主题变量** | `{Component}ThemeVars` | `ButtonThemeVars`、`InputThemeVars` |
| **实例方法** | `{Component}Inst` | `InputInst`、`SelectInst` |
| **注入键** | `{Component}Injection` | `FormItemInjection`、`ModalInjection` |
| **暴露引用** | `{Component}WrappedRef` | `InputWrappedRef` |
| **回调类型** | `OnXxx` | `OnUpdateValue`、`OnUpdateValueImpl` |
| **选项类型** | `{Component}XxxOption` | `SelectBaseOption`、`SelectGroupOption` |

### Slots 接口

```ts
// src/button/src/Button.tsx
export interface ButtonSlots {
  default?: () => VNode[]
  icon?: () => VNode[]
}

// src/input/src/Input.tsx — 带参数的 slot
export interface InputSlots {
  'clear-icon'?: () => VNode[]
  count?: (props: { value: string }) => VNode[]
  suffix?: () => VNode[]
  // kebab-case 命名
}

// src/modal/src/Modal.tsx — 组合 slots
export type ModalSlots = Omit<CardSlots & DialogSlots, 'default'> & {
  default?: (props: { draggableClass: string }) => VNode[]
}
```

### Props 类型提取

公开 API 的类型通过 `ExtractPublicPropTypes` 从 prop 定义中自动推导：

```ts
// src/_utils/naive/extract-public-props.ts
export type ExtractPublicPropTypes<T> = Omit<
  Partial<RemoveReadonly<ExtractPropTypes<T>>>,
  | Exclude<themePropKeys, 'themeOverrides'>
  | Extract<keyof T, `internal${string}`>
>
// 自动过滤掉 theme 内部属性和 internal 前缀属性
```

---

## 5. CSS-in-JS (cssr) 规范

使用 `css-render` + `@css-render/plugin-bem`，严格遵循 BEM 命名：

```ts
// src/_utils/cssr/index.ts 中的全局定义
const namespace = 'n'          // 命名空间前缀
const prefix = `.${namespace}-` // => '.n-'
const elementPrefix = '__'     // BEM 元素分隔符
const modifierPrefix = '--'    // BEM 修饰符分隔符
```

### DSL 函数速查

| 函数 | 生成结果 | 含义 |
|------|----------|------|
| `cB('button', ...)` | `.n-button` | Block 块级节点 |
| `cE('icon', ...)` | `.n-button__icon` | Element 元素节点 |
| `cM('disabled', ...)` | `.n-button--disabled` | Modifier 修饰符 |
| `cNotM('disabled', ...)` | `.n-button:not(.n-button--disabled)` | 否定修饰符 |
| `c('&:hover', ...)` | `.n-button:hover` | 伪类选择器 |
| `c('>', [...])` | `.n-button > ...` | 子选择器 |

### 典型写法

```ts
// src/button/src/styles/index.cssr.ts
export default c([
  cB('button', `
    font-weight: var(--n-font-weight);
    padding: var(--n-padding);
    height: var(--n-height);
    color: var(--n-text-color);
    background-color: var(--n-color);
  `, [
    // 按颜色类型（colorType）应用修饰符
    cM('color', [
      cE('border', { borderColor: 'var(--n-border-color)' }),
    ]),
    // 尺寸修饰符 + 动态生成
    ...['tiny', 'small', 'medium', 'large'].map(size =>
      cM(size, { padding: `var(--n-padding-${size})` })
    ),
    // 禁用状态
    cM('disabled', { backgroundColor: 'var(--n-color-disabled)' }),
    // 非禁用状态下的交互伪类
    cNotM('disabled', [
      c('&:focus', { backgroundColor: 'var(--n-color-focus)' }),
      c('&:hover', { backgroundColor: 'var(--n-color-hover)' }),
      c('&:active', { backgroundColor: 'var(--n-color-pressed)' }),
    ]),
  ]),
  // 同级组件样式（非嵌套）
  cB('button-group', `...`),
])
```

### CSS 自定义属性前缀

所有 CSS 变量统一使用 `--n-` 前缀：

```
var(--n-color)
var(--n-font-size)
var(--n-border)
var(--n-box-shadow)
```

这些变量通过组件 render 中的 `style={this.cssVars}` 内联注入，其中 `cssVarsRef` 是使用 `useTheme` 返回值计算得到的 computed。

---

## 6. 主题变量命名规范

### 文件约定

| 文件 | 作用 |
|------|------|
| `styles/_common.ts` | 尺寸相关共享常量（padding、iconSize、rippleDuration 等） |
| `styles/light.ts` | 亮色主题变量定义 |
| `styles/dark.ts` | 暗色主题变量定义 |
| `styles/rtl.ts` | RTL（从右到左）样式覆盖 |
| `styles/index.ts` | 主题文件重导出 |

### 变量命名模式

遵循 `<属性><状态><变体>` 的顺序：

```
colorHoverPrimary
 │     │     │
属性  状态   变体
```

**完整示例（Button）：**

```ts
// src/button/styles/light.ts
export function self(vars: ThemeCommonVars) {
  const { primaryColor, primaryColorHover, textColor2, borderColor, buttonColor2 } = vars
  return {
    // 默认状态（无后缀 = default type）
    color: '#0000',
    colorHover: '#0000',
    colorPressed: '#0000',
    colorFocus: '#0000',
    colorDisabled: '#0000',
    textColor: textColor2,
    textColorHover: primaryColorHover,
    textColorPressed: primaryColorPressed,
    textColorDisabled: textColor3,
    border: `1px solid ${borderColor}`,
    borderHover: `1px solid ${primaryColorHover}`,
    rippleColor: primaryColor,

    // Primary 变体
    colorPrimary: primaryColor,
    colorHoverPrimary: primaryColorHover,
    textColorPrimary: baseColor,
    textColorHoverPrimary: baseColor,
    borderPrimary: `1px solid ${primaryColor}`,

    // Info 变体
    colorInfo: infoColor,
    colorHoverInfo: infoColorHover,

    // Text（文本按钮）变体
    textColorText: textColor2,
    textColorTextHover: primaryColorHover,
    textColorTextPressed: primaryColorPressed,

    // Ghost 变体
    textColorGhost: textColor2,
    textColorGhostHover: primaryColorHover,

    // 其他变体
    colorSecondary: buttonColor2,
    colorSecondaryHover: buttonColor2Hover,
    colorTertiary: buttonColor2,
    colorQuaternary: '#0000',

    // 非交互属性（无状态后缀）
    waveOpacity: '0.6',
    fontWeight: '400',
    fontWeightStrong: '500',
  }
}
```

### Input 主题（无变体，仅交互状态）

```ts
// src/input/styles/light.ts
export function self(vars: ThemeCommonVars) {
  return {
    textColor: textColor2,
    textColorDisabled,
    color: inputColor,
    colorDisabled: inputColorDisabled,
    colorFocus: inputColor,
    border: `1px solid ${borderColor}`,
    borderHover: `1px solid ${primaryColorHover}`,
    borderFocus: `1px solid ${primaryColorHover}`,
    boxShadowFocus: `0 0 0 2px ...`,
    // 状态变体
    loadingColorWarning: warningColor,
    borderWarning: `1px solid ${warningColor}`,
    loadingColorError: errorColor,
    borderError: `1px solid ${errorColor}`,
  }
}
```

### 动态主题变量访问

使用 `createKey` 函数动态拼接属性名：

```ts
// src/_utils/cssr/index.ts
function createKey<P extends string, S extends string>(prefix: P, suffix: S) {
  return prefix + (suffix === 'default'
    ? ''
    : suffix.replace(/^[a-z]/, startChar => startChar.toUpperCase()))
}

// 使用示例
createKey('textColorGhost', mergedType)    // => 'textColorGhostPrimary'
createKey('color', mergedType)            // => 'colorPrimary'
createKey('padding', size)                // => 'paddingMedium'
```

### 主题构建方式

```ts
// Button 风格 — 直接对象返回
const buttonLight: Theme<'Button', ButtonThemeVars> = {
  name: 'Button',
  common: commonLight,
  self,
}

// Select 风格 — createTheme 函数（带 peers）
const selectLight = createTheme({
  name: 'Select',
  common: commonLight,
  peers: {
    InternalSelection: internalSelectionLight,
    InternalSelectMenu: internalSelectMenuLight,
  },
  self,
})
```

### 暗色主题（复用亮色 self，按需覆盖）

```ts
// src/button/styles/dark.ts
const buttonDark: ButtonTheme = {
  name: 'Button',
  common: commonDark,
  self(vars) {
    const commonSelf = self(vars)   // 复用亮色 self()
    commonSelf.waveOpacity = '0.8'   // 按需覆盖
    return commonSelf
  },
}
```

---

## 7. Emits 规范

**不使用 Vue 的 `emits` 选项**，事件全部通过 Props 传递，调用通过 `call()` 工具函数。

### 事件声明

```ts
// src/input/src/Input.tsx — props 声明中的事件
export const inputProps = {
  // 原生事件
  onMousedown: Function as PropType<(e: MouseEvent) => void>,
  onKeydown: Function as PropType<(e: KeyboardEvent) => void>,
  onKeyup: [Function, Array] as PropType<(e: KeyboardEvent) => void>,
  onFocus: [Function, Array] as PropType<MaybeArray<(e: FocusEvent) => void>>,
  onBlur: [Function, Array] as PropType<MaybeArray<(e: FocusEvent) => void>>,

  // 自定义事件
  onInput: [Function, Array] as PropType<OnUpdateValue>,
  onChange: [Function, Array] as PropType<OnUpdateValue>,
  onClear: [Function, Array] as PropType<MaybeArray<(e: MouseEvent) => void>>,

  // v-model（双形式）
  onUpdateValue: [Function, Array] as PropType<MaybeArray<OnUpdateValue>>,
  'onUpdate:value': [Function, Array] as PropType<MaybeArray<OnUpdateValue>>,
  onUpdateShow: [Function, Array] as PropType<MaybeArray<(value: boolean) => void>>,
  'onUpdate:show': [Function, Array] as PropType<MaybeArray<(value: boolean) => void>>,
} as const
```

### 事件调用

```ts
// src/_utils/vue/call.ts — 安全的函数调用工具
export function call<T extends (...args: any[]) => any>(
  func: MaybeArray<T>,
  ...args: Parameters<T>
): void

// 使用示例
import { call } from '../../_utils/vue/call'

function handleClick(e: MouseEvent): void {
  const { onClick } = props
  // ...
  if (onClick) call(onClick, e)
}

function doUpdateValue(value: string): void {
  const { onUpdateValue, 'onUpdate:value': _onUpdateValue } = props
  if (onUpdateValue) call(onUpdateValue, value)
  if (_onUpdateValue) call(_onUpdateValue, value)  // 两个都要调
}
```

### v-model 约定

- 模板中 `v-model:value` 对应 `'onUpdate:value'`
- JSX 中 `onUpdateValue` 提供类型推断
- 两者在组件内部都需要被同步调用

---

## 8. Slots 规范

### Slot 在 render 中的使用方式

```tsx
// 1. resolveWrappedSlot — 条件包裹渲染（empty 时有条件地渲染外层）
{resolveWrappedSlot(this.$slots.default, (children) =>
  <span class="wrapper">
    {children}
  </span>
)}

// 2. resolveSlot — 带 fallback 内容
{resolveSlot(this.$slots.separator, () => [
  <span key="fallback">/</span>
])}

// 3. 直接调用（可选链）
{this.$slots['clear-icon']?.()}
{$slots.arrow?.()}

// 4. 判断是否为空
isSlotEmpty(this.$slots.default)
```

---

## 9. 测试规范

使用 **Vitest** + **@vue/test-utils**，组件描述用 kebab-case：

```ts
// src/button/tests/Button.spec.tsx
import { mount } from '@vue/test-utils'
import { h } from 'vue'

describe('n-button', () => {

  // 1. 冒烟测试 — 确保可挂载
  it('should work with import on demand', () => {
    mount(NButton)
  })

  // 2. 事件测试 — 用 vi.fn() 做 spy
  it('clickable', async () => {
    const onClick = vi.fn()
    const inst = mount(NButton, { props: { onClick } })
    await inst.trigger('click')
    expect(onClick).toHaveBeenCalled()
    inst.unmount()
  })

  // 3. Props class 测试 — 检查 CSS 类名
  it('should work with `block` prop', async () => {
    const wrapper = mount(NButton)
    expect(wrapper.find('.n-button').classes()).not.toContain('n-button--block')

    await wrapper.setProps({ block: true })
    expect(wrapper.find('.n-button').classes()).toContain('n-button--block')
    wrapper.unmount()
  })

  // 4. 样式快照测试
  it('should work with `size` prop', async () => {
    ;(['tiny', 'small', 'medium', 'large'] as const).forEach((size) => {
      const wrapper = mount(NButton, { props: { size } })
      expect(wrapper.find('button').attributes('style')).toMatchSnapshot()
      wrapper.unmount()
    })
  })

  // 5. Slot 测试
  it('should work with `icon` slot', async () => {
    const wrapper = mount(NButton, {
      slots: {
        icon: () => h(NIcon, null, { default: () => h(CashIcon) }),
        default: () => 'test',
      },
    })
    expect(wrapper.find('.n-button__icon').exists()).toBe(true)
    wrapper.unmount()
  })

  // 6. 编译期类型检查
  it('passed native event & attr tsx type checking', () => {
    ;(() => <div><NxButton onMousedown={() => {}} /></div>)()
  })
})
```

### 测试规范速查

| 规则 | 说明 |
|------|------|
| `describe` 块名 | `'n-component-name'`（kebab-case） |
| 测试命名 | `it('should work with \`propName\` prop', ...)` |
| 挂载 | `mount(NComponent, { props: { ... } })` |
| 动态改 prop | `await wrapper.setProps({ ... })` |
| 事件断言 | `vi.fn()` + `await inst.trigger('click')` + `expect(fn).toHaveBeenCalled()` |
| 类名断言 | `wrapper.find('.n-x').classes().toContain('n-x--y')` |
| 样式断言 | `wrapper.find('button').attributes('style').toMatchSnapshot()` |
| 清理 | **始终调用 `wrapper.unmount()`** |
| SSR 测试 | 单独放在 `server.spec.tsx` |

---

## 10. 内部 Mixins/Utils 体系

### `src/_mixins/` — 核心 composable

| 文件 | 用途 |
|------|------|
| `use-theme.ts` | 主题解析：合并组件主题、ConfigProvider、默认值；导出 `useTheme`、`createTheme`、`Theme`、`ThemeProps` |
| `use-config.ts` | ConfigProvider 注入：`clsPrefix`、`bordered`、RTL、breakpoints、组件 prop 覆盖 |
| `use-form-item.ts` | FormItem 注入：`size`、`disabled`、`status`、验证触发器（`nTriggerFormBlur` 等） |
| `use-locale.ts` | i18n 注入 |
| `use-css-vars-class.ts` | CSS 变量 class-based 主题 |
| `use-style.ts` | CSS 渲染工具 |
| `use-rtl.ts` | RTL 方向检测 |
| `use-hljs.ts` | Highlight.js 集成 |
| `common.ts` | 共享常量 |

### `src/_utils/` — 工具模块

| 模块目录 | 关键导出 |
|----------|----------|
| `vue/` | `call()`、`createInjectionKey()`、`resolveSlot()`、`resolveWrappedSlot()`、`isSlotEmpty()`、`flatten()`、`Wrapper`、`omit()`、`keep()` |
| `cssr/` | `c`、`cB`、`cE`、`cM`、`cNotM`、`createKey()`、`insideModal()`、`insidePopover()` |
| `css/` | `color2Class()`、`formatLength()`、`rtlInset()` |
| `naive/` | `ExtractPublicPropTypes`、`warn()`、`warnOnce()`、`smallerSize()`、`largerSize()`、`getTitleAttribute()` |
| `composable/` | `useIsComposing`、`useAdjustedTo`、`useResize`、`useDeferredTrue`、`useLockHtmlScroll` |
| `dom/` | `download()`、`isDocument()` |
| `env/` | `isBrowser`、`isJsdom`、`isNativeLazyLoad`、`isSafari` |
| `ts/` | 通用 TypeScript 工具类型 |

### `src/_internal/` — 内部共享组件

| 组件 | 谁在使用 |
|------|----------|
| `scrollbar/` | Input textarea、Select menu |
| `select-menu/` | Select、Cascader |
| `selection/` | Select 触发区 |
| `loading/` | 全局 |
| `clear/` | Input、Select |
| `wave/` | Button ripple 效果 |
| `focus-detector/` | 焦点检测 |

---

## 11. 编码风格杂项

- **`as const`**：所有 prop 定义都用，确保最严格的类型推断
- **文件头注释**：不需要（代码自解释）
- **Prettier**：`semi: false`、`singleQuote: true`、`printWidth: 80`、`trailingComma: none`
- **ESLint**：`@antfu/eslint-config`
- **格式化命令**：`pnpm run lint:fix` + `pnpm run format`
- **Git Commit**：Angular 风格：`feat(button): add ghost prop`
- **不写无用注释**，保持代码简洁

---

## 快速参考卡片

```
┌─────────────────────────────────────────────────────────┐
│  目录结构       │ kebab-case / PascalCase / styles+tests │
│  Props 命名     │ camelCase, as const, useTheme 展开     │
│  Ref 命名       │ xxxRef, mergedXxxRef                   │
│  事件处理       │ handleXxx, doXxx                       │
│  类型命名       │ XxxProps / XxxSlots / XxxTheme         │
│  CSS 类名       │ .n-block__element--modifier (BEM)      │
│  CSS 变量       │ var(--n-property-state-variant)        │
│  主题变量       │ propertyStateVariant (camelCase)        │
│  Slots          │ interface XxxSlots, resolveWrappedSlot │
│  Emits          │ Props 传递，call() 调用，无 emits 选项  │
│  测试           │ vitest, mount(), toMatchSnapshot(), unmount() │
│  编译检查       │ pnpm run lint                          │
│  格式化         │ pnpm run lint:fix && pnpm run format    │
└─────────────────────────────────────────────────────────┘
```
