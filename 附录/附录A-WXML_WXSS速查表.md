# 微信小程序开发入门教程

> **适用读者**：具有 Java Web 开发经验（Spring Boot / Spring MVC）的后端开发者
> **学习目标**：从零掌握微信小程序开发，能够独立完成内容资讯类小程序项目
> **教程特色**：Java 视角对比、完整实战项目、企业级开发规范

---

## 附录A：WXML / WXSS 速查表

### A.1 WXML 常用标签与属性速查

#### 基础容器标签

| 标签 | 说明 | 常用属性 | 类比 HTML |
|------|------|----------|-----------|
| `<view>` | 视图容器（类似 div） | `hover-class`, `hover-start-time`, `hover-stay-time` | `<div>` |
| `<text>` | 文本（只能包含文本） | `selectable`, `space`, `decode` | `<span>` |
| `<scroll-view>` | 可滚动视图区域 | `scroll-x`, `scroll-y`, `scroll-top`, `scroll-left`, `scroll-into-view`, `bindscroll`, `upper-threshold`, `lower-threshold` | `<div style="overflow:auto">` |
| `<swiper>` | 滑块视图容器 | `indicator-dots`, `autoplay`, `interval`, `duration`, `current`, `circular` | 轮播图组件 |
| `<swiper-item>` | 滑块项 | — | 轮播子项 |
| `<movable-area>` | 可移动视图区域 | `scale-area` | — |
| `<movable-view>` | 可移动视图 | `direction`, `inertia`, `out-of-bounds`, `x`, `y`, `scale`, `scale-min`, `scale-max` | — |

#### 基础内容标签

| 标签 | 说明 | 常用属性 | 类比 HTML |
|------|------|----------|-----------|
| `<image>` | 图片 | `src`, `mode`, `lazy-load`, `binderror`, `bindload` | `<img>` |
| `<icon>` | 图标 | `type`, `size`, `color` | — |
| `<progress>` | 进度条 | `percent`, `show-info`, `stroke-width`, `activeColor`, `backgroundColor` | `<progress>` |
| `<rich-text>` | 富文本 | `nodes`, `bindtap` | `innerHTML` |

#### 表单标签

| 标签 | 说明 | 常用属性 | 类比 HTML |
|------|------|----------|-----------|
| `<button>` | 按钮 | `size`, `type`, `plain`, `disabled`, `loading`, `open-type`, `form-type`, `bindgetuserinfo` | `<button>` |
| `<input>` | 输入框 | `type`, `value`, `placeholder`, `password`, `disabled`, `maxlength`, `focus`, `bindinput`, `bindconfirm`, `bindblur` | `<input>` |
| `<textarea>` | 多行输入框 | `value`, `placeholder`, `disabled`, `maxlength`, `auto-height`, `focus`, `bindinput` | `<textarea>` |
| `<form>` | 表单 | `bindsubmit`, `bindreset`, `report-submit` | `<form>` |
| `<checkbox>` | 复选框 | `value`, `checked`, `disabled`, `bindchange` | `<input type="checkbox">` |
| `<checkbox-group>` | 复选框组 | `bindchange` | — |
| `<radio>` | 单选框 | `value`, `checked`, `disabled`, `bindchange` | `<input type="radio">` |
| `<radio-group>` | 单选框组 | `bindchange` | — |
| `<switch>` | 开关选择器 | `checked`, `disabled`, `color`, `bindchange` | `<input type="switch">` |
| `<slider>` | 滑动选择器 | `min`, `max`, `step`, `value`, `disabled`, `show-value`, `activeColor`, `bindchange` | `<input type="range">` |
| `<picker>` | 从底部弹起的滚动选择器 | `mode`, `range`, `range-key`, `value`, `bindchange`, `bindcancel` | `<select>` |
| `<picker-view>` | 嵌入页面的滚动选择器 | `value`, `indicator-style`, `bindchange` | — |
| `<label>` | 标签 | `for` | `<label>` |

