# scrollBar
## 介绍
  scrollBar是基于vue的组件,为方便生成浏览器间统一的滚动条样式
  * 滚动条可以拖动
  * 滚轮可滚动
  * 移动端可拖动
## 安装
```cmd
npm install scroll-bar-vue --sava
```
## 使用
```javascript
import scrollBar from 'scroll-bar-vue'

components:[
 scrollBar
}
```
```html
<div class='wrapper'>
 <!--wrapper的宽高决定scrollBar的宽高-->
 <scroll-bar>
  <div>
   ...<!-- 这里填写文字内容-->
  </div>
 </scroll-bar>
</div>
```
## 参数
```html
<scroll-bar :barHide='true'>
  <!--所有参数按此逻辑传递-->
</scroll-bar>
```
#### barHide
	是否隐藏滚动条 布尔值 默认true(显示)
