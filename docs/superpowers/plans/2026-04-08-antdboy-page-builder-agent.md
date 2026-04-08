# antdboy page-builder-agent Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 在 antdboy 插件中新增 `09-page-builder-agent` skill，作为线性流水线编排器，当用户意图新建中后台页面时自动触发，按顺序调用现有 00~08 规范 skill，生成完整可运行的 CRUD 模块。

**Architecture:** SKILL.md 作为主入口声明触发条件和流程编排，4 个 step 文件各自负责一个阶段（收集信息、生成 spec、生成代码、写入文件），调用现有 skill 而不复制其内容。

**Tech Stack:** Markdown skill files（Claude Code plugin 格式），不涉及编译或运行时依赖。

---

## 文件清单

| 操作 | 路径 | 职责 |
|------|------|------|
| 创建 | `plugins/antdboy/skills/09-page-builder-agent/SKILL.md` | 主入口：触发条件、流程编排、依赖声明 |
| 创建 | `plugins/antdboy/skills/09-page-builder-agent/steps/01-collect.md` | 解析意图 + 一次性收集缺失信息 |
| 创建 | `plugins/antdboy/skills/09-page-builder-agent/steps/02-plan.md` | 判断页面类型 + 生成 spec 文档 |
| 创建 | `plugins/antdboy/skills/09-page-builder-agent/steps/03-generate.md` | 按顺序调用各 skill 生成代码 |
| 创建 | `plugins/antdboy/skills/09-page-builder-agent/steps/04-write.md` | 用户确认 + 写入文件 |
| 修改 | `plugins/antdboy/plugin.json` 或版本文件 | 版本升级至 0.0.14（如存在） |

---

## Task 1：创建目录结构

**Files:**
- 创建目录：`plugins/antdboy/skills/09-page-builder-agent/`
- 创建目录：`plugins/antdboy/skills/09-page-builder-agent/steps/`

- [ ] **Step 1：确认目录不存在**

```bash
ls plugins/antdboy/skills/
```

预期输出：没有 `09-page-builder-agent`

- [ ] **Step 2：创建目录**

```bash
mkdir -p plugins/antdboy/skills/09-page-builder-agent/steps
```

- [ ] **Step 3：验证目录创建成功**

```bash
ls plugins/antdboy/skills/09-page-builder-agent/
```

预期输出：`steps/`

---

## Task 2：创建 SKILL.md（主入口）

**Files:**
- 创建：`plugins/antdboy/skills/09-page-builder-agent/SKILL.md`

- [ ] **Step 1：写入 SKILL.md**

完整内容如下：

```markdown
---
name: page-builder-agent
description: 新建中后台 Web 页面或 CRUD 模块时触发。当用户描述需要创建新页面、新模块、新管理界面时自动启动，按线性流水线编排现有 antdboy 规范，生成完整可运行模块。适用于：帮我建一个XX模块、新增一个CRUD页面、生成XX管理界面等场景。
user-invocable: true
---

# antdboy page-builder agent

## 触发条件

当用户意图为"新建中后台页面或模块"时触发，包括但不限于：
- 帮我建一个 XX 模块
- 新增一个 XX 管理页面
- 创建 XX 的 CRUD 页面
- 生成 XX 管理界面

语义匹配，不要求精确关键词。

## 依赖规范

本 agent 在各步骤中会调用以下现有 skill，执行时按需引用其内容：
- `antdboy:00-admin-page-organization`：页面组织方式判断
- `antdboy:00-antd-admin-basics`：基础工程规范
- `antdboy:01-api-organization`：接口层组织
- `antdboy:02-data-fetching`：数据请求规范
- `antdboy:03-authorized`：权限控制规范
- `antdboy:05-admin-form-standards`：表单规范
- `antdboy:06-code-comment-standards`：注释规范
- `antdboy:07-ts-type-organization`：TypeScript 类型规范
- `antdboy:08-naming-conventions`：命名规范

## 执行流程

严格按以下顺序执行，不跳步，不并行：

**Step 1-2：收集信息**
按 `steps/01-collect.md` 执行。

**Step 3-4：生成 spec 文档**
按 `steps/02-plan.md` 执行。

**Step 5：生成代码**
按 `steps/03-generate.md` 执行。

**Step 6：写入文件**
按 `steps/04-write.md` 执行。

## 约束

- 不跳过任何步骤
- 在 Step 3（用户审阅 spec）得到明确确认前，不生成代码
- 在 Step 6（用户确认写入）得到明确确认前，不写入文件
- 生成代码时，优先引用对应 skill 的规则，不自行发明规范
```

- [ ] **Step 2：验证文件存在**

```bash
ls plugins/antdboy/skills/09-page-builder-agent/SKILL.md
```

预期：文件存在，无报错

---

## Task 3：创建 steps/01-collect.md（收集信息）

**Files:**
- 创建：`plugins/antdboy/skills/09-page-builder-agent/steps/01-collect.md`

- [ ] **Step 1：写入 01-collect.md**