#### 导航标签

| 标签 | 说明 | 常用属性 | 类比 HTML |
|------|------|----------|-----------|
| `<navigator>` | 页面链接 | `url`, `open-type`, `delta`, `target`, `hover-class` | `<a>` |

> **open-type 可选值：** `navigate` | `redirect` | `switchTab` | `reLaunch` | `navigateBack`

#### 媒体标签

| 标签 | 说明 | 常用属性 |
|------|------|----------|
| `<audio>` | 音频 | `src`, `controls`, `loop`, `autoplay`, `bindplay` 等 |
| `<video>` | 视频 | `src`, `controls`, `autoplay`, `loop`, `muted`, `poster`, `bindplay` 等 |
| `<camera>` | 相机 | `device-position`, `flash`, `binderror` |
| `<map>` | 地图 | `latitude`, `longitude`, `scale`, `markers`, `polyline`, `bindregionchange` |

#### `<image>` 的 mode 属性详解

| mode 值 | 说明 |
|---------|------|
| `scaleToFill` | 不保持纵横比缩放，使图片完全填满（默认） |
| `aspectFit` | 保持纵横比缩放，使图片长边完全显示 |
| `aspectFill` | 保持纵横比缩放，只保证图片短边完全显示 |
| `widthFix` | 宽度不变，高度自动变化 |
| `top` / `bottom` / `center` / `left` / `right` | 不缩放，只裁剪 |

#### `<picker>` 的 mode 属性详解

| mode 值 | 说明 | 额外属性 |
|---------|------|----------|
| `selector` | 普通选择器 | `range`, `range-key` |
| `multiSelector` | 多列选择器 | `range`, `range-key` |
| `time` | 时间选择器 | — |
| `date` | 日期选择器 | `start`, `end`, `fields` |
| `region` | 省市区选择器 | `custom-item` |

---

### A.2 数据绑定语法速查

#### 数据绑定

```xml
<!-- 文本绑定 -->
<text>{{ message }}</text>

<!-- 组件属性绑定 -->
<view id="item-{{ id }}">内容</view>

<!-- 控制属性绑定（不需要 {{}}） -->
<view wx:if="{{ condition }}">条件渲染</view>

<!-- 关键字绑定（需要在双引号内） -->
<view wx:for="{{ array }}" wx:key="id">
  {{ item.name }}
</view>

<!-- 算术运算 -->
<view>{{ a + b }}</view>
<view>{{ "hello" + name }}</view>

<!-- 逻辑判断 -->
<view>{{ flag ? '是' : '否' }}</view>

<!-- 字符串运算 -->
<view>{{ "第" + index + "项" }}</view>

<!-- 数据路径运算 -->
<view>{{ object.key }}</view>
<view>{{ array[0].name }}</view>
```

#### 条件渲染

```xml
<!-- wx:if / wx:elif / wx:else -->
<view wx:if="{{ length > 5 }}">大于5</view>
<view wx:elif="{{ length > 2 }}">大于2</view>
<view wx:else">其他</view>

<!-- block 条件渲染（不产生实际节点） -->
<block wx:if="{{ true }}">
  <view>视图1</view>
  <view>视图2</view>
</block>

<!-- wx:if vs hidden 的区别 -->
<!-- wx:if：条件为 false 时节点不渲染（适合切换频率低的场景） -->
<!-- hidden：节点始终渲染，通过 display 控制显隐（适合切换频率高的场景） -->
<view wx:if="{{ show }}">动态创建</view>
<view hidden="{{ !show }}">始终存在</view>
```

#### 列表渲染

