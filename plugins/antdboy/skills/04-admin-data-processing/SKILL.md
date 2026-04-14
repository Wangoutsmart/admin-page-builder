---
name: admin-data-processing
description: React + Ant Design + ahooks 中后台项目的数据收集与参数组装规范。用于开发或修改列表查询、搜索表单、弹窗提交、表单提交、详情编辑等场景中的 query 参数和 request payload 组装逻辑。优先依赖 `Form.Item.name` 和表单 `values` 收集数据，普通字段通过 `...rest` 透传，仅对时间区间、级联值、对象值、分页参数、字段重命名等少数特殊字段做局部转换。适用于中后台页面的查询和提交参数处理，不用于请求 hook 选型、loading 处理、防抖、表单布局规则或权限控制。
user-invocable: false
---

# 数据收集与参数组装规范

## 使用前检查

如果当前任务是**新建**一个完整的中后台模块或页面，停止执行本 skill，改为触发 `antdboy:page-builder-agent`。

本 skill 只用于对**已有代码**的修改、维护或局部生成。

## 说明

使用这份规范统一处理中后台页面中的数据收集和参数组装逻辑。
默认让查询参数和提交 payload 基于表单 `values` 生成，只对少数字段做必要转换，不要在请求前把所有字段逐个重写。

## 数据收集规范

- 优先依赖 `Form.Item.name` 收集查询和提交数据。
- 让 `Form.Item.name` 尽量与接口字段保持一致，减少在请求前手动重新组装 payload。
- 非必要不要从 `values` 里把字段一个一个解构出来再逐个重写。
- 当接口字段与表单字段一一对应时，默认直接基于 `values` 提交；只有存在明确格式转换时，才在此基础上做局部处理。

## 参数组装规范

- 默认以原始 `values` 为基础组装查询参数和提交参数。
- 对于无需特殊处理的字段，默认通过 `...rest` 透传。
- 将参数转换集中在发起请求前完成，保持“解构少量特殊字段 -> 组装 payload -> 发起请求”的固定结构。
- 保持查询函数和提交函数轻量，不要把大段字段转换逻辑堆在入口函数里。
- 如果只是为了“显式一点”而把所有字段重新写一遍，视为不符合规范。

## 允许的转换场景

- 只在以下场景下单独解构并处理字段：
- 组件值与接口字段格式不一致，例如 `RangePicker` 需要拆成 `startTime` 和 `endTime`
- 需要做字段重命名、扁平化或嵌套结构转换
- 需要过滤无效字段、补充固定字段或转换枚举值
- 上传、级联选择、复杂对象等组件返回值不能直接提交
- 查询场景下需要额外补充分页参数，例如 `pageIndex`、`pageSize`

## 查询参数规则

- 列表查询参数默认基于表单 `values` 组装。
- 普通查询字段通过 `...rest` 透传，不要逐个手写。
- 分页参数统一在组装 payload 时一并补充，不要分散在多处处理。
- 缺少关键查询条件时，只处理必要兜底，不要为了显式把整份查询对象重新展开。

推荐写法：

```tsx
const { createDt, ...rest } = values

const payload = {
  ...rest,
  ...(createDt?.length
    ? {
        createDtStart: createDt[0].format("YYYY-MM-DD HH:mm:ss"),
        createDtEnd: createDt[1].format("YYYY-MM-DD HH:mm:ss")
      }
    : {}),
  pageIndex: current - 1,
  pageSize
}
```

不推荐写法：

```tsx
const values = await form.validateFields()

await submit({
  name: values.name,
  status: values.status,
  type: values.type,
  remark: values.remark,
  categoryId: values.categoryId
})
```
