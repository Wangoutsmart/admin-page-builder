# 数据请求约定

所有请求必须经过 `utils/axios.ts`，组件内禁止直接调用原生 axios。

组件内数据获取统一用 `useRequest`，因为`utils/axios.ts`会统一处理error，所以禁止useRequest的执行函数内 try/catch 后手动 `message.error`。

非必要情况，不需要重新写一个service.ts或apis.ts里面放各种api接口

枚举类的请求，或者你认为可以全局通用的请求，请封装成hook在`src/hooks`下面管理，hooks规范如下：

```typescript
import { useRequest } from "ahooks"
import { axios } from "@/utils"

export interface ProductOption {
  id: number
  name: string
}

export function useProductOptions(params: { categoryId?: number; brandId?: number; keyword?: string }) {
  const { categoryId, brandId, keyword } = params

  const { data, loading } = useRequest(
    async () => {
      const resp = await axios.get<ProductOption[]>("/opt-overseas-admin-service/selection/search/product", params)
      return resp.data
    },
    {
      ready: !!categoryId,
      refreshDeps: [categoryId, brandId, keyword]
    }
  )

  return {
    productOptions: data,
    loading
  }
}
```

## loading规范

- 如果请求的发起涉及到用户主动触发，场景是列表查询、创建、编辑、删除等操作场景要主动使用loading，loading要使用useRequest和useAntdTable的默认loading，不需要再重复使用useState保存状态。

## useRequest 使用规范

- 除了涉及到表格数据的请求，其他请求都应该使用useRequest触发
- 如果默认发起请求的话，就是不配置manual为true，如果是需要手动主动触发的话，需要配置manual为true
- 如果useRequest获取的数据需要用在Modal里面，则默认不发起请求，当弹窗打开的时候再发起请求，避免组件加载默认过多的接口请求
- 使用useRequest的场景是查询、创建、编辑等操作时应该加一下防抖，useRequest配置debounceWait为300
- 具体的使用规范代码如下

```typescript
const { data: enumData, loading } = useRequest(
  async () => {
    const { data } = await axios.get("url", { params })
    return data
  },
  {
    manual: true // 默认自动触发的话就不设置manual
  }
)
```

## useAntdTable 使用规范

> useAntdTable 用于承载查询逻辑，查询结果统一使用 Ant Design 的 Table 组件展示

- 默认分页20条一页，配置项 defaultPageSize: 20
- 具体使用规范代码如下

```typescript
const { tableProps } = useAntdTable(
    async ({ current, pageSize }) => {
      if (!ruleId) {
        return {
          list: [],
          total: 0
        }
      }

      const resp = await axios.post<Paginated<Log[]>>(
        '/opt-work-task/backend/task-auto-assign-rule/log-list',
        {
          id: ruleId,
          pageIndex: current - 1,
          pageSize
        }
      )

      return { list: resp.data || [], total: resp.totalCount }
    },
    {
      defaultPageSize: 20,
      refreshDeps: [ruleId]
    }
  )

<Table {...tableProps} bordered columns={columns} />
```