```xml
<!-- 基本用法 -->
<view wx:for="{{ array }}" wx:key="id">
  索引：{{ index }}，当前项：{{ item }}
</view>

<!-- 指定变量名 -->
<view wx:for="{{ array }}" wx:key="id" wx:for-index="idx" wx:for-item="itemName">
  索引：{{ idx }}，当前项：{{ itemName }}
</view>

<!-- block 列表渲染 -->
<block wx:for="{{ array }}" wx:key="id">
  <view>{{ item.name }}</view>
  <view>{{ item.desc }}</view>
</block>

<!-- wx:key 的取值 -->
<!-- 字符串：代表 item 中某个唯一属性名 -->
<!-- *this：代表 item 本身（当 item 是字符串或数字时） -->
```

#### 模板（template）

```xml
<!-- 定义模板 -->
<template name="msgItem">
  <view>
    <text>{{ index }}: {{ msg }}</text>
    <text>时间：{{ time }}</text>
  </view>
</template>

<!-- 使用模板（data 传入数据） -->
<template is="msgItem" data="{{ index: index, msg: item.msg, time: item.time }}" />

<!-- 使用模板（展开运算符传入对象） -->
<template is="msgItem" data="{{ ...item }}" />

<!-- 模板作用域 -->
<!-- 模板拥有自己的作用域，只能使用 data 传入的数据 -->
```

#### WXS 模块（WeiXin Script）

```xml
<!-- WXS 代码可以写在 wxml 文件中的 <wxs> 标签内 -->
<wxs module="filters">
  module.exports = {
    formatPrice: function(price) {
      return '¥' + Number(price).toFixed(2);
    },
    substring: function(str, len) {
      return str.length > len ? str.substring(0, len) + '...' : str;
    }
  }
</wxs>

<!-- 使用 WXS -->
<text>{{ filters.formatPrice(price) }}</text>

<!-- 也可以引用外部 wxs 文件 -->
<wxs src="../../utils/filters.wxs" module="filters" />
```

> **WXS 注意事项：**
> - WXS 运行在渲染层，JS 运行在逻辑层，二者不能直接互相调用
> - WXS 模块不支持 ES6 语法（如 `let`、`const`、箭头函数）
> - WXS 适合做格式化等轻量计算，减少 setData 数据量

---

### A.3 WXSS 常用样式速查

#### 尺寸单位：rpx

```
rpx（responsive pixel）：根据屏幕宽度进行自适应
规定屏幕宽为 750rpx（不论什么设备）
在 iPhone6 上：1rpx = 0.5px = 1物理像素

常见设备换算：
┌──────────────┬────────────┬──────────────┐
│    设备       │  屏幕宽度   │  1rpx = ?px  │
├──────────────┼────────────┼──────────────┤
│ iPhone 5     │  320px     │  0.42px      │
│ iPhone 6/7/8 │  375px     │  0.5px       │
│ iPhone 6 Plus│  414px     │  0.552px     │
│ iPhone X/11  │  375px     │  0.5px       │
└──────────────┴────────────┴──────────────┘

建议：设计稿使用 750px 宽度，1:1 使用 rpx 单位
```

#### 全局样式与局部样式

```
app.wxss        → 全局样式，作用于所有页面
page/*.wxss     → 页面局部样式，只作用于当前页面
page {}         → 页面根节点样式（类似 html {}）

优先级：行内样式 > 页面 WXSS > 全局 WXSS
```

#### @import 导入

```css
/* 导入外部样式文件 */
@import "../../common/common.wxss";

/* 只支持相对路径 */
```

#### Flex 布局速查

