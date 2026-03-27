## 全局基础规范

- 非必要不写样式文件，不新建less、css、scss文件
- 禁止 `any`，用 `unknown` + 类型守卫替代
- 禁止覆写 antd CSS 类名
- 请求接口使用 axios + ahooks `useRequest或者useAntdTable`
