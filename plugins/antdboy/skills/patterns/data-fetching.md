# 数据请求约定

- 所有请求统一通过 `utils/axios.ts` 暴露的 `axios` 实例发起，禁止在组件内直接使用原生 `axios`。
- 数据请求只允许使用 ahooks 的 `useRequest` 或 `useAntdTable`。
- 请求层已统一处理错误时，禁止在请求函数里再手动 `try/catch` 并调用 `message.error`，避免重复提示。
- 非必要不要新建 `service.ts`、`apis.ts`；请求逻辑优先就近放在页面、弹窗或 hook 中。
- 枚举类请求、下拉选项请求、跨页面复用的通用请求，统一封装为 `src/hooks` 下的 `useXxx` hook。

## loading 规范

- 列表查询、新增、编辑、删除、启停、提交等用户主动触发的请求，必须正确使用 loading。
- loading 直接使用 `useRequest` 和 `useAntdTable` 返回的状态，不要再额外用 `useState` 维护一份同义状态。

## useRequest 规范

- 除表格列表请求外，其他异步请求统一使用 `useRequest`。
- 默认自动触发的请求，不配置 `manual: true`；只有需要手动触发时才配置。
- 只在 `Modal` 内使用的数据，默认不要在组件初始化时自动请求，应在弹窗打开后再触发。
- 查询、新增、编辑等高频操作，优先配置：

```typescript
debounceWait: 300,
debounceLeading: true,
```

- 如果按钮或Modal自身已有明确 loading 且能防止重复提交，可以不额外配置防抖debounceWait。

## useAntdTable 规范

- useAntdTable 只用于“查询条件 + 分页列表 + Table 展示”场景。
- 默认分页大小统一为 20，配置 defaultPageSize: 20。
- - 请求结果统一返回 { list, total }。
    外部关键参数驱动的列表刷新，统一使用 refreshDeps。

## 补充约束

- 不要在同一页面里混用多套请求方案。
- 不要把同一个接口同时写在页面和通用 hook 里。
- 不要重复处理请求层已经统一兜底的错误、loading 和参数逻辑。
