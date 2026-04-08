# 命名规范代码示例

按 SKILL.md 的七个维度组织，每节对应推荐写法和禁止写法。

---

## 一、文件与目录命名

```text
✅ src/pages/user-management/
✅ src/pages/order-detail/
✅ src/components/date-range-picker/index.tsx
✅ src/hooks/use-permission.ts
✅ src/utils/format-date.ts

❌ src/pages/UserManagement/       # 禁止 PascalCase 目录
❌ src/hooks/usePermission.ts      # 禁止 camelCase 文件名
❌ src/utils/formatDate.ts         # 禁止 camelCase 文件名
```

私有子组件目录：

```text
✅ src/pages/user-management/components/user-form/index.tsx
✅ src/pages/user-management/components/batch-import-modal/index.tsx

❌ src/pages/user-management/components/UserForm.tsx   # 平铺文件，难以扩展
```

---

## 二、React 组件命名

### 组件函数名

```tsx
// 文件路径：components/user-form/index.tsx
// ✅ 组件函数名 PascalCase
export default function UserForm() { ... }

// ❌ 组件函数名跟着文件名走
export default function userForm() { ... }
```

### forwardRef 组件

```tsx
// ✅ 推荐：具名函数 + XxxRef 接口
export interface UserFormRef {
  show(options?: { record?: UserItem }): void
}

const UserForm = forwardRef<UserFormRef, UserFormProps>(function UserForm(props, ref) {
  useImperativeHandle(ref, () => ({ show }))
  ...
})

export default UserForm

// ❌ 禁止：匿名函数，DevTools 无法识别组件名
export default forwardRef<UserFormRef, UserFormProps>((props, ref) => { ... })
```

### HOC 命名

```tsx
// ✅ 推荐：with 前缀 + 返回有名函数
function withPermission<T>(WrappedComponent: React.ComponentType<T>) {
  function WithPermission(props: T) { ... }
  return WithPermission
}

// ❌ 禁止：返回匿名组件
function withPermission<T>(Component: React.ComponentType<T>) {
  return (props: T) => { ... }
}
```

### Context 命名

```tsx
// ✅ 推荐：XxxContext / XxxProvider / useXxx 三件套
export const ThemeContext = createContext<ThemeContextValue>(defaultValue)
export function ThemeProvider({ children }: { children: React.ReactNode }) { ... }
export function useTheme() { return useContext(ThemeContext) }
```

---

## 三、函数与方法命名

### 事件处理函数

```tsx
// ✅ 推荐：内部处理函数用 handle
const handleSubmit = () => { ... }
const handleDelete = (id: number) => { ... }
const handleStatusChange = (value: number) => { ... }

// ❌ 禁止：内部函数使用 on 前缀（on 专属 Props 回调）
const onSubmit = () => { ... }
const onDelete = () => { ... }
```

### 接口请求函数

```tsx
// ✅ 推荐：动词 + 名词
export const apis = {
  getUserList: (params: UserQueryParams) => ...,
  getUserDetail: (id: number) => ...,
  createUser: (data: UserCreateParams) => ...,
  updateUser: (id: number, data: UserUpdateParams) => ...,
  deleteUser: (id: number) => ...,
  exportUserList: (params: UserQueryParams) => ...,
}

// ❌ 禁止：动词缺失，无法区分操作语义
export const apis = {
  userList: (params) => ...,
  userData: (data) => ...,
}
```

### 条件判断函数

```ts
// ✅ 推荐：is / has / can / should 前缀
function isValidStatus(status: number): boolean { ... }
function hasEditPermission(role: string): boolean { ... }
function canSubmitForm(values: FormValues): boolean { ... }
function shouldShowWarning(type: string): boolean { ... }

// ❌ 禁止：无前缀，返回值类型不明确
function validStatus(status: number) { ... }
function checkPermission(role: string) { ... }
```

---

## 四、变量命名

### 状态变量（useState）

```tsx
// ✅ 推荐：[noun, setNoun]
const [open, setOpen] = useState(false)
const [selectedIds, setSelectedIds] = useState<number[]>([])
const [keyword, setKeyword] = useState('')

// ❌ 禁止：冗余前后缀
const [isModalOpen, setIsModalOpen] = useState(false)  // is 前缀冗余
const [modalState, setModalState] = useState(false)    // State 后缀冗余
const [showDialog, setShowDialog] = useState(false)    // show 是动词
```

### Loading 变量

