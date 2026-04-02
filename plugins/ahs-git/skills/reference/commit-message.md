## 提交规范

- 当ahs-git:git-push接收的有参数的时候，git commit的message就要完全使用这个参数
- 自己生成git message的时候必须用中文，除非是专业术语用英文
- 没有参数的时候，如果是修复bug的提交，则message的形式是 `fix: 总结的修复问题的描述`
- 如果是新功能，message则用`feat: XXXX`

## 重要
- 任何时候都不允许将默认的message尾巴`Co-Authored-By: Claude Opus 4.6 <noreply@anthropic.com>`带上！！！
- commit message的内容不管是fix还是feat还是chore,后面都不需要有(git-push、ahs-git)此类内容