完整内容如下：

```markdown
# Step 1-2：解析意图与收集信息

## Step 1：解析用户意图

从用户描述中提取以下信息（能提取则不再追问）：
- 模块中文名（如"用户管理"）
- 模块英文目录名（kebab-case，如"user-management"）
- 功能描述（列表、新增、编辑、删除等）
- 是否提供了接口文档（Swagger URL 或截图）

## Step 2：一次性收集缺失信息

检查上一步提取结果，**将所有缺失的必填项整合为一条追问消息**，不分多次追问。

### 必填项清单

以下字段缺失时必须收集：

| 字段 | 说明 | 示例 |
|------|------|------|
| 模块中文名 | 页面标题和面包屑显示名 | `用户管理` |
| 模块目录名 | 页面目录，kebab-case 英文 | `user-management` |
| 接口来源 | Swagger URL 或手动描述接口列表 | `http://xxx/api-docs` |
| 原型图/截图 | 设计稿图片或链接，用于判断复杂度、字段数、布局、权限点 | 图片附件或蓝湖/Figma 链接 |

### 可选项清单

以下字段缺失时可跳过，在 spec 中标注为"待补充"：

| 字段 | 说明 |
|------|------|
| 特殊字段说明 | 联动逻辑、枚举值等无法从接口或原型推断的信息 |
| 目标目录路径 | 默认 `src/pages/<module-name>/`，如有特殊路径则收集 |

### 追问格式示例

> 为了生成完整的页面，我需要以下信息：
>
> 1. **模块中文名**：页面标题叫什么？（如：用户管理）
> 2. **接口来源**：请提供 Swagger 地址，或描述主要接口（列表查询/新增/编辑/删除）
> 3. **原型图/截图**：请上传设计稿图片或提供蓝湖/Figma 链接

## 约束

- 所有必填项确认完毕后，才进入下一步
- 不重复追问已经提供的信息
- 用户提供 Swagger URL 时，通过 WebFetch 获取接口真实定义，不自行推断字段名
```

- [ ] **Step 2：验证文件存在**

```bash
ls plugins/antdboy/skills/09-page-builder-agent/steps/01-collect.md
```

---

## Task 4：创建 steps/02-plan.md（生成 spec）

**Files:**
- 创建：`plugins/antdboy/skills/09-page-builder-agent/steps/02-plan.md`

- [ ] **Step 1：写入 02-plan.md**

完整内容如下：

```markdown
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
```

- [ ] **Step 2：验证文件存在**

```bash
ls plugins/antdboy/skills/09-page-builder-agent/steps/02-plan.md
```

---

## Task 5：创建 steps/03-generate.md（生成代码）

**Files:**
- 创建：`plugins/antdboy/skills/09-page-builder-agent/steps/03-generate.md`

- [ ] **Step 1：写入 03-generate.md**

完整内容如下：

```markdown
# Step 5：生成代码

用户确认 spec 文档后，按以下顺序生成各文件代码。每个文件生成时，严格遵循对应 skill 的规范。

## 生成顺序

### 1. apis/index.ts

遵循 `antdboy:01-api-organization` 规范：
- 导出具名 `apis` 对象，不导出散装函数
- 入参类型（`XxxQueryParams`、`XxxCreateParams`、`XxxUpdateParams`）和响应类型（`XxxItem`、`XxxDetail`）定义在同一文件
- 所有 axios 调用使用 `utils/axios.ts` 暴露的实例
- 字段名以 Swagger/接口真实定义为准，不自行推断

同时遵循 `antdboy:02-data-fetching` 中的 swagger 请求规范：通过接口真实定义确认字段名。

### 2. index.tsx（列表页容器）

遵循 `antdboy:00-admin-page-organization` 规范：
- `Search` 组件的 `form` 实例由此处创建并下传
- 使用 `useAntdTable` 承载列表查询，配置 `defaultPageSize: 20`
- 弹窗通过 `ref.show(...)` 触发，不在此维护弹窗的 boolean state

遵循 `antdboy:02-data-fetching` 规范：
- 删除、启停等操作使用 `useRequest`，`manual: true`
- loading 直接绑定在触发按钮上

遵循 `antdboy:03-authorized` 规范（如 spec 中有权限点）：
- 操作按钮用权限组件包裹
- 权限标识符使用 spec 中确认的值

### 3. components/search/index.tsx

遵循 `antdboy:00-admin-page-organization` 的 Search 组件规范：
- 只接收 `form` 和 `search` 两个 props
- 不在内部创建 `useAntdTable`
- 不发列表请求

### 4. components/<module>-form/index.tsx（Modal 模式）或 create/、edit/ 页面（独立页面模式）

遵循 `antdboy:05-admin-form-standards` 规范：
- 使用 `Form`、`Form.Item`、`forwardRef` + `useImperativeHandle`（Modal 模式）
- 提交使用 `useRequest`，loading 绑定到确认按钮
- 表单验证规则按接口字段类型设置

