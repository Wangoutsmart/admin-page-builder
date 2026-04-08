# antdboy page-builder agent 设计文档

**日期：** 2026-04-08
**状态：** 已确认，待实现

---

## 背景与目标

antdboy 插件现有 10 个被动规范 skill（00~08），覆盖页面组织、接口层、数据请求、权限、表单、注释、类型等。这些 skill 各自约束代码质量，但没有统一的编排入口，每次新建中后台模块依赖用户手动触发各个 skill，且 skill 之间互不感知。

**目标：** 新增 `09-page-builder-agent` skill，作为线性流水线编排器，在用户意图新建中后台页面时自动触发，按顺序调用现有 skill 生成完整可运行模块，提升一致性和开发效率。

---

## 设计决策

| 决策点 | 结论 |
|--------|------|
| 输入方式 | 自然语言描述，agent 追问缺失信息（一次性列出所有缺失项） |
| 输出方式 | 写入本地 spec 文档供用户审阅，确认后写入代码文件 |
| spec 文档生命周期 | 写入后保留，不自动删除 |
| 页面类型支持 | Modal 模式 + 独立页面模式，agent 依据规范自动判断 |
| skill 位置 | `plugins/antdboy/skills/09-page-builder-agent/` |
| 触发条件 | 用户意图为"新建中后台页面/模块"时触发，宽松语义匹配 |
| 权限点 | 从原型图/截图自动识别，不单独收集 |

---

## 文件结构

```
plugins/antdboy/skills/09-page-builder-agent/
├── SKILL.md             # agent 主入口：触发条件、流程编排、依赖声明
└── steps/
    ├── 01-collect.md    # Step1-2：解析意图 + 一次性收集缺失信息
    ├── 02-plan.md       # Step3：判断页面类型 + 生成 spec 文档
    ├── 03-generate.md   # Step4：按顺序调用各 skill 生成代码
    └── 04-write.md      # Step5-6：用户确认 + 写入文件
```

---

## 执行流程（线性流水线）

### Step 1 - 解析意图
从用户描述提取：模块名（中/英）、大致功能、是否已有接口文档。

### Step 2 - 一次性收集缺失信息

**必填：**
| 字段 | 说明 | 示例 |
|------|------|------|
| 模块中文名 | 页面标题/面包屑用 | `用户管理` |
| 模块目录名 | kebab-case 英文 | `user-management` |
| 接口来源 | Swagger URL 或手动描述接口 | `http://xxx/swagger` |
| 原型图/截图 | 判断复杂度、字段数、布局、权限点 | 图片或设计稿链接 |

**可选：**
| 字段 | 说明 |
|------|------|
| 特殊字段说明 | 联动逻辑、枚举值等无法从接口推断的信息 |
| 目标目录路径 | 默认 `src/pages/<module-name>/` |

### Step 3 - 判断页面类型（依据 `antdboy:00-admin-page-organization`）
- 字段数 ≤ 10 且无复杂联动 → **Modal 模式**
- 否则 → **独立页面模式**
- 从原型图识别权限点（中文描述 → 推断标识符，如"新增"→ `<module>:create`）

### Step 4 - 写入 spec 文档
路径：`docs/page-builder-specs/<module-name>-spec.md`

内容包含：
- 基本信息（模块名、页面类型、判断依据）
- 目录结构树
- 文件职责说明
- 接口映射表
- 权限点列表（含待用户确认的标识符）
- 待确认项

### Step 5 - 用户审阅 spec
提示用户在编辑器中查看 spec 文档，确认或提出修改后继续。

### Step 6 - 生成代码（按顺序调用 skill）

| 顺序 | Skill | 生成产物 |
|------|-------|---------|
| 1 | `antdboy:01-api-organization` | `apis/index.ts`（接口函数 + 类型） |
| 2 | `antdboy:02-data-fetching` | `useRequest` / `useAntdTable` 用法 |
| 3 | `antdboy:00-admin-page-organization` | 目录结构 + `index.tsx` |
| 4 | `antdboy:05-admin-form-standards` | 表单组件 |
| 5 | `antdboy:03-authorized` | 权限包裹（依据 spec 权限点） |
| 6 | `antdboy:07-ts-type-organization` | 类型定义 |
| 7 | `antdboy:06-code-comment-standards` | 注释 |
| 8 | `antdboy:08-naming-conventions` | 命名检查 |

### Step 7 - 写入文件
将生成代码写入目标目录，spec 文档保留。

---

## spec 文档格式模板

```markdown
# <模块中文名> 页面生成计划

## 基本信息
- 模块名：user-management
- 页面类型：Modal / 独立页面
- 判断依据：字段数 X 个，无/有复杂联动

## 目录结构
src/pages/user-management/
├── apis/index.ts
├── index.tsx
└── components/
    ├── search/index.tsx
    └── user-form/index.tsx

## 文件职责说明
| 文件 | 职责 |
|------|------|
| apis/index.ts | 接口函数 + 入参/响应类型 |
| index.tsx | 列表页容器，useAntdTable |
| components/search/index.tsx | 搜索栏 |
| components/user-form/index.tsx | 新增/编辑弹窗 |

## 接口映射
| 接口 | 方法 | 用途 |
|------|------|------|
| /api/user/list | GET | 列表查询 |
| /api/user/create | POST | 新增 |
| /api/user/:id | PUT | 编辑 |
| /api/user/:id | DELETE | 删除 |

## 权限点
| 操作 | 权限标识 |
|------|----------|
| 新增 | user:create |
| 删除 | user:delete |

## 待确认项
- [ ] 编辑接口字段与新增是否复用同一表单
- [ ] 以下权限标识符请确认是否正确：user:create, user:delete
```

---

## SKILL.md frontmatter

```yaml
---
name: page-builder-agent
description: 新建中后台 Web 页面或 CRUD 模块时触发。当用户描述需要创建新页面、新模块、新管理界面时自动启动，按线性流水线编排现有 antdboy 规范，生成完整可运行模块。
user-invocable: true
---
```

依赖的现有 skill：
- `antdboy:00-admin-page-organization`
- `antdboy:00-antd-admin-basics`
- `antdboy:01-api-organization`
- `antdboy:02-data-fetching`
- `antdboy:03-authorized`
- `antdboy:05-admin-form-standards`
- `antdboy:06-code-comment-standards`
- `antdboy:07-ts-type-organization`
- `antdboy:08-naming-conventions`
