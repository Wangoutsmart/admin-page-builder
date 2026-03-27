# 页面组织模式

根据业务复杂度选择两种组织方式：

**选择依据**：表单字段 ≤ 10 个、无复杂联动 → 用 Modal；字段多、有分步/Tab 布局、需要独立 URL → 用独立页面。

---

## 场景 A：Modal 形式（表单简单、字段少）

新增/编辑/查看在弹窗内完成，列表页结构：

```
[module]/
├── index.tsx              # 页面容器：持有 useAntdTable、所有 ref、Authorized 权限控制
└── components/
    ├── search/index.tsx   # 搜索栏（纯展示，无请求，接收 form + search props）
    ├── [form]/index.tsx   # 新增/编辑/查看 Modal
    ├── [detail]/index.tsx # 详情/子项配置 Modal（如需要）
    └── logs/index.tsx     # 操作日志 Modal（如需要）
```

### Modal 控制：`forwardRef` + `useImperativeHandle`

弹窗通过 `ref.show()` 触发，**不通过父组件 state 控制开关**，避免每个弹窗维护独立 boolean。

```tsx
// 定义 Ref 类型
export interface TagTypeRef {
  show(options?: { values: TagTypeValues; disabled: boolean }): void
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

### `refreshDeps` 驱动弹窗内数据联动

弹窗内部请求以关键 id 作为 `refreshDeps`，打开不同记录时自动触发重新请求。

```tsx
const { data, loading, refresh } = useRequest(
  async () => {
    /* 用 id 发请求 */
  },
  { refreshDeps: [tagTypeId] }
)
```

---

## 场景 B：独立页面形式（表单复杂、字段多、需要独立路由）

新增/编辑/详情各为独立页面，通过路由跳转：

```
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
    └── [shared]/index.tsx # 列表/新增/编辑页共用的子组件
```

路由配置示例（`routes/[module].tsx`）：

```tsx
{ path: '/module',            element: <List /> },
{ path: '/module/create',     element: <Create /> },
{ path: '/module/:id/edit',   element: <Edit /> },
{ path: '/module/:id',        element: <Detail /> },
```

---

## 通用约定

### Search 组件：form 实例由父组件创建并下传

```tsx
// 父组件
const [search] = Form.useForm()
const table = useAntdTable(fetchFn, { form: search })
<Search form={search} search={table.search} />

// Search 组件 props 类型
interface Props {
  search: ReturnType<typeof useAntdTable>['search']
  form: FormInstance
}
```