遵循 `antdboy:00-admin-page-organization` 的弹窗控制规范（Modal 模式）：
- 暴露 `show(options)` 方法
- 内部自行维护 `open` 状态
- 成功/取消通过 `onSuccess`/`onCancel` 回传

### 5. 类型定义

遵循 `antdboy:07-ts-type-organization` 规范：
- 接口契约类型在 `apis/index.ts` 中定义
- 表单类型（`XxxFormValues`）在表单组件文件中定义
- 组件 Props 类型在各自文件中定义
- 禁止使用 `any`

### 6. 注释

遵循 `antdboy:06-code-comment-standards` 规范：
- 函数和接口加 JSDoc 注释（中文）
- 非显而易见的逻辑加行内注释
- 不加废话注释（如"这是一个函数"）

### 7. 命名检查

遵循 `antdboy:08-naming-conventions` 规范：
- 文件和目录名一律 kebab-case
- 组件名 PascalCase
- 变量和函数名 camelCase
- 类型名 PascalCase

## 生成完毕后

所有文件代码生成完毕后，**不要自动写入磁盘**，移交 `steps/04-write.md` 处理。
```

- [ ] **Step 2：验证文件存在**

```bash
ls plugins/antdboy/skills/09-page-builder-agent/steps/03-generate.md
```

---

## Task 6：创建 steps/04-write.md（写入文件）

**Files:**
- 创建：`plugins/antdboy/skills/09-page-builder-agent/steps/04-write.md`

- [ ] **Step 1：写入 04-write.md**

完整内容如下：

```markdown
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

## 写入完毕后的提示

> 文件已创建完毕。
>
> **下一步建议：**
> 1. 检查 `apis/index.ts` 中的接口路径是否与后端一致
> 2. 在 `src/routes/` 中注册新路由
> 3. 如有权限点，在权限管理系统中配置对应标识符
>
> spec 文档保留在 `docs/page-builder-specs/<module-name>-spec.md`，可作为参考。
```

- [ ] **Step 2：验证文件存在**

```bash
ls plugins/antdboy/skills/09-page-builder-agent/steps/04-write.md
```

---

## Task 7：验证完整 skill 结构

**Files:**
- 读取验证所有已创建文件

- [ ] **Step 1：检查目录结构完整性**

```bash
find plugins/antdboy/skills/09-page-builder-agent -type f | sort
```

预期输出：
```
plugins/antdboy/skills/09-page-builder-agent/SKILL.md
plugins/antdboy/skills/09-page-builder-agent/steps/01-collect.md
plugins/antdboy/skills/09-page-builder-agent/steps/02-plan.md
plugins/antdboy/skills/09-page-builder-agent/steps/03-generate.md
plugins/antdboy/skills/09-page-builder-agent/steps/04-write.md
```

- [ ] **Step 2：检查 SKILL.md frontmatter 格式**

```bash
head -10 plugins/antdboy/skills/09-page-builder-agent/SKILL.md
```

预期：包含 `name: page-builder-agent` 和 `user-invocable: true`

- [ ] **Step 3：检查各 step 文件非空**

```bash
wc -l plugins/antdboy/skills/09-page-builder-agent/steps/*.md
```

预期：每个文件行数 > 20

---

## Task 8：更新版本号

**Files:**
- 查找并修改版本文件

- [ ] **Step 1：查找版本配置文件**

```bash
find plugins/antdboy -name "*.json" | head -5
```

- [ ] **Step 2：查看当前版本**

查看找到的 JSON 文件，找到 `version` 字段。

- [ ] **Step 3：更新版本号**

将 antdboy 插件版本从 `0.0.13` 升级到 `0.0.14`，更新对应 JSON 文件中的 version 字段。

- [ ] **Step 4：验证版本更新**

```bash
grep -r "version" plugins/antdboy --include="*.json"
```

预期：版本号已更新为 `0.0.14`

---

## Self-Review

### Spec Coverage Check

| 设计文档要求 | 对应 Task |
|-------------|-----------|
| SKILL.md 触发条件、依赖声明、流程编排 | Task 2 |
| Step1-2：解析意图 + 一次性收集缺失信息 | Task 3 |
| Step3-4：判断页面类型（Modal/独立页面）+ 写 spec | Task 4 |
| Step5：按顺序调用各 skill 生成代码 | Task 5 |
| Step6：展示写入计划 + 用户确认 + 写入文件 | Task 6 |
| 完整性验证 | Task 7 |
| 版本号升级 | Task 8 |

所有设计要求均已覆盖。

### 关键约束验证

- [x] spec 文档不自动删除（04-write.md 末尾明确保留）
- [x] 写入代码前需用户确认（04-write.md 有明确等待确认步骤）
- [x] 代码生成前需用户确认 spec（02-plan.md 有明确等待步骤）
- [x] 文件和目录命名 kebab-case（03-generate.md 命名检查步骤）
- [x] 不自动执行 git commit（整个计划无 commit 步骤）