```css
/* ========== 容器属性 ========== */

/* 开启 flex 布局 */
display: flex;

/* 主轴方向 */
flex-direction: row;            /* 水平，起点在左（默认） */
flex-direction: row-reverse;    /* 水平，起点在右 */
flex-direction: column;         /* 垂直，起点在上 */
flex-direction: column-reverse; /* 垂直，起点在下 */

/* 换行方式 */
flex-wrap: nowrap;    /* 不换行（默认） */
flex-wrap: wrap;      /* 换行，第一行在上方 */
flex-wrap: wrap-reverse; /* 换行，第一行在下方 */

/* 主轴对齐 */
justify-content: flex-start;    /* 左对齐（默认） */
justify-content: flex-end;      /* 右对齐 */
justify-content: center;        /* 居中 */
justify-content: space-between; /* 两端对齐，项目间隔相等 */
justify-content: space-around;  /* 项目两侧间隔相等 */
justify-content: space-evenly;  /* 项目间隔完全相等 */

/* 交叉轴对齐 */
align-items: stretch;       /* 拉伸填满（默认） */
align-items: flex-start;    /* 顶部对齐 */
align-items: flex-end;      /* 底部对齐 */
align-items: center;        /* 居中对齐 */
align-items: baseline;      /* 文本基线对齐 */

/* 多轴对齐 */
align-content: flex-start;   /* 多行时，整体顶部对齐 */
align-content: center;       /* 多行时，整体居中 */
align-content: space-between;/* 多行时，两端对齐 */

/* ========== 项目属性 ========== */

/* 弹性伸缩 */
flex: 1;           /* 等分剩余空间 */
flex: 2;           /* 占 2 份 */
flex: none;        /* 不伸缩 */
flex: auto;        /* 自动伸缩 */

/* 单独对齐 */
align-self: auto;       /* 继承容器（默认） */
align-self: flex-start; /* 顶部对齐 */
align-self: flex-end;   /* 底部对齐 */
align-self: center;     /* 居中 */

/* 排序 */
order: 0;    /* 默认值，数值越小越靠前 */
order: -1;   /* 排在前面 */
order: 1;    /* 排在后面 */
```

#### 常用 CSS 属性速查

```css
/* ========== 定位 ========== */
position: static;    /* 静态定位（默认） */
position: relative;  /* 相对定位 */
position: absolute;  /* 绝对定位（相对最近的非 static 祖先） */
position: fixed;     /* 固定定位（相对屏幕） */

/* ========== 盒模型 ========== */
box-sizing: border-box;  /* width 包含 padding 和 border（推荐全局设置） */
margin: 0 auto;          /* 水平居中 */
padding: 10rpx 20rpx;    /* 上下10 左右20 */

/* ========== 文本 ========== */
font-size: 28rpx;
font-weight: bold;
color: #333333;
text-align: center;
line-height: 1.5;
text-overflow: ellipsis;  /* 文本溢出省略号（需配合以下属性） */
white-space: nowrap;
overflow: hidden;

/* 多行文本省略 */
display: -webkit-box;
-webkit-box-orient: vertical;
-webkit-line-clamp: 2;
overflow: hidden;

/* ========== 背景 ========== */
background-color: #f5f5f5;
background-image: url('');
background-size: cover;      /* 覆盖 */
background-size: contain;    /* 包含 */
background-position: center;

/* ========== 边框 ========== */
border: 1rpx solid #e5e5e5;
border-radius: 8rpx;         /* 圆角 */
border-top: 1rpx solid #ddd;

/* ========== 阴影 ========== */
box-shadow: 0 2rpx 8rpx rgba(0, 0, 0, 0.1);

/* ========== 显示与隐藏 ========== */
display: none;       /* 不显示，不占空间 */
display: block;      /* 块级 */
display: inline-block; /* 行内块 */
visibility: hidden;  /* 隐藏，占空间 */
opacity: 0;          /* 透明度为0，占空间 */

/* ========== 溢出 ========== */
overflow: hidden;    /* 隐藏溢出 */
overflow: scroll;    /* 始终显示滚动条 */
overflow: auto;      /* 需要时显示滚动条 */

/* ========== 过渡动画 ========== */
transition: all 0.3s ease;
transition: transform 0.3s, opacity 0.3s;
transform: translateX(100%);  /* 水平移动 */
transform: translateY(-50%);  /* 垂直移动 */
transform: scale(1.1);        /* 缩放 */
transform: rotate(45deg);     /* 旋转 */

/* ========== 安全区域适配（刘海屏） */
padding-bottom: env(safe-area-inset-bottom);
```

