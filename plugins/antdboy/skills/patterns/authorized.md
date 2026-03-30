### 权限控制

- 如果需求文档、设计稿或说明文档没有明确提到权限控制，就忽略权限相关实现，不要自行补权限判断，也不要编造权限名称。
- 如果文档明确说明某个按钮、操作区块或局部页面需要权限控制，并且给出了权限名称，才按权限规范实现。
- 如果文档明确说“需要权限”，但没有给出明确的权限名称或权限编码，停止补权限代码并向用户确认；不要自行猜测。
- 权限控制优先包裹最小可控单元，例如按钮、行内操作、卡片区块或局部功能区，不要无故把整页都包进权限组件。
- 需要权限的按钮或局部页面，使用 `<Authorized permission="...">` 包裹。
- 除了 `permission` 之外，如果当前需求里已经能确定显示条件、状态描述和最终调用的接口地址，就一并在 `Authorized` 上补充 `visible`、`status`、`apis`，不要把这些信息散落在外层。
- `visible` 表示业务上的显示条件，`status` 表示当前权限操作对应的业务状态描述，`apis` 表示当前操作最终命中的接口方法和路径。

例如：

```tsx
<Authorized permission="权限名字" visible={status === 1} status={["状态是待发货"]} apis={["POST /opt-test/shipping"]}>
  <Button>发货</Button>
</Authorized>
```

- 生成权限代码前，先检查项目的 src/components 下是否已经存在 Authorized 或其他授权包装组件。
- 如果项目里已经有现成实现，优先复用项目组件，不要再新建第二套权限包装。
- 如果 src/components 下没有授权相关组件，再参考[references/authorized-component.md](./references/authorized-component.md)生成默认的 Authorized 组件。
