---
name: data-fetching
description: React + Ant Design + ahooks 中后台项目的数据请求规范。用于开发或修改中后台页面时涉及接口请求的场景，包括列表查询、分页表格、搜索表单、弹窗数据加载、详情获取、新增编辑提交、删除启停、行内操作、远端模糊搜索等。默认要求普通请求使用 `useRequest`，表单表格查询使用 `useAntdTable`，所有请求统一通过 `utils/axios.ts` 暴露的 `axios` 实例发起，并统一约束 loading、刷新联动、防抖和防频繁请求策略。不用于权限控制、页面组织模式或表单校验规则。
user-invocable: false
---

# 数据请求约定

- 所有请求统一通过 `utils/axios.ts` 暴露的 `axios` 实例发起，禁止在组件内直接使用原生 `axios`。
- 数据请求只允许使用 ahooks 的 `useRequest` 或 `useAntdTable`。
- 普通查询、详情获取、创建、编辑、删除、启停、导出、行内操作等非表格列表请求，统一使用 `useRequest`。
- 涉及“查询条件 + 分页列表 + Table 展示”的场景，统一使用 `useAntdTable`。
- 请求层已统一处理错误时，禁止在 `useRequest` / `useAntdTable` 的执行函数里手动 `try/catch` 并调用 `message.error`，避免重复提示。
- 非必要不要新建 `service.ts`、`apis.ts`；请求逻辑优先就近放在页面、弹窗或 hook 中。
- 枚举类请求、下拉选项请求、跨页面复用的通用请求，统一封装为 `src/hooks` 下的 `useXxx` hook。

## loading 规范

- 列表查询、新增、编辑、删除、启停、提交、导出等由用户主动触发的请求，必须正确使用 loading。
- loading 直接使用 `useRequest` 和 `useAntdTable` 返回的状态，不要再额外用 `useState` 维护一份同义状态。
- 按钮主动触发的请求，尽量把 loading 直接绑定在当前按钮上，不要无故扩大到整页。
- 行内操作的 loading 只作用在当前操作项，不要因为一条记录操作把整个列表全部锁住。
- 如果按钮、Modal 确认按钮或组件本身已经有明确 loading 且能阻止重复提交，不要再额外叠加一套重复的 loading 状态。

## useRequest 规范

- 除表格列表请求外，其他异步请求统一使用 `useRequest`。
- 默认自动触发的请求，不配置 `manual: true`；只有明确需要用户主动触发时才配置 `manual: true`。
- 请求依赖关键参数时，优先使用 `ready` 或 `refreshDeps` 控制时机；参数未就绪时不要先发一次空请求。
- `Modal` / `Drawer` 内部使用的数据，默认不要在组件初始化时自动请求，应在弹窗打开后再触发，避免页面初始加载时请求过多。
- 关键参数变化需要重新拉取数据时，优先使用 `refreshDeps`，不要把重复刷新逻辑分散在多个 `useEffect` 里。
- 不要在请求执行函数里吞掉异常；错误交给统一请求层处理。

## useAntdTable 规范

- `useAntdTable` 只用于“查询条件 + 分页列表 + Table 展示”场景。
- Search 表单和 Table 查询绑定时，统一通过 `useAntdTable` 承载查询逻辑，不要额外再写一套并行查询状态。
- 默认分页大小统一为 `20`，配置 `defaultPageSize: 20`。
- 请求结果统一返回 `{ list, total }` 结构。
- 外部关键参数驱动的列表刷新，统一使用 `refreshDeps`。
- 缺少必要查询条件或关键 id 时，直接返回空结果，不要发无意义请求。

## 远端模糊搜索规范

- `Select`、自动完成或其他远端模糊搜索场景，必须显式处理频繁请求问题。
- 远端模糊搜索默认补充以下限制：
  - 增加防抖，避免连续输入时频繁请求
  - 搜索词为空时不请求
  - 显示明确的 loading 状态
  - 只保留最后一次输入对应的结果，避免旧请求返回覆盖新结果
- 如果清空搜索词，应同步清空当前远端结果，不要保留旧数据误导用户。
- 本地筛选和远端搜索不要混用；只要是远端模糊搜索，就按远端模式处理，不再依赖本地 `optionFilterProp` 过滤。

推荐配置：

```typescript
debounceWait: 300,
debounceLeading: true,
```
