--- 
  name: antdboy
  description: 使用 Ant Design 5 + TypeScript + ahooks 开发 React 中后台管理系统时，遵循此规范进行页面组织、数据请求和权限控制
---

## 1. 目录结构

```
src/
├── components/
│   └── [feature]/        # 业务组件（不含数据请求）
├── hooks/                # 自定义 Hooks
├── layouts/              # BasicLayout、MicroLayout 等
├── pages/
│   └── [module]/
│       ├── index.tsx     # 页面容器（数据编排）
│       └── components/   # 页面私有子组件
├── routes/
│   ├── index.tsx         # 路由树（所有页面懒加载）
│   └── [module].tsx      # 页面路由
├── styles/
│   └── globals.css
├── types/                # 全局 TS 类型
└── utils/                # 全局工具函数
```

## 2. 全局基础规范

@./patterns/basic-standards.md

## 3. 页面组织模式

@./patterns/page-organization.md

## 4. 涉及到接口请求相关的逻辑请参考

@./patterns/data-fetching.md

## 5. 涉及到权限相关的规范

@./patterns/authorized.md

## 6. 数据收集和处理相关规范

@./patterns/data-processing.md
