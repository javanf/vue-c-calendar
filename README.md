# vue-c-calendar

> vue日历选择器，可以自定义一共显示多少月份，起始日期和截止日期等。

## 预览

![VUE开发一个组件——日历选择控件](http://www.javanx.cn/wp-content/themes/lensnews2.2/images/post/20181102174827.png)

## 快速开始

``` bash
# 安装
npm install vue-c-calendar -S
```

引入模块
```javascript
import cCalendar from 'vue-c-calendar'
 
Vue.use(cCalendar)

```
## 组件参数
```json
labels: {// 头部显示的文字部分（出发/到达）
  type: Array,
  required: true
},
isSame: { // 是否可以选择相同日期（起始日期）
  type: Boolean,
  default: false
},
startDate: String, // 已选择的开始时间
endDate: String, // 已选择的结束时间
showAmount: { // 共显示月份 默认3个月
  type: Number,
  default: 3
},
disableBefore: { // 禁用什么日期之前的日期（默认今天）
  type: String,
  default: formatDate(TODAY)
},
disableAfter: String, // 禁用什么日期之后的日期
start: String, // 什么月份开始（可以为空）
sameEnable: { // 是否需要判断禁用日期（可以为true）
  type: Boolean,
  default: true
}
```

