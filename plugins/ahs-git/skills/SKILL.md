---
name: ahs-git
description: 自动分析改动并生成 commit message，执行 git commit 并且推送到远端
allowed-tools: Bash(git add:_), Bash(git status:_), Bash(git diff:_), Bash(git commit:_), Bash(git push:_)
disable-model-invocation: true
---

## 当前上下文

- 当前分支: !`git branch --show-current`
- 当前状态: !`git status`
- 改动内容: !`git diff HEAD`
- 最近提交: !`git log --oneline -6`

## 你的任务

任务执行步骤：

1. 分析改动的核心意图
2. 生成符合 Conventional Commits 规范的 commit message
   - 格式：`type(scope): description`
   - type 可选：feat / fix / refactor / docs / style / test / chore
3. 执行 `git add .`
4. 执行 `git commit -m "<参考commit-message.md规范>"`
5. 告诉我最终的 commit message 是什么
6. 执行 `git push` 输出push结果，同时要隐藏完整的push地址，只打印项目名称就行
