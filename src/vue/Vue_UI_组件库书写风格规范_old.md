# Naive UI Vue 组件书写风格规范 V2（函数式 · 无主题 · 增强健壮性）

> 基于 V1 迭代，新增响应式语义命名、Props 解构安全、枚举治理、Expose、副作用清理、透传 Attributes、组合式测试等关键规范。  
> **核心原则不变**：无 CSS-in-JS、无 `this`、无主题（`useTheme`）、仅 DOM 引用允许 `Ref` 后缀。

---

## 目录

- [Naive UI Vue 组件书写风格规范 V2（函数式 · 无主题 · 增强健壮性）](#naive-ui-vue-组件书写风格规范-v2函数式--无主题--增强健壮性)
  - [目录](#目录)
  - [1. 目录结构规范](#1-目录结构规范)
  - [2. Props 声明规范](#2-props-声明规范)
  - [3. 组件实现规范（函数式）](#3-组件实现规范函数式)
    - [3.1 基础模板](#31-基础模板)
    - [3.2 响应式命名语义前缀](#32-响应式命名语义前缀)
  - [4. 类型与枚举治理](#4-类型与枚举治理)
  - [5. Emits 规范](#5-emits-规范)
  - [6. Slots 规范（含类型定义）](#6-slots-规范含类型定义)
  - [7. Expose 实例方法规范](#7-expose-实例方法规范)
  - [8. 副作用与生命周期清理](#8-副作用与生命周期清理)
  - [9. 透传 Attributes 处理](#9-透传-attributes-处理)
  - [10. 测试规范（含组合式函数测试）](#10-测试规范含组合式函数测试)
    - [10.1 组件测试](#101-组件测试)
    - [10.2 组合式函数测试](#102-组合式函数测试)
  - [11. 内部 Mixins / Utils 体系](#11-内部-mixins--utils-体系)
    - [`src/_mixins/`（非主题部分）](#src_mixins非主题部分)
    - [`src/_utils/`](#src_utils)
  - [12. 编码风格杂项](#12-编码风格杂项)
  - [13. 快速参考卡片](#13-快速参考卡片)

---

## 1. 目录结构规范

```
src/<component>/
├── index.ts                       # 对外导出（component、props、types）
├── src/
│   ├── <Component>.tsx            # defineComponent 组件实现
│   ├── public-types.ts            # 公开类型、枚举常量数组
│   ├── interface.ts               # 内部类型/接口（可选）
│   ├── utils.ts                   # 组件专用工具函数（可选）
│   ├── styles/                    # 样式文件（CSS Modules 或普通 CSS）
│   └── <SubComponent>.tsx         # 子组件（可选）
├── tests/
│   ├── <Component>.spec.tsx       # Vitest 测试
│   ├── server.spec.tsx            # SSR 兼容测试
│   └── __snapshots__/
└── demos/                         # 文档示例
    ├── enUS/
    └── zhCN/
```

**关键约定**：组件实现文件 `PascalCase`，目录 `kebab-case`，子组件同级放置。

---

## 2. Props 声明规范

- 定义为独立的 `const` 对象，以 `as const` 结尾。
- 枚举类型**推荐使用常量数组衍生**（详见第 4 节）。
- v-model 同时提供 `onUpdateValue` 和 `'onUpdate:value'`。
- 废弃/私有属性标记 `@deprecated` / `@private`，私有键以 `internal` 开头。

```ts
// src/button/src/Button.tsx
export const buttonProps = {
  // 简单布尔
  block: Boolean,
  loading: Boolean,

  // 带默认值
  tag: { type: String as PropType<keyof HTMLElementTagNameMap>, default: 'button' },
  type: { type: String as PropType<ButtonType>, default: 'default' },
  size: { type: String as PropType<ButtonSize>, default: 'medium' },

  // 函数 / 多回调
  onClick: [Function, Array] as PropType<MaybeArray<(e: MouseEvent) => void>>,

  // v-model
  onUpdateValue: [Function, Array] as PropType<MaybeArray<OnUpdateValue>>,
  'onUpdate:value': [Function, Array] as PropType<MaybeArray<OnUpdateValue>>,
} as const
```

---

## 3. 组件实现规范（函数式）

### 3.1 基础模板

- `setup` 返回渲染函数，所有逻辑在闭包内完成，**禁止 `this`**。
- **禁止解构 Props**（`const { size } = props` 会破坏响应式）。使用 `props.xxx` 或 `toRefs`。
- 响应式数据命名：**普通 ref/computed 不带 `Ref` 后缀**，仅 DOM 元素引用使用 `xxxRef`。
- 推荐使用**语义前缀**区分数据类型（详见下节）。

```tsx
// src/button/src/Button.tsx
import { defineComponent, computed, ref, toRefs } from 'vue'
import { useConfig, useFormItem } from '../../_mixins'
import { call } from '../../_utils/vue/call'
import { buttonProps } from './props'

export default defineComponent({
  name: 'Button',
  props: buttonProps,
  slots: Object as SlotsType<ButtonSlots>,

  setup(props, { slots, attrs, expose }) {
    // ===== DOM 引用（唯一带 Ref 后缀） =====
    const selfElRef = ref<HTMLElement | null>(null)

    // ===== Config / FormItem =====
    const { mergedClsPrefix } = useConfig(props)   // 无 Ref 后缀
    const formItem = useFormItem(props)
    const { disabled: formDisabled } = toRefs(formItem || {}) // 若需要解构，必须用 toRefs

    // ===== 响应式状态（语义前缀） =====
    const isDisabled = computed(() => props.disabled || formDisabled?.value || false)
    const currentSize = computed(() => props.size || 'medium')

    // ===== 副作用执行 =====
    function doUpdateValue(val: string) {
      const { onUpdateValue, 'onUpdate:value': _onUpdateValue } = props
      if (onUpdateValue) call(onUpdateValue, val)
      if (_onUpdateValue) call(_onUpdateValue, val)
    }

    // ===== 事件处理 =====
    function handleClick(e: MouseEvent) {
      const { onClick } = props
      if (onClick) call(onClick, e)
    }

    // ===== 暴露实例方法 =====
    function focus() { selfElRef.value?.focus() }
    expose({ focus })

    // ===== 渲染函数 =====
    return () => {
      const clsPrefix = mergedClsPrefix.value
      const Component = props.tag || 'button'

      return (
        <Component
          ref={selfElRef}
          class={[
            `${clsPrefix}-button`,
            {
              [`${clsPrefix}-button--disabled`]: isDisabled.value,
              [`${clsPrefix}-button--block`]: props.block,
              [`${clsPrefix}-button--${currentSize.value}`]: true,
            },
          ]}
          {...attrs}   // 透传属性需手动合并（见第 9 节）
          onClick={handleClick}
          disabled={isDisabled.value}
        >
          {resolveWrappedSlot(slots.default, (children) => (
            <span class={`${clsPrefix}-button__content`}>{children}</span>
          ))}
        </Component>
      )
    }
  },
})
```

### 3.2 响应式命名语义前缀

为提高可读性，建议按类型添加前缀：

| 类别 | 前缀示例 | 说明 |
|------|----------|------|
| 布尔状态 | `is`、`has`、`can` | `isLoading`、`hasError`、`canSubmit` |
| 数值/字符串 | `current`、`raw`、`value` | `currentPage`、`rawText` |
| 列表/对象 | `state`、`list` 或直接名词 | `formState`、`options`（仍为 ref） |
| DOM 元素引用 | **必须** `xxxRef` | `inputRef`、`containerRef` |

```ts
// ✅ 推荐
const isLoading = ref(false)
const currentValue = ref('')
const userList = ref<User[]>([])
const inputRef = ref<HTMLInputElement>()

// ❌ 避免
const loading = ref(false)   // 不明是否为布尔
const value = ref('')        // 过于泛化
const refInput = ref()       // 滥用 Ref 后缀
```

---

## 4. 类型与枚举治理

**优先使用常量数组衍生联合类型**，便于测试和文档枚举。

```ts
// src/button/src/public-types.ts
export const buttonSizes = ['tiny', 'small', 'medium', 'large'] as const
export const buttonTypes = ['default', 'primary', 'success', 'warning', 'error'] as const
export const buttonVariants = ['solid', 'outline', 'ghost'] as const

export type ButtonSize = typeof buttonSizes[number]
export type ButtonType = typeof buttonTypes[number]
export type ButtonVariant = typeof buttonVariants[number]

// Props 中引用
size: { type: String as PropType<ButtonSize>, default: 'medium' },
type: { type: String as PropType<ButtonType>, default: 'default' },
variant: { type: String as PropType<ButtonVariant>, default: 'solid' },
```

**Slots 接口**：在 `public-types.ts` 中定义，作用域插槽必须明确参数类型。

```ts
export interface ButtonSlots {
  default?: () => VNode[]
  icon?: () => VNode[]
}

export interface InputSlots {
  count?: (props: { value: string; maxlength: number }) => VNode[]
  'clear-icon'?: () => VNode[]
}
```

---

## 5. Emits 规范

**不使用 `emits` 选项**，全部通过 Props 传递回调，使用 `call()` 工具调用。

- 同时支持数组和单个函数。
- v-model 双形式（`onUpdateValue` 和 `'onUpdate:value'`）均需调用。

```ts
// 声明
onUpdateValue: [Function, Array] as PropType<MaybeArray<OnUpdateValue>>,
'onUpdate:value': [Function, Array] as PropType<MaybeArray<OnUpdateValue>>,

// 调用
function doUpdateValue(val: string) {
  const { onUpdateValue, 'onUpdate:value': _onUpdateValue } = props
  if (onUpdateValue) call(onUpdateValue, val)
  if (_onUpdateValue) call(_onUpdateValue, val)
}
```

---

## 6. Slots 规范（含类型定义）

- `slots` 从 `setup` 第二个参数解构。
- 使用 `resolveWrappedSlot`、`resolveSlot`、`isSlotEmpty` 工具。
- **必须为所有 slot 定义类型**（`SlotsType<XxxSlots>`），作用域插槽参数需完整。

```tsx
setup(props, { slots }) {
  return () => (
    <div>
      {resolveWrappedSlot(slots.default, (children) => (
        <div class="wrapper">{children}</div>
      ))}
      {resolveSlot(slots.prefix, () => <span>Prefix</span>)}
      {slots.suffix?.()}
      {slots.count?.({ value: rawText.value, maxlength: props.maxlength })}
    </div>
  )
}
```

---

## 7. Expose 实例方法规范

当组件需要对外暴露方法（如 `focus`、`clear`、`validate`）时，在 `setup` 中使用 `expose` 显式暴露，**并同步定义实例类型**。

```ts
// src/input/src/public-types.ts
export interface InputInst {
  focus: () => void
  blur: () => void
  clear: () => void
}

// src/input/src/Input.tsx
setup(props, { slots, expose }) {
  const inputRef = ref<HTMLInputElement | null>(null)

  function focus() { inputRef.value?.focus() }
  function blur() { inputRef.value?.blur() }
  function clear() { /* ... */ }

  expose({ focus, blur, clear })

  return () => <input ref={inputRef} ... />
}
```

父组件通过 `ref` 获取实例时，类型为 `InputInst`。

---

## 8. 副作用与生命周期清理

- 所有 `setInterval`、`setTimeout`、`addEventListener`、`watch` 等副作用必须配套清理。
- 清理函数命名 `cleanupXxx`，在 `onBeforeUnmount` 中执行。
- 使用 `watch` 时，若不需要在组件卸载后继续，建议使用 `onWatcherCleanup`（Vue 3.5+）或手动停止。

```ts
import { onBeforeUnmount, watch } from 'vue'

let timer: ReturnType<typeof setTimeout> | null = null
function startTimer() { /* ... */ }

function cleanupTimer() {
  if (timer) { clearTimeout(timer); timer = null }
}

onBeforeUnmount(() => {
  cleanupTimer()
  // 如有其他监听器，一并清理
})

// 对于 watch，若需在卸载时停止，可存储 stop 句柄
const stopWatch = watch(source, () => { /* ... */ })
onBeforeUnmount(() => stopWatch())
```

---

## 9. 透传 Attributes 处理

组件根元素必须显式合并 `attrs`（来自 `setup` 第二个参数），避免丢失 `class`、`style` 等。

```tsx
setup(props, { attrs, slots }) {
  return () => {
    const clsPrefix = mergedClsPrefix.value
    return (
      <div
        {...attrs}   // 透传所有属性（包括 class、style）
        class={[
          `${clsPrefix}-component`,
          attrs.class,   // 合并外部 class
        ]}
        style={[styles.root, attrs.style]} // 合并 style
      >
        {/* ... */}
      </div>
    )
  }
}
```

**注意**：若使用 `attrs`，需排除组件已声明的 props（Vue 自动处理），仅透传未声明的属性。

---

## 10. 测试规范（含组合式函数测试）

### 10.1 组件测试

- 使用 `vitest` + `@vue/test-utils`。
- 每个测试用例后必须 `unmount()`。
- 快照测试使用 `toMatchSnapshot()`。

```ts
describe('n-button', () => {
  it('should work with `size` prop', () => {
    buttonSizes.forEach(size => {
      const wrapper = mount(NButton, { props: { size } })
      expect(wrapper.classes()).toContain(`n-button--${size}`)
      wrapper.unmount()
    })
  })
})
```

### 10.2 组合式函数测试

对于 `src/_mixins/` 中的自定义 composable，使用 `@vue/test-utils` 的 `setup` 模式或 `withSetup` 辅助。

```ts
// src/_mixins/__tests__/use-form-item.spec.ts
import { mount } from '@vue/test-utils'
import { defineComponent } from 'vue'
import { useFormItem } from '../use-form-item'

const TestComponent = defineComponent({
  setup() {
    const result = useFormItem({ disabled: true })
    return { result }
  },
  template: `<div></div>`,
})

it('should merge disabled prop', () => {
  const wrapper = mount(TestComponent)
  expect(wrapper.vm.result.disabled.value).toBe(true)
  wrapper.unmount()
})
```

**原则**：组合式函数测试应覆盖各种输入（props、inject 值）与输出（响应式状态、方法）。

---

## 11. 内部 Mixins / Utils 体系

保留原 `src/_mixins/` 和 `src/_utils/` 结构，但剔除主题相关模块（`use-theme`、`use-css-vars-class`）。

### `src/_mixins/`（非主题部分）
- `use-config.ts`：`clsPrefix`、`bordered`、RTL 等
- `use-form-item.ts`：`size`、`disabled`、`status`
- `use-locale.ts`：i18n
- `use-rtl.ts`：方向

### `src/_utils/`
- `vue/`：`call`、`createInjectionKey`、`resolveSlot`、`resolveWrappedSlot`、`isSlotEmpty`、`omit`、`keep`
- `naive/`：`warn`、`warnOnce`、`smallerSize`、`largerSize`
- `composable/`：`useIsComposing`、`useResize`、`useDeferredTrue`、`useLockHtmlScroll`
- `dom/`、`env/`、`ts/`

---

## 12. 编码风格杂项

- **Prop 定义**：`as const` 结尾。
- **格式**：`semi: false`、`singleQuote: true`、`printWidth: 80`、`trailingComma: none`。
- **Lint**：`@antfu/eslint-config`。
- **Git Commit**：Angular 风格（`feat(button): add ghost prop`）。
- **禁止**：`this`、`emits` 选项、`cssr`、`useTheme`。
- **响应式命名**：语义前缀，仅 DOM 引用可带 `Ref`。
- **Props 解构**：禁止直接解构，必须用 `props.xxx` 或 `toRefs`。

---

## 13. 快速参考卡片

```
┌──────────────────────────────────────────────────────────────┐
│ 目录结构        kebab-case / PascalCase / tests             │
│ Props 命名      camelCase, as const                         │
│ 响应式命名      语义前缀 + 不带 Ref（除 DOM ref）            │
│ 枚举类型        常量数组 as const 衍生联合                   │
│ 事件处理        handleXxx, doXxx, call()                    │
│ Slots           类型化 XxxSlots, resolveWrappedSlot         │
│ Expose          显式暴露，定义 Inst 类型                     │
│ 透传 Attributes  手动合并 {...attrs}, class, style          │
│ 副作用清理      onBeforeUnmount + cleanupXxx                │
│ 测试            vitest, mount, unmount, 快照                │
│ 组合式测试      用 mount(defineComponent) 或 withSetup      │
│ 风格           无 this, 无主题, 函数式                      │
└──────────────────────────────────────────────────────────────┘
```