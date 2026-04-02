# Authorized Component Template

```tsx
import { Permvis, usePermvis } from "@pjt/permvis"

import { useUser } from "@/hooks"

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
  const hasVisibleProp = Object.prototype.hasOwnProperty.call(props, "visible")
  const isVisible = hasVisibleProp ? visible : true
  const user = useUser()

  const permissions = Object.keys(user.permissions)
  let hasPermission = false

  if (Array.isArray(permission)) {
    hasPermission = permission.some((perm) => permissions.includes(perm))
  } else {
    hasPermission = permissions.includes(permission)
  }

  if (process.env.MODE === "production") {
    return !hasPermission || !isVisible ? null : children
  }

  return (
    <Permvis permission={permission} status={status} apis={apis}>
      {!hasPermission || !isVisible ? null : children}
    </Permvis>
  )
}
```
