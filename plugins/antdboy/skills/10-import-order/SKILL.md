---
name: import-order
description: React + Ant Design + ahooks 中后台项目的 import 语句排列规范。当需要新增、修改、整理 import 语句，或新建 .tsx/.ts 文件时必须使用此 skill。确保所有 import 按照固定分组顺序排列，类型导入使用正确语法。
user-invocable: false
---

# Import 排列规范（中后台 Web 应用）

## 分组顺序

import 语句按以下 **9 个分组** 从上到下排列，**组与组之间不加空行**：

| 顺序 | 分组           | 示例                                                     |
| ---- | -------------- | -------------------------------------------------------- |
| 1    | React 核心     | `react`、`react-dom`                                     |
| 2    | Ant Design     | `antd`                                                   |
| 3    | ahooks         | `ahooks`                                                 |
| 4    | 其他第三方库   | `dayjs`、`lodash` 等                                     |
| 5    | 路由           | `react-router-dom`                                       |
| 6    | 国际化         | `react-i18next`                                          |
| 7    | 内部 scoped 包 | `@pjt/*`                                                 |
| 8    | 内部路径别名   | `@/utils`、`@/hooks`、`@/components`、`@/components/xxx` |
| 9    | 相对路径       | `../../types`、`../apis`、`./utils`                      |

## 格式规则

### 单行 vs 多行

- **≤ 5 个具名导出**：单行写法
- **> 5 个具名导出**：多行写法，每个成员独占一行，**按字母顺序排列**

```tsx
// 单行（≤5 项）
import { useRef, useState } from "react"
import { useAntdTable, useRequest } from "ahooks"

// 多行（>5 项，字母序）
import { Button, Col, Form, InputNumber, Modal, Row, Space, Table, message, type TableColumnsType } from "antd"
```

### 类型导入语法

两种合法写法，根据场景选用：

```tsx
// 1. 纯类型导入 —— 整个 import 语句只含类型
import type { BookingOrderProductItem } from "../../types"
import type { BookingOrderStatusOption } from "../../apis"

// 2. 混合导入 —— 运行时值与类型混用时，对类型成员加 type 修饰
import { Form, type FormInstance, Input } from "antd"
import { ImportFileModal, type ImportFileModalRef } from "@/components"
```

不要把运行时值和类型混写在同一个 `import type {}` 语句里。

### React 导入

React 18 使用自动 JSX 转换，**不需要** `import React from 'react'`。
只在用到 hooks 或工具函数时才写 react 导入：

```tsx
// 正确
import { useRef, useState } from "react"

// 错误（React 18 不需要）
import React from "react"
```

## 完整示例

```tsx
import { useRef, useState } from "react"
import { Button, Col, Form, InputNumber, Modal, Row, Space, Table, message, type TableColumnsType } from "antd"
import { useAntdTable, useRequest } from "ahooks"
import dayjs from "dayjs"
import { useNavigate } from "react-router-dom"
import { useTranslation } from "react-i18next"
import { PaginatedResponse } from "@pjt/axios"
import { axios, formatMoney } from "@/utils"
import { useCurrencyOptions } from "@/hooks/use-currency-options"
import { ImportFileModal, type ImportFileModalRef } from "@/components"
import { OverseasMerchantSearch } from "@/components/overseas-merchant-search"
import type { BookingOrderProductItem } from "../../types"
import type { BookingOrderStatusOption } from "../../apis"
```

## 常见错误

```tsx
// ❌ 顺序错误：@/utils 放在了 ahooks 前面
import { axios } from "@/utils"
import { useRequest } from "ahooks"

// ❌ 多行 antd 未按字母序
import { Table, Button, Form } from "antd"

// ❌ 纯类型用了普通 import
import { BookingOrderProductItem } from "../../types"

// ❌ 不必要的 React default import
import React, { useState } from "react"
```
