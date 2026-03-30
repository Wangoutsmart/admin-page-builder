# 数据处理规范

## 表单提交数据规范

- 提交表单时，优先依赖 `Form.Item` 的 `name` 收集提交数据。
- `Form.Item.name` 应尽量与接口字段保持一致，减少在提交阶段手动重新组装 payload。
- 非必要不要在提交时从 `values` 里把字段一个一个解构出来再重新拼装提交对象。

- 如果大部分字段都可以直接复用，提交时应以原始 `values` 为基础，仅对少数字段做必要转换，其他字段默认透传。
- 只有在以下场景下，才允许单独解构并处理字段：
  - 组件值与接口字段格式不一致，例如 `RangePicker` 需要拆成 `startTime` 和 `endTime`
  - 需要做字段重命名、扁平化或嵌套结构转换
  - 需要过滤无效字段、补充固定字段或转换枚举值
  - 上传、级联选择、复杂对象等组件返回值不能直接提交

- 如果只是为了“显式一点”而把每个字段重新写一遍，视为不符合规范。
- 如果接口字段命名明确，优先在 `Form.Item.name` 层面就对齐，不要把字段映射逻辑全部堆到提交函数里。
- 提交函数应保持轻量：校验表单、提取少量特殊字段、组装最终 payload、发起请求；不要把大段字段转换逻辑堆在提交入口。

推荐写法：

```tsx
const values = await form.validateFields()

const { timeRange, ...rest } = values

const payload = {
  ...rest,
  startTime: timeRange?.[0],
  endTime: timeRange?.[1]
}

await submit(payload)
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

```md
- 当接口字段与表单字段一一对应时，默认使用 `await submit(values)`；只有存在明确格式转换时才在此基础上做局部处理。
```
