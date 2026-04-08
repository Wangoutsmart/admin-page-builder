# Step 5：生成代码

用户确认 spec 文档后，按以下顺序生成各文件代码。每个文件生成时，严格遵循对应 skill 的规范。

## 生成顺序

### 1. apis/index.ts

遵循 `antdboy:01-api-organization` 规范：
- 导出具名 `apis` 对象，不导出散装函数
- 入参类型（`XxxQueryParams`、`XxxCreateParams`、`XxxUpdateParams`）和响应类型（`XxxItem`、`XxxDetail`）定义在同一文件
- 所有 axios 调用使用 `utils/axios.ts` 暴露的实例
- 字段名以 Swagger/接口真实定义为准，不自行推断

同时遵循 `antdboy:02-data-fetching` 中的 swagger 请求规范：通过接口真实定义确认字段名。

### 2. index.tsx（列表页容器）

遵循 `antdboy:00-admin-page-organization` 规范：
- `Search` 组件的 `form` 实例由此处创建并下传
- 使用 `useAntdTable` 承载列表查询，配置 `defaultPageSize: 20`
- 弹窗通过 `ref.show(...)` 触发，不在此维护弹窗的 boolean state

遵循 `antdboy:02-data-fetching` 规范：
- 删除、启停等操作使用 `useRequest`，`manual: true`
- loading 直接绑定在触发按钮上

遵循 `antdboy:03-authorized` 规范（如 spec 中有权限点）：
- 操作按钮用权限组件包裹
- 权限标识符使用 spec 中确认的值

### 3. components/search/index.tsx

遵循 `antdboy:00-admin-page-organization` 的 Search 组件规范：
- 只接收 `form` 和 `search` 两个 props
- 不在内部创建 `useAntdTable`
- 不发列表请求

### 4. components/<module>-form/index.tsx（Modal 模式）或 create/、edit/ 页面（独立页面模式）

遵循 `antdboy:05-admin-form-standards` 规范：
- 使用 `Form`、`Form.Item`、`forwardRef` + `useImperativeHandle`（Modal 模式）
- 提交使用 `useRequest`，loading 绑定到确认按钮
- 表单验证规则按接口字段类型设置

遵循 `antdboy:00-admin-page-organization` 的弹窗控制规范（Modal 模式）：
- 暴露 `show(options)` 方法
- 内部自行维护 `open` 状态
- 成功/取消通过 `onSuccess`/`onCancel` 回传

### 5. 类型定义

遵循 `antdboy:07-ts-type-organization` 规范：
- 接口契约类型在 `apis/index.ts` 中定义
- 表单类型（`XxxFormValues`）在表单组件文件中定义
- 组件 Props 类型在各自文件中定义
- 禁止使用 `any`

### 6. 注释

遵循 `antdboy:06-code-comment-standards` 规范：
- 函数和接口加 JSDoc 注释（中文）
- 非显而易见的逻辑加行内注释
- 不加废话注释（如"这是一个函数"）

### 7. 命名检查

遵循 `antdboy:08-naming-conventions` 规范：
- 文件和目录名一律 kebab-case
- 组件名 PascalCase
- 变量和函数名 camelCase
- 类型名 PascalCase

## 生成完毕后

所有文件代码生成完毕后，**不要自动写入磁盘**，移交 `steps/04-write.md` 处理。
