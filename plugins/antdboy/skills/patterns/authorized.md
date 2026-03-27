### 权限控制
- 需求文档或者说明文档里面没有说明要限制权限的时候，忽略以下内容，不要自己编权限名称
- 当文档或者说明明确提示功能有权限且指定了权限名称，执行以下规范的操作
明确说明需要权限的按钮或者局部页面，用 `<Authorized permission="...">` 包裹，且把按钮的显示隐藏条件、状态描述、最终作用的目标API地址一起在`Authorized`配置,
例如`<Authorized permission="权限名字" visible={status === 1} status={['状态是待发货']} apis={['POST /opt-test/shipping']}><Button>发货</Button></Authorized> `。
如果项目目录`src/components`下面没有authorized相关组件的话，参考下面模板代码生成这个组件

```typescript
import { Permvis, usePermvis } from '@pjt/permvis'

import { useUser } from '@/hooks'

interface Props {
permission: string | string[]
apis?: string[]
status?: string | string[]
visible?: boolean
children: React.ReactElement
}

export function Authorized(props: Props) {
const { children, permission, status, visible, apis = [] } = props
// 未传 visible 时默认 true；只要显式传了 visible（即使是 undefined）都按传入值判断
const hasVisibleProp = Object.prototype.hasOwnProperty.call(props, 'visible')
const isVisible = hasVisibleProp ? visible : true
const user = useUser()

const permissions = Object.keys(user.permissions)
let hasPermission = false

if (Array.isArray(permission)) {
hasPermission = permission.some((perm) => permissions.includes(perm))
} else {
hasPermission = permissions.includes(permission)
}

if (process.env.MODE === 'production') {
return !hasPermission || !isVisible ? null : children
}

return (
<Permvis permission={permission} status={status} apis={apis}>
{!hasPermission || !isVisible ? null : children}
</Permvis>
)
}
```