---

### A.4 事件绑定速查

#### 事件绑定方式

| 方式 | 说明 | 冒泡 |
|------|------|------|
| `bind:eventName` | 绑定事件，事件会冒泡 | 是 |
| `catch:eventName` | 绑定事件，阻止事件冒泡 | 否 |
| `mut-bind:eventName` | 互斥绑定，同组事件只触发一个 | 是 |

#### 常用事件列表

| 事件名 | 说明 | 触发时机 |
|--------|------|----------|
| `tap` | 触摸后离开 | 手指触摸后离开 |
| `longpress` | 长按（推荐，替代 longtap） | 手指触摸后超过 350ms |
| `longtap` | 长按（已废弃，建议用 longpress） | 手指触摸后超过 350ms |
| `touchstart` | 触摸开始 | 手指触摸动作开始 |
| `touchmove` | 触摸移动 | 手指触摸后移动 |
| `touchend` | 触摸结束 | 手指触摸动作结束 |
| `touchcancel` | 触摸打断 | 手指触摸被打断（如来电） |
| `input` | 输入事件 | 键盘输入时 |
| `confirm` | 确认事件 | 点击完成按钮时 |
| `focus` | 获取焦点 | 输入框获取焦点 |
| `blur` | 失去焦点 | 输入框失去焦点 |
| `submit` | 表单提交 | 表单提交时 |
| `reset` | 表单重置 | 表单重置时 |
| `scroll` | 滚动事件 | 滚动时触发 |
| `change` | 值改变 | 值改变时触发 |

#### 事件绑定示例

```xml
<!-- tap 事件 -->
<view bindtap="handleTap">点击我</view>
<view catchtap="handleTap">点击我（阻止冒泡）</view>

<!-- 传参：通过 data-* 属性 -->
<view bindtap="handleTap" data-id="{{ item.id }}" data-name="{{ item.name }}">
  点击传参
</view>

<!-- input 事件 -->
<input bindinput="handleInput" placeholder="请输入" />

<!-- confirm 事件 -->
<input bindconfirm="handleConfirm" placeholder="输入后按确认" />

<!-- form 事件 -->
<form bindsubmit="handleSubmit" bindreset="handleReset">
  <input name="username" />
  <button form-type="submit">提交</button>
  <button form-type="reset">重置</button>
</form>

<!-- scroll 事件 -->
<scroll-view bindscroll="handleScroll" scroll-y style="height: 300rpx;">
  <view style="height: 1000rpx;">可滚动内容</view>
</scroll-view>

<!-- 事件对象 e 的常用属性 -->
<!-- e.currentTarget.dataset  → 当前元素上的 data-* 数据 -->
<!-- e.target.dataset         → 触发事件的源元素上的 data-* 数据 -->
<!-- e.detail                 → 额外信息（如 input 事件的 detail.value） -->
```

#### 事件对象（e）结构

```javascript
Page({
  handleTap: function(e) {
    console.log(e.type);           // 事件类型，如 "tap"
    console.log(e.timeStamp);      // 页面打开到触发事件经过的毫秒数
    console.log(e.target.id);      // 触发事件的源组件 id
    console.log(e.target.dataset); // 源组件上 data-* 数据
    console.log(e.currentTarget.id);      // 当前组件 id
    console.log(e.currentTarget.dataset); // 当前组件 data-* 数据
    console.log(e.detail);         // 额外信息
    console.log(e.touches);        // 触摸事件，当前触摸点信息
    console.log(e.changedTouches); // 触摸事件，变化的触摸点信息
  },

  handleInput: function(e) {
    console.log(e.detail.value);   // input 输入的值
  },

  handleSubmit: function(e) {
    console.log(e.detail.value);   // form 表单数据 { username: "xxx" }
  }
});
```

---

