# 微信小程序开发入门教程

> **适用读者**：具有 Java Web 开发经验（Spring Boot / Spring MVC）的后端开发者
> **学习目标**：从零掌握微信小程序开发，能够独立完成内容资讯类小程序项目
> **教程特色**：Java 视角对比、完整实战项目、企业级开发规范

---

## 第5章 WXSS样式系统

WXSS（WeiXin Style Sheets）是微信小程序的样式语言，用于描述 WXML 组件的视觉表现。如果你熟悉 CSS，那么 WXSS 几乎没有学习成本——它在 CSS 基础上做了少量扩展和约束，同时新增了响应式单位 `rpx` 来适配不同尺寸的移动设备。

### 5.1 WXSS 与 CSS 的兼容性

WXSS 支持**绝大部分 CSS3 特性**，包括但不限于：

- **选择器**：类选择器（`.class`）、ID选择器（`#id`）、元素选择器（`view`）、伪类（`:hover`）、伪元素（`::before`）等
- **盒模型**：`margin`、`padding`、`border`、`width`、`height`
- **布局**：`display: flex`、`position`、`float`
- **视觉效果**：`background`、`box-shadow`、`border-radius`、`opacity`
- **文本**：`font-size`、`color`、`text-align`、`line-height`
- **动画**：`@keyframes`、`animation`、`transition`
- **变换**：`transform`（`translate`、`rotate`、`scale`）

**不支持的特性**（需要特别注意）：

| 不支持的 CSS 特性 | 替代方案 |
|-------------------|----------|
| `@import url()` 远程导入 | 仅支持本地文件 `@import` |
| 通配符选择器 `*` | 使用具体的类选择器 |
| `!important` | 尽量避免使用，依赖选择器优先级 |
| `em` / `rem` 单位 | 使用 `rpx` 响应式单位 |

> **Java 开发者注意点**
>
> 如果你之前主要做后端开发，对 CSS 不太熟悉，可以这样理解 WXSS：它就像是 HTML 元素的"外观配置文件"。在 Java Web 开发中，你可能用过 Bootstrap 或 Layui 等 UI 框架，它们本质上也是 CSS 的封装。WXSS 与这些框架的 CSS 写法完全一致，唯一的区别是新增了 `rpx` 单位来处理移动端适配。

### 5.2 rpx 响应式单位

`rpx`（responsive pixel）是 WXSS 新增的长度单位，它可以根据屏幕宽度进行自适应。小程序规定：**屏幕宽度固定为 750rpx**。

换算公式：

```
1rpx = 屏幕宽度 / 750
```

以 iPhone 6 为例（屏幕宽度 375px）：

```
1rpx = 375px / 750 = 0.5px
```

常见设备换算对照：

| 设备 | 屏幕宽度（px） | 1rpx = ? px | 750rpx = ? px |
|------|---------------|-------------|---------------|
| iPhone 5 | 320 | 0.4267 | 320 |
| iPhone 6/7/8 | 375 | 0.5 | 375 |
| iPhone 6/7/8 Plus | 414 | 0.552 | 414 |
| iPhone X/11/12 | 375 | 0.5 | 375 |
| iPhone 14 Pro Max | 430 | 0.5733 | 430 |

```css
/* 使用 rpx 的示例 */
.container {
  width: 750rpx;          /* 占满整个屏幕宽度 */
  padding: 30rpx;         /* 左右各 30rpx 内边距 */
}

.card {
  width: 690rpx;          /* 750 - 30*2 = 690rpx */
  height: 400rpx;
  border-radius: 16rpx;
  margin-bottom: 20rpx;
}

.title {
  font-size: 36rpx;       /* 约等于 18px（iPhone 6） */
  font-weight: bold;
}

.subtitle {
  font-size: 28rpx;       /* 约等于 14px（iPhone 6） */
  color: #666666;
}
```

> **类比理解：rpx 与 Bootstrap 栅格系统的对比**
>
> | 特性 | Bootstrap 栅格 | WXSS rpx |
> |------|---------------|----------|
> | 适配方式 | 12列栅格，百分比宽度 | 固定750等分，绝对宽度 |
> | 断点机制 | 需要定义 `sm/md/lg/xl` | 无需断点，自动适配 |
> | 使用方式 | `col-md-6`（占50%） | `width: 375rpx`（占50%） |
> | 精确度 | 相对比例，不够精确 | 绝对像素，精确控制 |
>
> **设计建议**：在微信开发者工具中，可以使用 750px 的设计稿，1:1 对应 rpx 值。例如设计稿上标注 `width: 200px`，在 WXSS 中直接写 `width: 200rpx` 即可。这大大简化了从设计稿到代码的转换过程。

### 5.3 样式导入 @import

使用 `@import` 语句可以导入外联样式表，`@import` 后跟需要导入的外联样式表的相对路径，用 `;` 表示语句结束。

```css
/* common.wxss - 公共样式 */
.flex-center {
  display: flex;
  justify-content: center;
  align-items: center;
}

.text-ellipsis {
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.card-shadow {
  box-shadow: 0 4rpx 12rpx rgba(0, 0, 0, 0.08);
  border-radius: 16rpx;
}
```

```css
/* pages/index/index.wxss - 页面样式 */
@import "../../common.wxss";

.container {
  padding: 30rpx;
  background-color: #f5f5f5;
}

.card {
  background-color: #ffffff;
  margin-bottom: 20rpx;
  /* 复用导入的公共样式 */
}

.title {
  font-size: 36rpx;
  font-weight: bold;
  /* 复用导入的公共样式 */
}
```

> **Java 开发者注意点**
>
> `@import` 的概念类似于 Java 中的 `import` 语句，用于引入外部定义的样式。与 CSS 预处理器（如 Sass/Less）的 `@import` 不同，WXSS 的 `@import` 是在运行时处理的，而非编译时。另外，WXSS 的 `@import` 只支持相对路径，不支持网络路径。

