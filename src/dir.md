## 整体设计思路

- **MECE原则**：相互独立，完全穷尽，避免知识点交叉混乱
- **实战导向**：每个模块都有"理论+实践+案例"三层结构
- **面试友好**：便于快速定位和输出，面试前能快速回顾
- **持续演进**：预留扩展空间，随技术发展迭代

---

## 完整目录结构

```
frontend-knowledge-base/
├── 00-INDEX/                          # 索引与导航
│   ├── README.md                      # 知识库总览、学习路线图
│   ├── quick-reference/               # 速查表
│   │   ├── git-commands.md
│   │   ├── docker-commands.md
│   │   └── regex-cheatsheet.md
│   └── interview-prep/                # 面试准备专题
│       ├── 25k-level-requirements.md  # 对标要求清单
│       └── common-questions.md        # 高频面试题汇总
│
├── 01-FOUNDATION/                     # 地基：计算机基础与开发环境
│   ├── network/
│   │   ├── http-https.md
│   │   ├── websocket.md
│   │   ├── cors.md
│   │   └── cases/                     # 实战案例
│   │       └── cors-troubleshooting.md
│   ├── browser/
│   │   ├── rendering-principles.md
│   │   ├── event-loop.md
│   │   ├── memory-management.md
│   │   └── cases/
│   │       └── memory-leak-debug.md
│   ├── dev-tools/
│   │   ├── git-workflow.md
│   │   ├── pnpm-monorepo.md
│   │   ├── webpack-deep-dive.md
│   │   ├── vite-principles.md
│   │   └── cases/
│   │       └── custom-webpack-plugin.md
│   └── design-patterns/
│       ├── observer-vs-pubsub.md
│       ├── singleton.md
│       ├── factory.md
│       └── cases/
│           └── event-bus-implementation.md
│
├── 02-LANGUAGE/                       # 语言核心：JS/TS
│   ├── javascript/
│   │   ├── prototype-chain.md
│   │   ├── closure-scope.md
│   │   ├── promise-async.md
│   │   ├── es6-plus.md
│   │   └── cases/
│   │       ├── promise-implementation.md
│   │       └── deep-clone-implementation.md
│   ├── typescript/
│   │   ├── type-system.md
│   │   ├── generics.md
│   │   ├── utility-types.md
│   │   ├── type-challenges/           # 类型体操专项
│   │   │   ├── easy.md
│   │   │   ├── medium.md
│   │   │   └── hard.md
│   │   └── cases/
│   │       └── type-safe-api-client.md
│   └── functional-programming/
│       ├── pure-functions.md
│       ├── curry-compose.md
│       └── immutable-data.md
│
├── 03-VUE/                           # Vue 生态
│   ├── core/
│   │   ├── reactivity.md             # 响应式原理（含 Proxy 手写）
│   │   ├── virtual-dom.md            # 虚拟 DOM 与 Diff
│   │   ├── compiler.md               # 编译原理
│   │   └── lifecycle.md
│   ├── ecosystem/
│   │   ├── pinia.md                  # 状态管理（含原理）
│   │   ├── vue-router.md
│   │   └── nuxt.md                   # SSR 实践
│   ├── performance/
│   │   ├── optimization-guide.md
│   │   └── cases/
│   │       └── large-list-optimization.md
│   └── source-code/
│       ├── reactivity-source.md      # 源码阅读笔记
│       └── runtime-core-source.md
│
├── 04-REACT/                         # React 生态
│   ├── core/
│   │   ├── jsx-fiber.md              # Fiber 架构
│   │   ├── reconciliation.md         # 调和算法
│   │   ├── hooks.md                  # Hooks 原理（链表）
│   │   └── concurrent-features.md    # 并发特性
│   ├── ecosystem/
│   │   ├── zustand.md                # 轻量状态管理
│   │   ├── redux-toolkit.md
│   │   ├── react-router.md
│   │   └── nextjs.md                 # App Router 实践
│   ├── performance/
│   │   ├── optimization-guide.md
│   │   └── cases/
│   │       └── infinite-scroll-optimization.md
│   └── source-code/
│       ├── hooks-implementation.md   # 手写 useState/useEffect
│       └── fiber-source.md
│
├── 05-COMPONENT-LIB/                 # 组件库封装（核心亮点）
│   ├── design-principles/
│   │   ├── atomic-design.md
│   │   ├── api-design.md             # Props/Slots/Events 设计规范
│   │   └── accessibility.md          # a11y 无障碍
│   ├── styling/
│   │   ├── css-modules.md
│   │   ├── tailwind-strategy.md
│   │   ├── theme-system.md           # 动态换肤方案
│   │   └── cases/
│   │       └── dark-mode-implementation.md
│   ├── engineering/
│   │   ├── monorepo-structure.md     # pnpm workspace 结构
│   │   ├── build-config.md           # Vite/Rollup 配置
│   │   ├── tree-shaking.md
│   │   ├── documentation.md          # Storybook/Vitepress
│   │   └── cases/
│   │       └── component-publish-workflow.md
│   ├── component-cases/              # 具体组件实现
│   │   ├── button.md                 # 最简但最全
│   │   ├── modal.md                  # 传送门、动画
│   │   ├── table.md                  # 虚拟滚动
│   │   ├── form.md                   # 表单受控/非受控
│   │   └── virtual-list.md
│   └── advanced/
│       ├── renderless-components.md  # 无渲染组件
│       └── headless-ui.md
│
├── 06-CROSS-PLATFORM/                # 跨端框架
│   ├── mini-program/
│   │   ├── principles.md             # 小程序双线程模型
│   │   ├── taro-deep-dive.md         # 编译时框架原理
│   │   └── cases/
│   │       └── taro-plugin-development.md
│   ├── react-native/
│   │   ├── bridge-jsi.md             # 通信机制
│   │   ├── new-architecture.md       # Fabric/TurboModules
│   │   └── cases/
│   │       └── native-module-implementation.md
│   ├── flutter/                      # 可选
│   │   └── vs-react-native.md
│   └── electron/
│       └── desktop-app-development.md
│
├── 07-ENGINEERING/                   # 工程化架构（25k 核心亮点）
│   ├── architecture/
│   │   ├── monorepo-practice.md      # 大型项目 Monorepo 设计
│   │   ├── micro-frontend.md         # 微前端方案对比
│   │   └── low-code/                 # 低代码平台
│   │       ├── renderer-design.md
│   │       ├── editor-design.md
│   │       └── schema-protocol.md
│   ├── quality/
│   │   ├── eslint-custom-rules.md
│   │   ├── git-hooks.md
│   │   ├── unit-test.md              # Vitest/Jest
│   │   ├── e2e-test.md               # Playwright/Cypress
│   │   └── cases/
│   │       └── test-strategy-for-large-project.md
│   ├── ci-cd/
│   │   ├── github-actions.md
│   │   ├── docker-deployment.md
│   │   └── cases/
│   │       └── automated-release-flow.md
│   └── monitoring/
│       ├── sentry-integration.md
│       ├── performance-monitoring.md
│       └── cases/
│           └── frontend-sdk-design.md
│
├── 08-PERFORMANCE/                   # 性能优化实战（面试高频）
│   ├── metrics/
│   │   ├── core-web-vitals.md        # FCP/LCP/FID/CLS
│   │   └── performance-budget.md
│   ├── optimization/
│   │   ├── code-splitting.md
│   │   ├── lazy-loading.md
│   │   ├── image-optimization.md
│   │   ├── caching-strategy.md
│   │   └── cases/
│   │       ├── ecommerce-performance.md
│   │       └── ssr-performance.md
│   └── tools/
│       ├── lighthouse.md
│       ├── webpack-bundle-analyzer.md
│       └── chrome-devtools.md
│
├── 09-AI-EMPOWER/                   # AI 赋能研发（加分项）
│   ├── ai-tools/
│   │   ├── copilot-best-practices.md
│   │   ├── cursor-tips.md
│   │   └── prompt-engineering.md
│   ├── ai-integration/
│   │   ├── llm-api-calling.md
│   │   ├── ai-chatbot.md
│   │   ├── image-recognition.md
│   │   └── cases/
│   │       └── ai-powered-search.md
│   └── ai-for-dev/
│       ├── ai-code-review.md
│       ├── ai-test-generation.md
│       └── future-trends.md
│
├── 10-PROJECTS/                     # 项目实战与复盘（面试核心素材）
│   ├── project-a/                   # 按实际项目命名
│   │   ├── overview.md              # 项目背景、技术栈、我的角色
│   │   ├── architecture.md          # 架构设计图
│   │   ├── challenges.md            # 技术难点与解决方案（STAR 法则）
│   │   ├── performance.md           # 性能优化记录
│   │   └── reflection.md            # 复盘：如果重来会如何改进
│   ├── project-b/
│   │   └── ...
│   └── open-source-contributions/   # 开源贡献记录
│       └── xxx-pr.md
│
├── 11-SOFT-SKILLS/                  # 软技能与职业发展
│   ├── communication/
│   │   ├── requirement-gathering.md
│   │   ├── cross-team-collaboration.md
│   │   └── technical-proposal.md
│   ├── management/
│   │   ├── task-decomposition.md
│   │   ├── code-review-guide.md
│   │   └── mentoring-junior.md
│   └── career/
│       ├── resume-tips.md
│       ├── interview-stories.md
│       └── salary-negotiation.md
├── 12-BACKEND-BFF/                    # 新增：后端能力（BFF 层）
│   ├── README.md                      # BFF 概念、选型指南
│   │
│   ├── nodejs/                        # Node.js 生态（主力）
│   │   ├── core/
│   │   │   ├── event-loop.md          # 深入理解 libuv
│   │   │   ├── stream.md              # Stream 原理与实践
│   │   │   ├── buffer.md
│   │   │   ├── cluster.md             # 多进程
│   │   │   └── memory-leak.md         # 内存泄漏排查
│   │   │
│   │   ├── frameworks/
│   │   │   ├── express.md             # 中间件机制
│   │   │   ├── koa.md                 # 洋葱模型实现
│   │   │   ├── nestjs/                # 企业级框架（推荐）
│   │   │   │   ├── overview.md
│   │   │   │   ├── dependency-injection.md
│   │   │   │   ├── modules.md
│   │   │   │   ├── guards-interceptors.md
│   │   │   │   └── cases/
│   │   │   │       └── restful-api-design.md
│   │   │   └── fastify.md             # 高性能场景
│   │   │
│   │   ├── database/
│   │   │   ├── mysql.md               # SQL 基础与优化
│   │   │   ├── mongodb.md             # NoSQL 场景
│   │   │   ├── redis.md               # 缓存策略
│   │   │   ├── orm/                   # ORM/ODM
│   │   │   │   ├── prisma.md          # 推荐
│   │   │   │   ├── typeorm.md
│   │   │   │   └── mongoose.md
│   │   │   └── cases/
│   │   │       └── complex-query-optimization.md
│   │   │
│   │   ├── api-design/
│   │   │   ├── restful.md
│   │   │   ├── graphql.md
│   │   │   ├── websocket-server.md
│   │   │   └── cases/
│   │   │       └── bff-aggregation.md  # BFF 接口聚合实战
│   │   │
│   │   ├── authentication/
│   │   │   ├── jwt.md                  # 原理与最佳实践
│   │   │   ├── oauth2.md               # OAuth 2.0 / OIDC
│   │   │   ├── session.md
│   │   │   └── rbac.md                 # 权限控制
│   │   │
│   │   ├── security/
│   │   │   ├── xss-csrf.md
│   │   │   ├── sql-injection.md
│   │   │   ├── rate-limit.md
│   │   │   └── helmet.md
│   │   │
│   │   └── cases/                      # Node.js 实战案例
│   │       ├── bff-for-micro-frontend.md  # 微前端 BFF
│   │       ├── ssr-render-service.md     # SSR 渲染服务
│   │       ├── api-gateway.md            # 简单网关实现
│   │       └── file-upload-service.md
│   │
│   ├── golang/                         # Go 语言（高性能场景）
│   │   ├── core/
│   │   │   ├── goroutine.md            # 协程原理
│   │   │   ├── channel.md              # CSP 并发模型
│   │   │   ├── interface.md            # 接口与多态
│   │   │   └── memory-model.md
│   │   │
│   │   ├── web-frameworks/
│   │   │   ├── gin.md                  # 主流框架
│   │   │   ├── echo.md
│   │   │   └── fiber.md                # 高性能场景
│   │   │
│   │   ├── database/
│   │   │   ├── gorm.md                 # ORM
│   │   │   └── sqlx.md
│   │   │
│   │   └── cases/
│   │       ├── high-concurrency-api.md  # 高并发接口
│   │       └── websocket-chat.md
│   │
│   └── python/                         # Python 脚本能力（自动化）
│       ├── scripting/
│       │   ├── argparse.md             # 命令行参数
│       │   ├── file-operations.md      # 文件处理
│       │   ├── subprocess.md           # 系统调用
│       │   └── cases/
│       │       ├── log-parser.md       # 日志分析脚本
│       │       └── data-migration.md   # 数据迁移脚本
│       │
│       ├── automation/
│       │   ├── selenium.md             # Web 自动化
│       │   ├── requests.md             # HTTP 请求
│       │   └── cases/
│       │       └── auto-deploy-script.md
│       │
│       └── data-processing/
│           ├── pandas.md               # 数据分析
│           ├── json-xml-convert.md
│           └── cases/
│               └── performance-report-generator.md
│
├── 13-DEVOPS/                         # 新增：运维与部署（从原 07 独立出来）
│   ├── containerization/
│   │   ├── docker.md
│   │   │   ├── dockerfile-best-practice.md
│   │   │   ├── multi-stage-builds.md
│   │   │   └── docker-compose.md
│   │   └── kubernetes/                # 可选
│   │       ├── concepts.md
│   │       └── deployment.yaml
│   │
│   ├── ci-cd/
│   │   ├── github-actions.md
│   │   ├── gitlab-ci.md
│   │   └── cases/
│   │       └── automated-test-deploy.md
│   │
│   ├── cloud-services/
│   │   ├── nginx.md                   # 反向代理配置
│   │   ├── cdn.md                     # CDN 配置
│   │   └── serverless/                # 云函数
│   │       ├── cloudflare-workers.md
│   │       └── aliyun-fc.md
│   │
│   └── monitoring/
│       ├── log-system.md              # ELK/日志服务
│       └── alerting.md                # 告警配置
│
├── 14-ARCHITECTURE/                   # 新增：架构设计（从原 07 细化）
│   ├── system-design/
│   │   ├── high-availability.md       # 高可用
│   │   ├── high-concurrency.md        # 高并发
│   │   ├── scalability.md             # 可扩展性
│   │   └── cases/
│   │       └── spike-solution.md      # 秒杀系统设计
│   │
│   ├── microservices/
│   │   ├── service-mesh.md
│   │   ├── api-gateway.md
│   │   └── service-discovery.md
│   │
│   └── database-design/
│       ├── sharding.md                # 分库分表
│       ├── indexing-strategy.md
│       └── cache-strategy.md
│
└── 15-CODE-QUALITY/                   # 新增：代码质量（从原 07 细化）
    ├── testing/
    │   ├── unit-test.md               # 单元测试（深化）
    │   ├── integration-test.md
    │   ├── e2e-test.md
    │   └── test-pyramid.md
    │
    ├── code-review/
    │   ├── checklist.md
    │   └── cases/
    │       └── security-code-review.md
    │
    └── documentation/
        ├── api-documentation.md       # Swagger/OpenAPI
        ├── architecture-documentation.md  # C4 模型
        └── technical-wiki.md
```

