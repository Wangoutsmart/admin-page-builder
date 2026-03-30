# 页面组织模式

页面只允许两种组织方式：`Modal` 或 `独立页面`。先判断复杂度，再确定结构；不要在同一个模块里同时维护两套新增/编辑实现。

## 选择规则

满足以下条件时，优先使用 `Modal` 形式：

- 表单字段数量 `<= 10`
- 无分步、Tab、折叠区块等复杂布局
- 无复杂联动、子表单、跨区块编辑
- 不需要独立路由、独立 URL、页面级面包屑或可分享链接

只要命中以下任一条件，就使用 `独立页面` 形式：

- 表单字段数量 `> 10`
- 存在分步、Tab、分区布局
- 存在复杂联动、子配置、长链路交互
- 需要独立路由、独立 URL、页面级权限或前进/返回可恢复状态

---

## 场景 A：Modal 形式

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

### 弹窗控制：forwardRef + useImperativeHandle

- 弹窗统一通过 ref.show(...) 触发，父组件不要为每个弹窗维护独立的 boolean state。
- 父组件负责触发弹窗，并通过 show 方法传入 mode、record、initialValues 等上下文数据；弹窗组件内部自行维护开关状态。
- 弹窗的成功、取消、关闭等结果通过 onSuccess、onCancel、onClose 回传给父组件；父组件只负责刷新和联动，不直接控制弹窗开关。
- 除非项目已有明确约束，否则不要回退到“父组件 state + 子组件 open 属性”这一套写法。
- useImperativeHandle 暴露的方法只做轻量调度，例如“打开弹窗 + 同步上下文 + 初始化表单”。
- 如果 useImperativeHandle 暴露的方法实现超过五行，先把实际逻辑抽离成独立函数，再在暴露方法里调用；不要把完整业务流程堆进 show、hide、open 之类的方法里。

```tsx
// 定义 Ref 类型
export interface TagTypeRef {
  show(options?: { values?: TagTypeValues; disabled?: boolean }): void
}

// 子组件暴露方法
useImperativeHandle(ref, () => ({
  show(options) {
    setOpen(true)
    setDisabled(!!options?.disabled)
    options?.values ? form.setFieldsValue(options.values) : form.resetFields()
  }
}))

export default forwardRef(TagType)

// 父组件调用
const tagTypeRef = useRef<TagTypeRef>(null)
tagTypeRef.current?.show({ values: record, disabled: false })
```

### 弹窗内数据联动：refreshDeps

- 弹窗内部如果存在依赖记录 id 的请求，使用关键 id 作为 refreshDeps。
- 打开不同记录时，应通过依赖变更自动触发重新请求，不要把同一套刷新逻辑分散在多个 useEffect 里。
- refreshDeps 只绑定真正影响请求结果的关键参数，例如 id、typeId、detailId。

```tsx
const { data, loading, refresh } = useRequest(
  async () => {
    /* 用 id 发请求 */
  },
  { refreshDeps: [tagTypeId] }
)
```

## 场景 B：独立页面形式

当新增、编辑、详情本身已经是完整业务页面时，使用独立页面形式，通过路由切换。

目录结构约定：

```text
[module]/
├── index.tsx # 列表页容器
├── create/
│ └── index.tsx # 新增页
├── edit/
│ └── index.tsx # 编辑页（通过路由参数 id 加载数据）
├── detail/
│ └── index.tsx # 详情页
└── components/
├── search/index.tsx # 搜索栏
└── [shared]/index.tsx # 列表/新增/编辑/详情共用子组件
```

路由约定示例：

```tsx
{ path: '/module',          element: <List /> },
{ path: '/module/create',   element: <Create /> },
{ path: '/module/:id/edit', element: <Edit /> },
{ path: '/module/:id',      element: <Detail /> },
```

## 通用约定

### Search 组件

- Search 组件只负责渲染查询项和触发查询/重置，不负责发列表请求。
- form 实例由父组件创建并下传，Search 组件内部不要重新 Form.useForm()。
- Search 组件只接收父组件传入的 form 和 search，不要在内部直接创建 useAntdTable。

```tsx
// 父组件
const [search] = Form.useForm()
const table = useAntdTable(fetchFn, { form: search })

<Search form={search} search={table.search} />

// Search 组件 props 类型
interface Props {
  form: FormInstance
  search: ReturnType<typeof useAntdTable>['search']
}
```
