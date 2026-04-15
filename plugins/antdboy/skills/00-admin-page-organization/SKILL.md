---
name: admin-page-organization
description: React + Ant Design + ahooks 中后台项目的页面组织规范。用于开发、修改或重构中后台 CRUD 模块时，判断新增、编辑、详情应采用 `Modal` 还是 `独立页面`，并统一约束模块目录结构、弹窗触发方式、独立路由组织和 Search 组件边界。适用于列表页、弹窗页、详情页、编辑页、搜索区等中后台页面组织场景。默认要求同一模块只维护一套新增/编辑实现，不在同一模块中混用 `Modal` 和 `独立页面` 两种主组织方式。
user-invocable: false
---

# 页面组织规范

## 使用前检查

如果当前任务是**新建**一个完整的中后台模块或页面，停止执行本 skill，改为触发 `antdboy:page-scaffold`。

本 skill 只用于对**已有代码**的修改、维护或局部生成。

## 说明

使用这份规范统一处理中后台 CRUD 模块的页面组织方式。
先判断业务复杂度，再决定采用 `Modal` 还是 `独立页面`。同一模块默认只允许一种主组织方式，不要同时维护两套新增、编辑实现。

## 全局目录规范

整个项目默认遵循下面这套目录结构：

```text
src/
├── components/
│   └── [feature]/        # 业务组件（不含数据请求）
├── hooks/                # 自定义 Hooks
├── layouts/              # BasicLayout、MicroLayout 等
├── pages/
│   └── [module]/
│       ├── apis/         # module 下所有涉及的 api 接口和接口出入参类型定义
│       ├── components/   # 页面私有子组件
│       └── index.tsx     # 页面容器（数据编排）
├── routes/
│   ├── index.tsx         # 路由树（所有页面懒加载）
│   └── [module].tsx      # 页面路由
├── styles/
│   └── globals.css
├── types/                # 全局 TS 类型
└── utils/                # 全局工具函数
```

- `src/components` 只放可跨页面复用的业务组件，不承载页面级数据请求。
- `src/pages/[module]/apis` 统一放当前模块涉及的接口、请求入参和响应类型定义。
- `src/pages/[module]/components` 只放当前模块私有子组件。
- `src/pages/[module]/index.tsx` 默认作为当前模块的页面容器，负责数据编排和主交互组织。

## 选择规则

- 如果需求、设计稿、截图或现有上下文已经明确说明使用 `Modal` 或 `独立页面`，优先遵循明确约束。
- 如果没有明确说明，则按下面的复杂度规则自行判断。

优先使用 `Modal` 的条件：

- 表单字段数量 `<= 10`
- 无分步、Tab、折叠区块等复杂布局
- 无复杂联动、子表单、跨区块编辑
- 不需要独立路由、独立 URL、页面级面包屑或可分享链接

只要命中以下任一条件，就使用 `独立页面`：

- 表单字段数量 `> 10`
- 存在分步、Tab、分区布局
- 存在复杂联动、子配置、长链路交互
- 需要独立路由、独立 URL、页面级权限或前进、返回可恢复状态

## Modal 组织方式

当新增、编辑、详情都可以在当前列表页内完成时，使用 `Modal` 形式。

目录结构约定：

```text
[module]/
├── index.tsx
└── components/
    ├── search/index.tsx   # 搜索栏：纯展示和交互，不发列表请求，只接收 form + search
    ├── [form]/index.tsx   # 新增/编辑/详情弹窗
    ├── [detail]/index.tsx # 详情/子项配置弹窗（如需要）
    └── logs/index.tsx     # 操作日志弹窗（如需要）
```

### 弹窗控制规范

- 弹窗统一通过 `ref.show(...)` 触发，父组件不要为每个弹窗维护独立的 boolean state。
- 父组件负责触发弹窗，并通过 `show` 方法传入 `mode`、`record`、`initialValues` 等上下文数据；弹窗组件内部自行维护开关状态。
- 弹窗的成功、取消、关闭等结果通过 `onSuccess`、`onCancel`、`onClose` 回传给父组件；父组件只负责刷新和联动，不直接控制弹窗开关。
- 除非项目已有明确约束，否则不要回退到“父组件 state + 子组件 `open` 属性”这一套写法。
- `useImperativeHandle` 暴露的方法只做轻量调度，例如“打开弹窗 + 同步上下文 + 初始化表单”。
- 如果 `useImperativeHandle` 暴露的方法实现超过五行，先把实际逻辑抽离成独立函数，再在暴露方法里调用；不要把完整业务流程堆进 `show`、`hide`、`open` 之类的方法里。

示例：

```tsx
export interface TagTypeRef {
  show(options?: { values?: TagTypeValues; disabled?: boolean }): void
}

useImperativeHandle(ref, () => ({
  show(options) {
    setOpen(true)
    setDisabled(!!options?.disabled)
    options?.values ? form.setFieldsValue(options.values) : form.resetFields()
  }
}))

export default forwardRef(TagType)

const tagTypeRef = useRef<TagTypeRef>(null)
tagTypeRef.current?.show({ values: record, disabled: false })
```

### 弹窗内数据联动规范

- 弹窗内部如果存在依赖记录 id 的请求，使用关键 id 作为 `refreshDeps`。
- 打开不同记录时，应通过依赖变更自动触发重新请求，不要把同一套刷新逻辑分散在多个 `useEffect` 里。
- `refreshDeps` 只绑定真正影响请求结果的关键参数，例如 `id`、`typeId`、`detailId`。

示例：

```tsx
const { data, loading, refresh } = useRequest(
  async () => {
    /* 用 id 发请求 */
  },
  { refreshDeps: [tagTypeId] }
)
```

## 独立页面组织方式

当新增、编辑、详情本身已经是完整业务页面时，使用独立页面形式，通过路由切换。

目录结构约定：

```text
[module]/
├── index.tsx              # 列表页容器
├── create/
│   └── index.tsx          # 新增页
├── edit/
│   └── index.tsx          # 编辑页（通过路由参数 id 加载数据）
├── detail/
│   └── index.tsx          # 详情页
└── components/
    ├── search/index.tsx   # 搜索栏
    └── [shared]/index.tsx # 列表/新增/编辑/详情共用子组件
```

路由约定示例：

```tsx
{ path: '/module',          element: <List /> },
{ path: '/module/create',   element: <Create /> },
{ path: '/module/:id/edit', element: <Edit /> },
{ path: '/module/:id',      element: <Detail /> },
```

## Search 组件规范

- `Search` 组件只负责渲染查询项和触发查询、重置，不负责发列表请求。
- `form` 实例由父组件创建并下传，`Search` 组件内部不要重新 `Form.useForm()`。
- `Search` 组件只接收父组件传入的 `form` 和 `search`，不要在内部直接创建 `useAntdTable`。

示例：

```tsx
const [search] = Form.useForm()
const table = useAntdTable(fetchFn, { form: search })

<Search form={search} search={table.search} />

interface Props {
  form: FormInstance
  search: ReturnType<typeof useAntdTable>['search']
}
```

## 默认约束

- 同一模块默认只保留一种主组织方式，不要同时维护 Modal 版和独立页面版新增、编辑实现。
- 如果选择了 `Modal`，新增、编辑、详情的主交互默认都在当前列表页上下文内完成。
- 如果选择了 `独立页面`，新增、编辑、详情默认都通过路由切换完成，不要只把其中一个单独做成页面。
- `Search` 组件默认作为页面子组件存在，不承载请求逻辑和页面编排逻辑。
- 页面组织规则只解决模块结构和交互承载方式，不替代表单规范、请求规范、权限规范。
