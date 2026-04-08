---
name: api-organization
description: React + Ant Design + ahooks 中后台项目的模块接口层组织规范。用于开发或修改中后台业务模块时，统一约束接口请求函数的文件位置、导出方式、函数写法及对应 TS 类型的放置规则。默认要求每个业务模块在根目录创建 `apis/index.ts`，导出一个 `apis` 对象，对象方法为基于 `utils/axios.ts` 实例的请求函数，入参和响应类型在同一文件定义，业务组件通过 `apis.xxx` 在 `useRequest` / `useAntdTable` 中调用。不用于 hook 选型、loading 状态管理、权限控制或表单规范。
user-invocable: false
---

# 模块接口层组织规范

## 说明

本规范约束业务模块中接口请求函数的组织方式，让请求函数、入参类型、响应类型统一在一个固定位置维护，方便查找、替换和类型追溯。
业务组件只需导入 `apis` 对象，不关心请求实现细节。

## 目录结构

每个业务模块（`src/pages/<ModuleName>/`）根目录下统一创建 `apis/` 文件夹，只维护一个 `index.ts`：

```
src/pages/user-management/
├── apis/
│   └── index.ts      # 请求函数定义 + 入参/响应类型
├── index.tsx         # 页面入口
└── components/
```

## `apis/index.ts` 写法规范

- 导出一个具名 `apis` 对象，**不要导出散装函数，不要使用 class**。
- 对象的每个方法对应一个接口，直接 `return` axios 实例的调用结果。
- **所有 axios 调用必须使用 `utils/axios.ts` 暴露的实例**，禁止直接 `import axios from 'axios'`。
- 接口入参类型（`XxxQueryParams`、`XxxCreateParams`、`XxxUpdateParams`）和响应类型（`XxxItem`、`XxxDetail`、`XxxResponse`）在同一文件内定义，放在 `apis` 对象上方。
- **字段名以 Swagger/OpenAPI 实际定义为准**，不自行推断或根据 UI 语义命名。

### 推荐写法

```typescript
import { axios } from '@/utils'

// 入参 / 响应类型就近定义
export interface UserQueryParams {
  keyword?: string
  status?: number
  pageNum: number
  pageSize: number
}

export interface UserItem {
  id: number
  name: string
  status: number
  createdAt: string
}

export interface UserCreateParams {
  name: string
  roleId: number
}

// 统一导出 apis 对象
export const apis = {
  /** 获取用户列表 */
  getUserList: (params: UserQueryParams) =>
    axios.get<{ list: UserItem[]; total: number }>('/api/user/list', { params }),

  /** 新增用户 */
  createUser: (data: UserCreateParams) =>
    axios.post<void>('/api/user/create', data),

  /** 删除用户 */
  deleteUser: (id: number) =>
    axios.delete<void>(`/api/user/${id}`),
}
```

### 不推荐写法

```typescript
// ❌ 散装导出函数，难以统一管理
export function getUserList(params: any) { ... }
export function createUser(data: any) { ... }

// ❌ 直接使用原生 axios，绕过统一实例
import axios from 'axios'
export const apis = {
  getUserList: (params) => axios.get('/api/user/list', { params }),
}

// ❌ 类型定义和请求函数拆到不同文件
// types.ts 里定义接口类型，apis/index.ts 只写函数体
```

## 业务组件中的使用方式

统一通过 `import { apis } from './apis'` 导入，在 `useRequest` / `useAntdTable` 中以 `apis.xxx` 形式传入，不要在组件 body 里散写裸请求调用。

```typescript
import { useRequest } from 'ahooks'
import { apis } from './apis'

// useRequest 示例（手动触发）
const { run: handleDelete, loading: deleteLoading } = useRequest(apis.deleteUser, {
  manual: true,
  onSuccess: () => {
    message.success('删除成功')
    refresh()
  },
})

// useAntdTable 示例
const { tableProps, search } = useAntdTable(
  ({ current, pageSize }, formData) =>
    apis.getUserList({ pageNum: current, pageSize, ...formData }),
  { form, defaultPageSize: 20 },
)
```

## 类型放置边界

- `apis/index.ts` **只放接口契约类型**（请求入参、响应数据结构）。
- 表单类型（`XxxFormValues`）、搜索表单类型（`XxxSearchFormValues`）、组件 Props 不属于接口契约，不放在此文件。
- 当模块内多个页面或子组件需要共享同一批业务类型时，再抽到模块级 `types.ts`，不要提前抽取。
