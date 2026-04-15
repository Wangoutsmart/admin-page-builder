# Step 6：写入文件

代码生成完毕后，执行以下流程。

## 展示写入计划

向用户展示将要创建的文件树：

> 即将在以下路径创建文件：
>
> ```
> src/pages/<module-name>/
> ├── apis/index.ts
> ├── index.tsx
> └── components/
>     ├── search/index.tsx
>     └── <module>-form/index.tsx
> ```
>
> 目标目录：`<target-directory>`（来自 spec，默认 `src/pages/<module-name>/`）
>
> 确认写入？

## 等待确认

**在用户明确回复确认前，不写入任何文件。**

## 写入文件

收到用户确认后，依次创建以下文件（如目录不存在则先创建目录）：

1. `<target>/apis/index.ts`
2. `<target>/index.tsx`
3. `<target>/components/search/index.tsx`
4. `<target>/components/<module>-form/index.tsx`（Modal 模式）
   或 `<target>/create/index.tsx`、`<target>/edit/index.tsx`（独立页面模式）

## 写入完毕后：自动触发合规审查

文件写入完毕后，**立即**使用 `Agent` tool 派发 `antdboy:compliance-reviewer` agent，对刚才写入的文件进行合规审查。

派发时，prompt 中列出本次写入的所有文件路径，明确说明这些是新生成的文件：

```
请对以下新生成的文件进行 antdboy 合规审查（未列出的文件无需检查）：

- <target>/apis/index.ts
- <target>/index.tsx
- <target>/components/search/index.tsx
- <target>/components/<module>-form/index.tsx
（根据实际写入文件列出）
```

审查结果返回后，直接展示给用户。

## 写入完毕后的提示

> 文件已创建完毕，合规审查已完成（见上方报告）。
>
> **下一步建议：**
> 1. 检查 `apis/index.ts` 中的接口路径是否与后端一致
> 2. 在 `src/routes/` 中注册新路由
> 3. 如有权限点，在权限管理系统中配置对应标识符
>
> spec 文档保留在 `docs/page-builder-specs/<module-name>-spec.md`，可作为参考。
