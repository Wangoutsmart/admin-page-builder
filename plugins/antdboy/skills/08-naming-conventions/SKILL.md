---
name: naming-conventions
description: React + Ant Design + ahooks 中后台项目的统一命名规范。用于开发、修改或重构中后台模块时，统一约束文件与目录命名、React 组件命名、函数与方法命名、变量与常量命名、自定义 Hook 命名、事件 Props 命名、枚举与业务常量命名。默认以"见名知意、作用域就近、风格一致"为核心原则。当你生成或修改任何涉及文件路径、组件名、函数名、变量名、类型名、枚举定义的代码时，都应当检查并应用本规范。
user-invocable: false
---

# 命名规范

## 使用前检查

如果当前任务是**新建**一个完整的中后台模块或页面，停止执行本 skill，改为触发 `antdboy:page-scaffold`。

本 skill 只用于对**已有代码**的修改、维护或局部生成。

## 说明

统一处理中后台前端项目中的所有命名决策。核心原则：**见名知意**（命名直接反映用途，不靠注释兜底）、**作用域就近**（命名粒度与作用范围匹配）、**风格一致**（同类事物全项目统一）。  
如果需要向用户展示对比示例，或对某个维度的具体写法有疑问，读取 `references/examples.md` 对应章节。  
类型放置位置见 `07-ts-type-organization`，接口层组织见 `01-api-organization`，目录结构选型见 `00-admin-page-organization`。

---

## 一、文件与目录命名

- 文件和目录名**一律 kebab-case**，禁止 PascalCase 或 camelCase（保证跨操作系统路径一致）
- 每个模块或子组件入口统一命名为 `index.tsx`
- 私有子组件在 `components/` 下建独立 kebab-case 目录，入口仍为 `index.tsx`

---

## 二、React 组件命名

- 文件目录 kebab-case，**组件函数名、类型名、枚举名等代码标识符仍用 PascalCase**（路径是系统概念，导出名是代码概念）
- 组件函数名与所在目录名语义一致，禁止脱节
- `forwardRef` 组件禁止匿名函数形式，必须具名；ref 类型命名为 `XxxRef`（`interface` 定义）
- HOC 以 `with` 为前缀，返回有名函数，不返回匿名箭头函数
- Context 三件套命名：`XxxContext` / `XxxProvider` / `useXxx`

---

## 三、函数与方法命名

**事件处理函数**：组件内部统一用 `handle + 动作[+ 对象]`（`handleSubmit`、`handleStatusChange`）。`on` 前缀**只用于 Props 回调**，内部处理函数禁止使用。

**接口请求函数**：`动词 + 名词`，动词反映 HTTP 语义：

| 操作 | 动词 |
|------|------|
| 查询 | `get` / `fetch` |
| 新增 | `create` / `add` |
| 修改 | `update` / `edit` |
| 删除 | `delete` / `remove` |
| 启停 | `enable` / `disable` |
| 导入/导出 | `import` / `export` |

**格式化函数**：`format` 前缀（`formatDate`、`formatAmount`）；表格 render 提取为函数时用 `render` 前缀（`renderStatusTag`）。

**条件判断函数**：`is`（状态/类型）、`has`（是否拥有）、`can`（是否能执行）、`should`（是否应触发）。

---

## 四、变量命名

**状态变量**：`[noun, setNoun]`，禁止 `isXxx` / `xxxState` / `showXxx` 等冗余形式。

**Loading**：直接使用 `useRequest` / `useAntdTable` 返回的 `loading`，禁止额外 `useState`。多个并发 loading 时解构重命名，格式 `动作 + Loading`（`submitLoading`、`deleteLoading`）。

**Ref**：`名词 + Ref` 后缀（`userFormRef`、`tableContainerRef`）。

**常量**：
- 模块/文件级配置常量（跨调用共享）→ SCREAMING_SNAKE_CASE（`DEFAULT_PAGE_SIZE`）
- 局部 const 不变量 → camelCase，不因是 `const` 就大写

**数据形态**：`xxxList`（列表）/ `xxxDetail`（详情）/ `xxxOptions`（选项），禁止统一用 `data` 后缀掩盖用途。

---

## 五、自定义 Hook 命名

- 必须以 `use` 开头，后接**名词**，禁止动词开头（`useFetchUsers` ❌、`useUserList` ✅）
- 按职责区分：`useXxxList`（列表）、`useXxxDetail`（详情）、`useXxxForm`（表单）、`useXxxModal`（弹窗）、`useXxxOptions`（选项）、`useXxx`（通用能力）
- 只服务单个模块的 Hook 放模块内 `hooks/`，跨模块复用才移到 `src/hooks/`

---

## 六、事件 Props 命名

- Props 回调统一 `on + 动作`（`onSuccess`、`onCancel`、`onStatusChange`）
- 纯原生事件透传直接用原生名（`onChange`、`onBlur`），不重新包装
- 对原生事件做业务处理后回传的回调用自定义 `onXxx` 命名，与原生事件对象区分

---

## 七、枚举与业务常量命名

**业务枚举**：优先用**中文字符串联合类型**（`type UserStatus = '启用' | '禁用' | '待审核'`），无需额外标签映射。  
需与后端数字值对齐或运行时遍历时才用 `enum`，此时**枚举成员名使用中文**，省去独立 LabelMap：

```ts
enum OrderStatus { 待处理 = 0, 处理中 = 1, 已完成 = 2 }
const ORDER_STATUS_COLOR: Record<OrderStatus, string> = {
  [OrderStatus.待处理]: 'warning',
  [OrderStatus.处理中]: 'processing',
  [OrderStatus.已完成]: 'success',
}
```

枚举名本身 PascalCase（`OrderStatus`、`UserRole`）。  
选项数组命名 `xxxOptions`；颜色/图标映射命名 `XxxColorMap`；配置常量带语义后缀（`_SIZE`、`_TIMEOUT`）。

---

## 默认约束

**硬性（必须遵守）：**
- 文件/目录名一律 kebab-case，禁止 PascalCase / camelCase
- 组件函数名、类型名、枚举名使用 PascalCase
- `on` 专属 Props 回调，组件内部处理函数禁止使用 `on`
- 自定义 Hook 必须 `use` 开头，禁止动词开头
- 禁止 `Data`、`Info`、`Obj`、`Temp`、`Tmp` 等无业务语义命名
- 禁止数字序号后缀（`form1`、`modal2`），改用语义词区分

**推荐（优先遵守）：**
- `xxxList` / `xxxDetail` / `xxxOptions` 区分数据形态
- `format` / `is` / `has` / `can` / `should` 前缀各司其职
- `handle + 动作` 事件处理；`动作 + Loading` 多 loading 区分
- 枚举成员名使用中文（仅限纯前端枚举）
- 模块级配置常量 SCREAMING_SNAKE_CASE，局部 const 用 camelCase