### 5.4 全局样式与局部样式

小程序的样式分为两个层级：

- **全局样式**（`app.wxss`）：作用于所有页面
- **局部样式**（页面级 `.wxss` 文件）：仅作用于当前页面

**优先级规则**：局部样式 > 全局样式 > app.json 中配置的样式

```css
/* app.wxss - 全局样式 */
page {
  background-color: #f5f5f5;
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
  font-size: 28rpx;
  color: #333333;
}

.container {
  padding: 30rpx;
}

/* 全局按钮样式 */
.btn-primary {
  background-color: #07c160;
  color: #ffffff;
  border-radius: 8rpx;
  padding: 20rpx 40rpx;
  text-align: center;
}
```

```css
/* pages/index/index.wxss - 局部样式（会覆盖全局样式） */
page {
  /* 覆盖全局背景色 */
  background-color: #ffffff;
}

.container {
  /* 覆盖全局内边距 */
  padding: 20rpx;
}

.btn-primary {
  /* 覆盖全局按钮颜色 */
  background-color: #1890ff;
}
```

> **类比理解：样式优先级与 Spring Boot 配置**
>
> 这与 Spring Boot 的配置优先级机制非常相似：
>
> | 层级 | WXSS | Spring Boot |
> |------|------|-------------|
> | 最高 | 页面级 `.wxss` | `application-local.yml` |
> | 中间 | `app.wxss` | `application.yml` |
> | 最低 | 组件默认样式 | 框架默认配置 |
>
> 局部配置会覆盖全局配置，这与 Spring Boot 中 `application-{profile}.yml` 覆盖 `application.yml` 的机制完全一致。

### 5.5 Flex 布局在小程序中的实践

Flex 布局是小程序中最常用的布局方式，微信小程序的 `<view>` 组件默认就是块级元素，配合 Flex 可以轻松实现各种复杂布局。

**常用 Flex 布局模式：**

```css
/* ===== 1. 水平居中 ===== */
.header {
  display: flex;
  justify-content: center;    /* 主轴（水平）居中 */
  align-items: center;        /* 交叉轴（垂直）居中 */
  height: 100rpx;
}

/* ===== 2. 两端对齐（导航栏经典布局） ===== */
.nav-bar {
  display: flex;
  justify-content: space-between;  /* 两端对齐 */
  align-items: center;
  padding: 20rpx 30rpx;
}

/* ===== 3. 等间距网格布局 ===== */
.icon-grid {
  display: flex;
  flex-wrap: wrap;            /* 允许换行 */
  justify-content: space-around; /* 均匀分布 */
}

.icon-item {
  width: 150rpx;
  height: 150rpx;
  display: flex;
  flex-direction: column;     /* 纵向排列 */
  justify-content: center;
  align-items: center;
}

/* ===== 4. 经典商品卡片布局 ===== */
.product-card {
  display: flex;
  padding: 20rpx;
  background-color: #fff;
  border-radius: 12rpx;
  margin-bottom: 20rpx;
}

.product-image {
  width: 200rpx;
  height: 200rpx;
  flex-shrink: 0;             /* 不被压缩 */
  border-radius: 8rpx;
}

.product-info {
  flex: 1;                    /* 占据剩余空间 */
  margin-left: 20rpx;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
}

.product-name {
  font-size: 30rpx;
  font-weight: bold;
  /* 文本溢出省略 */
  display: -webkit-box;
  -webkit-box-orient: vertical;
  -webkit-line-clamp: 2;
  overflow: hidden;
}

.product-bottom {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.price {
  font-size: 36rpx;
  color: #e4393c;
  font-weight: bold;
}

.buy-btn {
  background-color: #e4393c;
  color: #fff;
  padding: 10rpx 30rpx;
  border-radius: 30rpx;
  font-size: 24rpx;
}
```

```xml
<!-- 对应的 WXML 结构 -->
<view class="product-card">
  <image class="product-image" src="{{item.image}}" mode="aspectFill"/>
  <view class="product-info">
    <text class="product-name">{{item.name}}</text>
    <view class="product-bottom">
      <text class="price">{{item.price}}</text>
      <view class="buy-btn" bindtap="buyProduct" data-id="{{item.id}}">
        加入购物车
      </view>
    </view>
  </view>
</view>
```

### 5.6 与 Bootstrap 响应式设计的对比

| 对比维度 | Bootstrap | WXSS |
|---------|-----------|------|
| **适配策略** | 媒体查询 + 栅格系统 | rpx 单位自动适配 |
| **栅格实现** | 12列栅格（`col-md-6`） | Flex 布局手动实现 |
| **断点** | 576px / 768px / 992px / 1200px | 无需断点 |
| **预定义样式** | 丰富的组件类（`.btn`、`.card`） | 需要自行编写或使用第三方UI库 |
| **CSS预处理** | 内置 Sass 支持 | 原生不支持（可通过构建工具引入） |
| **UI组件库** | Bootstrap 自带 | 推荐 Vant Weapp / WeUI |

> **Java 开发者注意点**
>
> 如果你习惯使用 Bootstrap 或 Layui 等 UI 框架，在小程序开发中推荐使用以下成熟的 UI 组件库：
>
> - **Vant Weapp**：有赞出品，组件丰富，文档完善，类似 Element UI 在 Vue 中的地位
> - **WeUI**：微信官方设计团队出品，风格与微信原生一致
> - **ColorUI**：轻量级 CSS 组件库，适合快速开发
>
> 这些库的使用方式与 Bootstrap 类似——引入样式文件后，直接在 WXML 中使用预定义的类名即可。

---

