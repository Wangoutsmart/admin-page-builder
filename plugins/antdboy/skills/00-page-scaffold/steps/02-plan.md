# Step 3-4：判断页面类型与生成 spec 文档

## Step 3：判断页面类型

依据 `antdboy:00-admin-page-organization` 的选择规则，结合原型图和接口信息判断：

**使用 Modal 模式的条件（满足全部）：**
- 表单字段数量 ≤ 10
- 无分步、Tab、折叠区块等复杂布局
- 无复杂联动、子表单、跨区块编辑
- 不需要独立路由或可分享链接

**使用独立页面模式的条件（满足任一）：**
- 表单字段数量 > 10
- 存在分步、Tab、分区布局
- 存在复杂联动、子配置、长链路交互
- 需要独立路由、独立 URL 或前进/返回可恢复状态

### 从原型图识别权限点

扫描原型图中的操作按钮区域，识别需要鉴权的操作（通常在列表操作列或页面顶部工具栏），按以下规则推断权限标识符：

| 中文操作 | 推断标识符格式 |
|----------|---------------|
| 新增 / 创建 | `<module>:create` |
| 编辑 / 修改 | `<module>:update` |
| 删除 | `<module>:delete` |
| 导出 | `<module>:export` |
| 启用 / 禁用 | `<module>:status` |
| 查看详情 | `<module>:detail` |

其中 `<module>` 为模块目录名去掉连字符后的驼峰形式（如 `user-management` → `userManagement`）。

## Step 4：写入 spec 文档

将生成计划写入 `docs/page-builder-specs/<module-name>-spec.md`。

### spec 文档模板

```markdown
# <模块中文名> 页面生成计划

## 基本信息
- 模块名：<module-name>
- 页面类型：Modal / 独立页面
- 判断依据：字段数 X 个，无/有复杂联动

## 目录结构

（Modal 模式）
src/pages/<module-name>/
├── apis/index.ts
├── index.tsx
└── components/
    ├── search/index.tsx
    └── <module>-form/index.tsx

（独立页面模式）
src/pages/<module-name>/
├── apis/index.ts
├── index.tsx
├── create/index.tsx
├── edit/index.tsx
├── detail/index.tsx（如需要）
└── components/
    └── search/index.tsx

## 文件职责说明
| 文件 | 职责 |
|------|------|
| apis/index.ts | 接口函数 + 入参/响应类型 |
| index.tsx | 列表页容器，useAntdTable |
| components/search/index.tsx | 搜索栏 |
| components/<module>-form/index.tsx | 新增/编辑弹窗（Modal 模式） |

## 接口映射
| 接口路径 | 方法 | 用途 |
|----------|------|------|
| （从 Swagger 或用户描述填入） | | |

## 权限点
| 操作 | 权限标识 | 来源 |
|------|----------|------|
| （从原型图识别填入） | | 原型图推断 |

## 待确认项
- [ ] 编辑接口字段与新增是否复用同一表单
- [ ] 以下权限标识符请确认是否正确：<列出推断的权限标识符>
（如有其他不确定项，在此列出）
```

## Step 4 后续：等待用户审阅

写入 spec 文档后，提示用户：

> spec 文档已生成：`docs/page-builder-specs/<module-name>-spec.md`
>
> 请在编辑器中查看，确认以下内容：
> 1. 目录结构和文件职责是否符合预期
> 2. 接口映射是否正确
> 3. 权限点标识符是否正确
>
> 确认无误后回复"确认"，或提出需要修改的地方。

**在收到用户明确确认前，不进入 Step 5（代码生成）。**