```tsx
// ✅ 推荐：从 hook 解构，多个时重命名
const { loading: submitLoading, run: handleSubmit } = useRequest(apis.createUser, { manual: true })
const { loading: deleteLoading, run: handleDelete } = useRequest(apis.deleteUser, { manual: true })

// ❌ 禁止：额外 useState loading
const [loading, setLoading] = useState(false)
const { run } = useRequest(apis.createUser, {
  manual: true,
  onBefore: () => setLoading(true),
  onFinally: () => setLoading(false),
})
```

### Ref 变量

```tsx
// ✅ 推荐：名词 + Ref 后缀
const userFormRef = useRef<UserFormRef>(null)
const tableContainerRef = useRef<HTMLDivElement>(null)

// ❌ 禁止：无语义或缺少 Ref 后缀
const ref = useRef(null)
const userForm = useRef(null)
```

### 常量命名

```ts
// ✅ 推荐：模块级配置常量用 SCREAMING_SNAKE_CASE
const DEFAULT_PAGE_SIZE = 20
const MAX_EXPORT_ROWS = 10_000
const REQUEST_TIMEOUT_MS = 30_000

// ✅ 推荐：局部不变量用 camelCase
const { id, name, ...rest } = record
const currentUser = useCurrentUser()

// ❌ 禁止：局部变量大写，夸大"不变性"语义
const { ID, NAME } = record
```

### 数据形态区分

```tsx
// ✅ 推荐：xxxList / xxxDetail / xxxOptions 区分形态
const { data: userList } = useRequest(apis.getUserList)
const { data: userDetail } = useRequest(() => apis.getUserDetail(id))
const { data: roleOptions } = useRequest(apis.getRoleOptions)

// ❌ 禁止：统一用 data，掩盖数据形态
const { data: userData } = useRequest(apis.getUserList)
```

---

## 五、自定义 Hook 命名

```ts
// ✅ 推荐：use + 名词
useUserList()
useUserForm()
usePermission()

// ❌ 禁止：动词开头
useFetchUsers()
useHandleForm()
useLoadOptions()
```

---

## 六、事件 Props 命名

```tsx
// ✅ 推荐：Props 回调用 on，内部实现用 handle
interface UserFormProps {
  onSuccess: (record: UserItem) => void
  onCancel: () => void
  onStatusChange: (status: number) => void
}

const handleFormSuccess = (record: UserItem) => { refresh() }
<UserForm onSuccess={handleFormSuccess} onCancel={() => setOpen(false)} />

// ✅ 推荐：原生事件透传，直接用原生名
interface SearchInputProps {
  onChange?: React.ChangeEventHandler<HTMLInputElement>
  onBlur?: React.FocusEventHandler<HTMLInputElement>
}

// ✅ 推荐：处理后回传，用自定义命名
interface AmountInputProps {
  onAmountChange: (amount: number) => void  // 内部已处理精度，回传数字
}

// ❌ 禁止：把原生事件包装成自定义名
interface SearchInputProps {
  onInputChange?: (value: string) => void   // onChange 的无意义重包装
}
```

---

## 七、枚举与业务常量命名

### 字符串联合类型（优先）

```ts
// ✅ 推荐：中文字符串联合类型，直观无需映射
type UserStatus = '启用' | '禁用' | '待审核'
type OrderType = '采购单' | '销售单' | '退货单'
```

### enum（需与后端数字值对齐时）

```ts
// ✅ 推荐：enum 成员名用中文，省去 LabelMap
enum OrderStatus {
  待处理 = 0,
  处理中 = 1,
  已完成 = 2,
  已取消 = 3,
}

const ORDER_STATUS_COLOR: Record<OrderStatus, string> = {
  [OrderStatus.待处理]: 'warning',
  [OrderStatus.处理中]: 'processing',
  [OrderStatus.已完成]: 'success',
  [OrderStatus.已取消]: 'default',
}

// ❌ 禁止：英文成员名 + 额外维护一份中文 LabelMap
enum OrderStatus {
  Pending = 0,
  Processing = 1,
}
const ORDER_STATUS_LABEL = {
  [OrderStatus.Pending]: '待处理',
  [OrderStatus.Processing]: '处理中',
}
```

### 选项数组与配置常量

```ts
// ✅ 推荐
const userStatusOptions = [
  { label: '启用', value: '启用' },
  { label: '禁用', value: '禁用' },
]

const ORDER_STATUS_COLOR: Record<OrderStatus, string> = { ... }

const DEFAULT_PAGE_SIZE = 20
const MAX_EXPORT_ROWS = 10_000
```
