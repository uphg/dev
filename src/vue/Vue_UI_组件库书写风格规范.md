# Naive UI Vue 组件书写风格规范 V3.1（函数式 · 无主题 · 官方 emit + 驼峰回调兼容）

> 基于 V3 迭代，**移除 kebab-case 形式的 Props 回调**（如 `'onUpdate:value'`），仅保留驼峰格式（如 `onUpdateValue`）作为 Props 兼容层。  
> **官方 emit 采用 kebab-case**（如 `'update:value'`），符合 Vue 原生规范。  
> 其他核心原则（无 `this`、无主题、响应式语义命名、枚举治理、目录扁平化等）保持不变。

---

## 目录

- [Naive UI Vue 组件书写风格规范 V3.1（函数式 · 无主题 · 官方 emit + 驼峰回调兼容）](#naive-ui-vue-组件书写风格规范-v31函数式--无主题--官方-emit--驼峰回调兼容)
  - [目录](#目录)
  - [1. 目录结构规范](#1-目录结构规范)
  - [2. Props 声明规范](#2-props-声明规范)
  - [3. 组件实现规范（函数式）](#3-组件实现规范函数式)
  - [4. 类型与枚举治理](#4-类型与枚举治理)
  - [5. Emits 规范（官方 emit + 驼峰回调兼容）](#5-emits-规范官方-emit--驼峰回调兼容)
  - [6. Slots 规范](#6-slots-规范)
  - [7. Expose 实例方法规范](#7-expose-实例方法规范)
  - [8. 副作用与生命周期清理](#8-副作用与生命周期清理)
  - [9. 透传 Attributes 处理](#9-透传-attributes-处理)
  - [10. 测试规范（含组合式函数测试）](#10-测试规范含组合式函数测试)
  - [11. Hooks 与 Internal 工具体系](#11-hooks-与-internal-工具体系)
  - [12. 编码风格杂项](#12-编码风格杂项)
  - [13. 快速参考卡片](#13-快速参考卡片)

---

## 1. 目录结构规范

组件目录扁平化，所有文件直接置于组件根目录下：

```
src/component/<component>/
├── index.ts                   # 对外导出（component、props、types）
├── <Component>.tsx            # defineComponent 组件实现（函数式）
├── public-types.ts            # 公开类型、枚举常量数组
├── interface.ts               # 内部类型/接口（可选）
├── utils.ts                   # 组件专用工具函数（可选）
└── tests/
    ├── <Component>.spec.tsx   # Vitest 测试
    ├── server.spec.tsx        # SSR 兼容测试
    └── __snapshots__/         # 快照产物
```

**命名约定**：组件文件 `PascalCase`，目录 `kebab-case`，子组件可同目录放置。

---

## 2. Props 声明规范

Props 定义为独立的 `const` 对象，以 `as const` 结尾。  
枚举类型由常量数组衍生（详见第 4 节）。  
**事件回调仅提供驼峰格式**（如 `onUpdateValue`），不再提供 `'onUpdate:value'` 这类 kebab-case 形式。

```ts
// src/component/button/Button.tsx
export const buttonProps = {
  block: Boolean,
  loading: Boolean,
  tag: { type: String as PropType<keyof HTMLElementTagNameMap>, default: 'button' },
  type: { type: String as PropType<ButtonType>, default: 'default' },
  size: { type: String as PropType<ButtonSize>, default: 'medium' },

  // 事件回调（驼峰格式，与 emits 互补）
  onClick: [Function, Array] as PropType<MaybeArray<(e: MouseEvent) => void>>,
  onUpdateValue: [Function, Array] as PropType<MaybeArray<OnUpdateValue>>,
} as const
```

> **注意**：不再声明 `'onUpdate:value'`，回调仅通过驼峰 `onUpdateValue` 传递。

---

## 3. 组件实现规范（函数式）

- `setup` 直接返回渲染函数，**禁止 `this`**。
- **禁止直接解构 Props**，使用 `props.xxx` 或 `toRefs`。
- 响应式数据命名：**语义前缀** + 无 `Ref` 后缀（仅 DOM 引用可带 `Ref`）。
- 事件触发时：**先执行官方 `emit`（kebab-case），再调用 `props.onXxx`（驼峰）**。

```tsx
// src/component/button/Button.tsx
import { defineComponent, computed, ref, toRefs } from 'vue'
import { useConfig, useFormItem } from '../../hooks'
import { call } from '../../internal/call'
import { buttonProps } from './props'

export default defineComponent({
  name: 'Button',
  props: buttonProps,
  emits: {
    'update:value': (val: any) => true,
    click: (e: MouseEvent) => true,
  },
  slots: Object as SlotsType<ButtonSlots>,

  setup(props, { slots, attrs, emit, expose }) {
    const selfElRef = ref<HTMLElement | null>(null)
    const { mergedClsPrefix } = useConfig(props)
    const formItem = useFormItem(props)
    const { disabled: formDisabled } = toRefs(formItem || {})

    const isDisabled = computed(() => props.disabled || formDisabled?.value || false)

    function doUpdateValue(val: string) {
      // 1) 官方 emit（kebab-case）
      emit('update:value', val)
      // 2) Props 回调兼容（驼峰）
      const { onUpdateValue } = props
      if (onUpdateValue) call(onUpdateValue, val)
    }

    function handleClick(e: MouseEvent) {
      emit('click', e)
      const { onClick } = props
      if (onClick) call(onClick, e)
    }

    function focus() { selfElRef.value?.focus() }
    expose({ focus })

    return () => {
      const clsPrefix = mergedClsPrefix.value
      return (
        <button
          ref={selfElRef}
          class={[`${clsPrefix}-button`, { /* ... */ }]}
          {...attrs}
          onClick={handleClick}
        >
          {resolveWrappedSlot(slots.default, (children) => (
            <span class={`${clsPrefix}-button__content`}>{children}</span>
          ))}
        </button>
      )
    }
  },
})
```

---

## 4. 类型与枚举治理

使用 `as const` 数组衍生联合类型，便于复用和测试。

```ts
// public-types.ts
export const buttonSizes = ['tiny', 'small', 'medium', 'large'] as const
export const buttonTypes = ['default', 'primary', 'success', 'warning', 'error'] as const
export type ButtonSize = typeof buttonSizes[number]
export type ButtonType = typeof buttonTypes[number]

// Slots 接口
export interface ButtonSlots {
  default?: () => VNode[]
  icon?: () => VNode[]
}
```

---

## 5. Emits 规范（官方 emit + 驼峰回调兼容）

**核心规则**：

1. **官方 `emits` 选项**：使用 kebab-case 命名（如 `'update:value'`、`'click'`），符合 Vue 原生规范。
2. **Props 回调兼容**：每个事件同时提供**驼峰格式**的 Props 回调（如 `onUpdateValue`、`onClick`），但**不再提供 kebab-case 形式的回调**（如 `'onUpdate:value'`）。
3. **触发顺序**：先执行 `emit`，再调用 `props.onXxx`。

**好处**：
- 模板中使用 `v-model:value` 或 `@update:value` 符合 Vue 习惯。
- JSX 中可通过 `onUpdateValue` 传递回调（驼峰，符合 Props 命名规范）。
- 避免了 Props 中同时存在 `onUpdateValue` 和 `'onUpdate:value'` 的冗余。

```ts
// 1. 声明 emits（kebab-case）
export const buttonEmits = {
  click: (e: MouseEvent) => true,
  'update:value': (val: any) => true,
}

// 2. Props 中仅声明驼峰回调（无 kebab-case）
export const buttonProps = {
  onClick: [Function, Array] as PropType<MaybeArray<(e: MouseEvent) => void>>,
  onUpdateValue: [Function, Array] as PropType<MaybeArray<OnUpdateValue>>,
}

// 3. 触发时双发（先 emit 后 call）
function handleClick(e: MouseEvent) {
  emit('click', e)                // 官方 emit（kebab-case）
  const { onClick } = props
  if (onClick) call(onClick, e)  // 回调兼容（驼峰）
}

function doUpdateValue(val: string) {
  emit('update:value', val)       // 官方 emit（kebab-case）
  const { onUpdateValue } = props
  if (onUpdateValue) call(onUpdateValue, val)  // 回调兼容（驼峰）
}
```

**使用示例**：

```tsx
// 模板方式（自动绑定 emit）
<NButton v-model:value="text" @click="handleClick" />

// JSX 方式（通过 Props 传递回调）
<NButton onUpdateValue={(val) => (text = val)} onClick={handleClick} />
```

---

## 6. Slots 规范

- `slots` 从 `setup` 第二个参数解构。
- 使用 `resolveWrappedSlot`、`resolveSlot`、`isSlotEmpty` 工具。
- 必须为所有 slot 定义类型（`SlotsType<XxxSlots>`），作用域插槽参数明确。

```tsx
setup(props, { slots }) {
  return () => (
    <div>
      {resolveWrappedSlot(slots.default, (children) => (
        <div class="wrapper">{children}</div>
      ))}
      {slots.prefix?.()}
      {slots.count?.({ value: rawText.value, maxlength: props.maxlength })}
    </div>
  )
}
```

---

## 7. Expose 实例方法规范

当组件需要暴露实例方法时，使用 `expose` 显式暴露，并定义对应的 `Inst` 类型。

```ts
// public-types.ts
export interface InputInst {
  focus: () => void
  blur: () => void
  clear: () => void
}

// 组件中
setup(props, { expose }) {
  const inputRef = ref<HTMLInputElement | null>(null)
  function focus() { inputRef.value?.focus() }
  function blur() { inputRef.value?.blur() }
  function clear() { /* ... */ }
  expose({ focus, blur, clear })
}
```

---

## 8. 副作用与生命周期清理

所有副作用（定时器、事件监听、`watch` 等）必须配套清理，清理函数命名为 `cleanupXxx`，在 `onBeforeUnmount` 中执行。

```ts
import { onBeforeUnmount, watch } from 'vue'

let timer: ReturnType<typeof setTimeout> | null = null
function cleanupTimer() {
  if (timer) { clearTimeout(timer); timer = null }
}
onBeforeUnmount(cleanupTimer)

// watch 停止
const stopWatch = watch(source, () => { /* ... */ })
onBeforeUnmount(stopWatch)
```

---

## 9. 透传 Attributes 处理

手动合并 `attrs`，避免丢失外部传入的 `class`、`style` 等。

```tsx
setup(props, { attrs }) {
  return () => (
    <div
      {...attrs}
      class={[`${clsPrefix}-component`, attrs.class]}
      style={[rootStyle, attrs.style]}
    >
      {/* ... */}
    </div>
  )
}
```

---

## 10. 测试规范（含组合式函数测试）

- 使用 `vitest` + `@vue/test-utils`。
- 每个测试用例后必须 `unmount()`。
- 快照测试使用 `toMatchSnapshot()`。
- 组合式函数测试：使用 `mount(defineComponent({ setup() { /* use hook */ } }))` 或借助 `withSetup` 辅助。

```ts
describe('n-button', () => {
  it('emits click event', async () => {
    const wrapper = mount(NButton)
    await wrapper.trigger('click')
    expect(wrapper.emitted('click')).toBeTruthy()
  })

  it('supports onClick prop callback', async () => {
    const onClick = vi.fn()
    const wrapper = mount(NButton, { props: { onClick } })
    await wrapper.trigger('click')
    expect(onClick).toHaveBeenCalled()
  })

  it('supports v-model via onUpdateValue prop', async () => {
    const onUpdateValue = vi.fn()
    const wrapper = mount(NInput, { props: { onUpdateValue } })
    // 模拟输入变化...
    expect(onUpdateValue).toHaveBeenCalled()
  })
})
```

---

## 11. Hooks 与 Internal 工具体系

**移除原有的 `_mixins` 和 `_utils` 细粒度分类**，统一为：

- **`src/hooks/`**：存放所有组合式函数（Composables），如：
  - `useConfig.ts`：ConfigProvider 注入（`clsPrefix`、`bordered`、RTL）
  - `useFormItem.ts`：FormItem 注入（`size`、`disabled`、`status`）
  - `useLocale.ts`：国际化
  - `useRtl.ts`：RTL 方向
  - 以及自定义的 `useDeferredTrue`、`useResize` 等

- **`src/internal/`**：存放内部工具函数，不再细分二级目录，按功能直接导出：
  - `call.ts`：安全调用回调
  - `createInjectionKey.ts`：创建注入键
  - `resolveSlot.ts` / `resolveWrappedSlot.ts` / `isSlotEmpty.ts`
  - `omit.ts` / `keep.ts` 等对象工具
  - `warn.ts` / `warnOnce.ts`
  - 通用类型工具（`types.ts`）

**原则**：非必要不细分，扁平化便于查找。

---

## 12. 编码风格杂项

- **Prop 定义**：`as const` 结尾。
- **格式**：`semi: false`、`singleQuote: true`、`printWidth: 80`、`trailingComma: none`。
- **Lint**：`@antfu/eslint-config`。
- **Git Commit**：Angular 风格。
- **禁止**：`this`、`cssr`、`useTheme`、直接解构 Props。
- **响应式命名**：语义前缀 + 不带 `Ref`（除 DOM 引用）。
- **事件机制**：官方 `emits`（kebab-case）+ 驼峰 Props 回调（双发，先 `emit` 后 `call`）。
- **Props 回调**：仅提供驼峰格式（如 `onUpdateValue`），不再提供 kebab-case（如 `'onUpdate:value'`）。

---

## 13. 快速参考卡片

```
┌──────────────────────────────────────────────────────────────┐
│ 目录结构        组件根目录扁平（无 src 子目录）             │
│ Props 命名      camelCase, as const                         │
│ 响应式命名      语义前缀 + 不带 Ref（除 DOM ref）            │
│ 枚举类型        常量数组 as const 衍生联合                   │
│ 事件机制        官方 emits（kebab-case）+ 驼峰回调（双发）   │
│                 先 emit 后 call(props.onXxx)                │
│ Slots           类型化 XxxSlots, resolveWrappedSlot         │
│ Expose          显式暴露，定义 Inst 类型                     │
│ 透传 Attributes 手动合并 {...attrs}, class, style          │
│ 副作用清理      onBeforeUnmount + cleanupXxx                │
│ 测试            vitest, mount, unmount, 快照                │
│ Hooks           src/hooks/ 所有组合式函数                    │
│ Internal        src/internal/ 扁平工具函数                   │
│ 风格           无 this, 无主题, 函数式                      │
│ Props 回调      仅驼峰（onXxx），无 kebab-case              │
└──────────────────────────────────────────────────────────────┘
```