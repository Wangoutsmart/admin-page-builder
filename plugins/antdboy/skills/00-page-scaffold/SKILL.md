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
- `antdboy:10-import-order`：import 排列规范

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

**Step 7：自动合规审查**
文件写入完毕后，按 `steps/04-write.md` 中"写入完毕后：自动触发合规审查"节执行，派发 `antdboy:compliance-reviewer` agent 审查本次新生成的文件。

## 约束

- 不跳过任何步骤
- 在 Step 3（用户审阅 spec）得到明确确认前，不生成代码
- 在 Step 6（用户确认写入）得到明确确认前，不写入文件
- Step 7 不需要用户确认，写入完毕后自动执行
- 生成代码时，优先引用对应 skill 的规则，不自行发明规范

## 编码规则

- **规范空白处理**：遇到 antdboy 规范未覆盖的场景时，禁止参考项目中已有页面/组件的代码风格，应独立判断并采用你认为最优的方案
- **Table 前端分页**：使用 antd Table 展示数据时，若后端接口无分页，不得手动实现分页逻辑；保持 Table 默认 `pagination` 行为，由组件自行处理前端分页展示
