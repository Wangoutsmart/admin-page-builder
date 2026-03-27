## 全局基础规范

- 实现的页面，全部使用antd自有组件，非必要不写原生dom，如果没有antd组件，则输出日志询问下一步操作
- 非必要不写样式文件，不新建less、css、scss文件
- 禁止 `any`，用 `unknown` + 类型守卫替代
- 禁止覆写 antd CSS 类名
- 请求接口使用 axios + ahooks `useRequest或者useAntdTable`


### 按钮使用逻辑

- 当需要实现一个按钮时，优先使用antd的Button组件，link形式的按钮非必要不用a标签

### 下拉框Select实现逻辑

- 使用Antd的Select组件
- 默认下拉框选择的，都需要支持showSearch，如果不是说明从实时从远端搜索的话，设置optionFilterProp="children"
- 默认都有allowClear
- 实时从远端搜索内容的话需要实现对应的loading效果以及在对应接口增加防抖，避免多次重复请求

### Form规范

- Form的labelCol要显式的写在Form组件，span的大小要参考所有formItem的最大label的长度，并实现大小屏幕的弹性展示
- Form.Item的rules
  - 参考用户提示或者设计稿图片，如果明确说明该项是必填或者有红色*，则在rules标明必传
  - 如果明确该项只能输入数字或者保留两位小数之类的，如果通过增加InputNumber上的原生属性不能完全限制，则在rules需要限制pattern，至于正则怎么写，依据真实情况实时判断

