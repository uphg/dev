## 概述

**FSD** 是一种用于构建前端应用的架构方法论，核心是**通过规则和约定组织代码**，旨在提升项目的**可理解性**和**面对需求变更的稳定性**。

### 核心特点

- **框架无关**：不限制使用的编程语言、UI框架或状态管理库。
- **适用广泛**：适用于任何规模的前端项目（Web、移动端、桌面端）。
- **渐进式采用**：支持在现有项目中逐步迁移。
- **工具链支持**：提供 Linter、CLI 生成器等工具。

## 核心概念

### 1. 分层（Layers）

**7个标准化分层**（从上至下，依赖方向严格向下）：

1. **App** - 应用初始化（路由、全局样式、Provider）。
2. **Processes** - ❌ 已弃用。
3. **Pages** - 完整页面或大型路由块。
4. **Widgets** - 大型自包含功能/UI块（通常实现完整用例）。
5. **Features** - **可复用的完整产品功能**（为用户带来业务价值）。
6. **Entities** - 业务实体（如 `user`、`product`）。
7. **Shared** - 可复用功能（尤其与业务无关的通用逻辑）。

> ⚠️ **App** 和 **Shared** 层不包含切片，直接由分段组成。

### 2. 切片（Slices）

- **作用**：按**业务域**划分代码（例如 `article-reader`、`feed`）。
- **规则**：
  - 同一层的切片**不能相互引用**。
  - 必须通过**公共API（如 `index.ts`）** 暴露模块。

### 3. 分段（Segments）

- **作用**：按**技术用途**组织切片内的代码。
- **常用分段**：
  - `ui` - UI组件、样式
  - `api` - 后端交互
  - `model` - 数据模型、业务逻辑
  - `lib` - 内部库代码
  - `config` - 配置、特性开关

## 优点

1. **一致性** - 标准化结构降低团队 onboarding 成本。
2. **变更稳定** - 分层隔离使修改局部化，减少意外影响。
3. **可控复用** - 平衡 DRY 原则与实用性。
4. **业务导向** - 使用业务语言命名，便于理解产品需求。

## 分层结构

### 1. Shared层

- 包含通用的工具、API客户端、UI组件等，与业务无关。
- 例如：`shared/ui` 包含Button、Input等基础组件；`shared/api` 包含HTTP客户端；`shared/lib` 包含日期处理库等。

### 2. Entities层

- 业务实体，如User、Post、Comment、Notification。
- 每个实体是一个切片，包含该实体的模型、API、UI组件（仅展示，无业务逻辑）等。
- 例如：`entities/user` 包含用户模型、获取用户信息的API、用户头像组件等。

### 3. Features层

- 可复用的用户交互功能，通常涉及多个实体。
- 例如：`features/auth` 包含登录、注册功能；`features/comments` 包含评论功能（包括评论列表、发表评论）；`features/likes` 包含点赞功能。

### 4. Widgets层

- 组合多个Feature和Entity形成的较大UI块，通常是一个完整的用例。
- 例如：`widgets/post-card` 包含显示帖子、点赞、评论的入口；`widgets/sidebar` 包含侧边栏，显示用户信息和通知。

### 5. Pages层

- 完整页面，由多个Widget和Feature组成。
- 例如：`pages/feed` 首页，包含发布帖子的表单和帖子列表；`pages/profile` 用户个人页面。

### 6. App层

- 应用级别的配置：路由、全局状态、主题等。
- 例如：`app/routes` 定义所有路由；`app/store` 配置Redux store；`app/styles` 全局样式。

### 7. Processes层（已弃用，但为了完整列出）

- 复杂跨页面流程，例如OAuth登录流程。现在推荐放在`features`或`app`中。

## 示例代码结构

以下是一个社交媒体应用的目录结构示例，展示了完整的分层：

```
src/
├── app/                    # App层 - 应用基础设施
│   ├── routes/            # 路由配置
│   ├── providers/         # 全局Provider（主题、状态）
│   ├── store/             # 全局状态配置
│   ├── styles/            # 全局样式
│   └── entrypoint.tsx     # 应用入口
├── processes/             # ❌ Processes层（已弃用）
├── pages/                 # Pages层 - 完整页面
│   ├── feed/              # 动态流页面
│   ├── profile/           # 用户资料页
│   ├── messages/          # 消息页面
│   └── settings/          # 设置页面
├── widgets/               # Widgets层 - 大型UI块
│   ├── user-sidebar/      # 用户侧边栏（含关注推荐）
│   ├── trending-topics/   # 热门话题组件
│   ├── notification-bell/ # 通知铃铛（含下拉面板）
│   └── post-composer/     # 发帖编辑器
├── features/              # Features层 - 可复用业务功能
│   ├── auth/              # 身份验证功能
│   ├── post-interactions/ # 帖子交互（点赞、评论、分享）
│   ├── follow-system/     # 关注系统
│   ├── search/            # 搜索功能
│   └── notifications/     # 通知系统
├── entities/              # Entities层 - 业务实体
│   ├── user/              # 用户实体
│   ├── post/              # 帖子实体
│   ├── comment/           # 评论实体
│   ├── message/           # 消息实体
│   └── notification/      # 通知实体
└── shared/                # Shared层 - 通用基础设施
    ├── ui/                # UI组件库
    ├── api/               # API客户端
    ├── lib/               # 工具库
    ├── config/            # 配置
    └── types/             # 通用类型定义
```