---

## 使用指南与维护策略

### 1. 文档模板规范
每个技术文档建议遵循统一格式：

```markdown
# 标题

## 概述
- 这是什么？解决什么问题？

## 核心原理
- 图文结合，建议使用 Mermaid 流程图
- 关键代码片段

## 实践应用
- 如何在项目中使用

## 常见问题
- 踩坑记录与解决方案

## 延伸思考
- 与其他方案的对比
- 适用场景与局限性

## 参考资料
- 官方文档、优质文章、视频链接
```

### 2. 优先级建议
根据面试频率和重要性，建议优先填充：
- **P0（必填）**：`02-LANGUAGE`、`03-VUE`、`04-REACT`、`05-COMPONENT-LIB`、`10-PROJECTS`
- **P1（核心）**：`01-FOUNDATION`、`07-ENGINEERING`、`08-PERFORMANCE`
- **P2（亮点）**：`06-CROSS-PLATFORM`、`09-AI-EMPOWER`
- **P3（储备）**：`11-SOFT-SKILLS`

### 3. 维护节奏
- **每周**：记录 1-2 个解决过的技术难题到对应 `cases/` 目录
- **每月**：复盘一个项目到 `10-PROJECTS`
- **面试前**：重点回顾 `00-INDEX/interview-prep/` 和 `10-PROJECTS/`

### 4. 工具推荐
- **本地编辑**：VS Code + Markdown 插件
- **知识库管理**：Obsidian（双向链接、图谱视图）
- **图表工具**：Mermaid（内嵌在 Markdown 中）
- **云端备份**：GitHub 私有仓库

---

这个目录结构的优势在于：
1. **面试导向**：`00-INDEX/interview-prep/` 和 `10-PROJECTS/` 让你能快速输出
2. **深度与广度平衡**：既有框架原理，又有工程化、跨端、AI 等拓展方向
3. **案例驱动**：每个模块都有 `cases/` 目录，积累实战素材
4. **可演进**：新增技术领域时，在对应层级扩展即可


## 知识点补充

- BFF 设计：如何设计一个聚合层？如何处理多端适配？
- 数据库设计：表结构设计、索引优化、慢查询分析
- 并发处理：Node.js 如何应对高并发？Go 协程优势？
- 安全防护：常见的 Web 攻击与防御
- 系统设计：设计一个短链接服务、设计一个评论系统