更详细的层级关系版本

::: details 查看

```
src/
├── app/                    # App层
│   ├── routes/            # 路由配置
│   ├── store/             # 全局状态配置
│   ├── styles/            # 全局样式
│   └── entrypoint/        # 应用入口
├── pages/                 # Pages层
│   ├── feed/              # 首页
│   │   ├── ui/            # 页面UI
│   │   ├── api/           # 页面特定API（如获取feed流）
│   │   └── index.ts       # 公共API
│   ├── profile/           # 用户个人页
│   │   ├── ui/
│   │   ├── api/
│   │   └── index.ts
│   └── ...                # 其他页面
├── widgets/               # Widgets层
│   ├── post-card/         # 帖子卡片
│   │   ├── ui/            # 帖子卡片的UI
│   │   ├── model/         # 帖子卡片的状态逻辑（如点赞、收藏）
│   │   └── index.ts
│   ├── sidebar/           # 侧边栏
│   │   ├── ui/
│   │   ├── model/         # 侧边栏状态（如通知数量）
│   │   └── index.ts
│   └── ...                # 其他Widgets
├── features/              # Features层
│   ├── auth/              # 认证功能
│   │   ├── ui/            # 登录、注册表单
│   │   ├── api/           # 认证API
│   │   ├── model/         # 认证状态（如当前用户）
│   │   └── index.ts
│   ├── comments/          # 评论功能
│   │   ├── ui/            # 评论列表、评论表单
│   │   ├── api/           # 评论API
│   │   ├── model/         # 评论状态（如评论列表、发表评论）
│   │   └── index.ts
│   ├── likes/             # 点赞功能
│   │   ├── ui/            # 点赞按钮
│   │   ├── api/           # 点赞API
│   │   ├── model/         # 点赞状态
│   │   └── index.ts
│   └── ...                # 其他Features
├── entities/              # Entities层
│   ├── user/              # 用户实体
│   │   ├── ui/            # 用户相关UI（如用户头像）
│   │   ├── api/           # 用户API（如获取用户信息）
│   │   ├── model/         # 用户模型（如User类型定义）
│   │   └── index.ts
│   ├── post/              # 帖子实体
│   │   ├── ui/            # 帖子展示组件（纯UI，无业务逻辑）
│   │   ├── api/           # 帖子API（如获取帖子）
│   │   ├── model/         # 帖子模型
│   │   └── index.ts
│   ├── comment/           # 评论实体
│   │   ├── ui/            # 评论展示组件
│   │   ├── api/           # 评论API
│   │   ├── model/         # 评论模型
│   │   └── index.ts
│   └── ...                # 其他Entities
└── shared/                # Shared层
    ├── ui/                # 通用UI组件（如Button、Input、Modal）
    ├── api/               # 通用API客户端（如axios实例、请求拦截）
    ├── lib/               # 通用库（如日期格式化、字符串处理）
    ├── config/            # 全局配置（如环境变量）
    └── ...                # 其他共享资源
```

:::

## 依赖关系

- **App层**可以依赖所有下层。
- **Pages层**可以依赖Widgets、Features、Entities、Shared。
- **Widgets层**可以依赖Features、Entities、Shared。
- **Features层**可以依赖Entities、Shared。
- **Entities层**可以依赖Shared。
- **Shared层**不依赖任何其他层。

## 示例：点赞功能的数据流

1. **Entities/Post** 定义了帖子的数据类型和获取帖子的API。
2. **Features/Likes** 定义了点赞的UI（按钮）和业务逻辑（发送点赞请求、更新本地状态）。
3. **Widgets/Post-card** 组合了Entities/Post的UI组件和Features/Likes的点赞按钮，并处理帖子卡片内部的逻辑。
4. **Pages/Feed** 使用Widgets/Post-card来展示帖子列表。

## 总结

通过这样的分层，我们实现了：

- **可维护性**：每层职责明确，修改隔离。
- **可复用性**：Features和Entities可以在多个Widgets和Pages中复用。
- **可测试性**：每层可以独立测试。

虽然这个例子比之前的Conduit示例复杂，但它展示了FSD在大型项目中的优势。在实际项目中，可以根据需要选择使用哪些层，不必强制使用所有层。