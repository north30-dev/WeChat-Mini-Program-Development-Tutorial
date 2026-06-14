# 微信小程序开发入门教程

> **适用读者**：具有 Java Web 开发经验（Spring Boot / Spring MVC）的后端开发者
> **学习目标**：从零掌握微信小程序开发，能够独立完成内容资讯类小程序项目
> **教程特色**：Java 视角对比、完整实战项目、企业级开发规范

---

## 第4章 WXML标记语言

WXML（WeiXin Markup Language）是微信小程序的视图层描述语言，用于构建小程序的页面结构。如果你有 HTML 和 Thymeleaf 模板引擎的开发经验，WXML 的学习曲线会非常平缓——它的核心理念是**数据驱动视图**，与 Thymeleaf 的服务端渲染思想一脉相承，但运行机制完全不同：WXML 是在客户端通过 JavaScript 数据绑定实现动态更新的。

### 4.1 WXML 与 HTML 的差异

WXML 并非 HTML 的超集，而是一套独立的标记语言。对于 Java Web 开发者来说，理解以下几点至关重要：

| 特性 | HTML | WXML |
|------|------|------|
| 标签自由度 | 可使用任意标签 | 仅支持小程序内置组件（`<view>`、`<text>`、`<image>` 等） |
| 事件绑定 | `onclick`、`onchange` | `bindtap`、`bindinput`、`catchtap` 等 |
| 数据绑定 | 依赖模板引擎（Thymeleaf/JSP） | 原生支持 `{{}}` 双花括号语法 |
| DOM 操作 | 可直接操作 DOM | **禁止直接操作 DOM**，必须通过数据驱动 |
| 组件化 | 需要框架支持（如 Vue/React） | 原生支持自定义组件 |

**核心区别**：在 Spring Boot 项目中，Thymeleaf 模板在服务端渲染完成后返回完整 HTML；而 WXML 是在客户端通过 JavaScript 的 `data` 对象动态渲染的，页面加载后仍可通过 `setData()` 实时更新视图。

### 4.2 数据绑定语法 {{}}

双花括号 `{{}}` 是 WXML 最基础也最核心的语法，用于将 Page 中 `data` 对象的数据渲染到视图上。

```xml
<!-- pages/index/index.wxml -->
<view class="container">
  <!-- 简单变量绑定 -->
  <text>{{message}}</text>

  <!-- 对象属性访问 -->
  <text>{{user.name}}</text>
  <text>{{user.age}}岁</text>

  <!-- 算术运算 -->
  <text>价格：{{price * count}}元</text>

  <!-- 三元运算符 -->
  <text>{{isVip ? 'VIP会员' : '普通用户'}}</text>

  <!-- 字符串拼接 -->
  <text>{{'欢迎，' + user.name}}</text>

  <!-- 逻辑判断 -->
  <text>{{score >= 60 ? '及格' : '不及格'}}</text>
</view>
```

```javascript
// pages/index/index.js
Page({
  data: {
    message: 'Hello 小程序',
    user: { name: '张三', age: 28 },
    price: 99.9,
    count: 2,
    isVip: true,
    score: 85
  }
})
```

> **Java 开发者注意点**
>
> WXML 的 `{{}}` 语法与 Thymeleaf 的 `${}` 表达式功能类似，但有以下关键区别：
>
> 1. **运行位置不同**：Thymeleaf 的 `${}` 在服务端渲染时求值，WXML 的 `{{}}` 在客户端运行时求值。
> 2. **更新机制不同**：Thymeleaf 每次都是完整页面渲染，WXML 通过 `setData()` 实现局部更新（类似 Vue 的响应式系统）。
> 3. **表达式限制**：WXML 的 `{{}}` 中不能调用函数（如 `{{user.getName()}}`），也不能使用 `if/for` 等流程控制语句，流程控制需要使用专门的 WXML 指令。
> 4. **不支持方法调用**：Thymeleaf 支持 `${user.getName()}`，但 WXML 只支持属性访问和简单运算。

### 4.3 条件渲染

WXML 提供了 `wx:if`、`wx:elif`、`wx:else` 三个指令来实现条件渲染，逻辑上等同于 Java 的 `if-else if-else`。

```xml
<!-- pages/order/order.wxml -->
<view class="order-container">
  <!-- 基本条件渲染 -->
  <view wx:if="{{orderStatus === 'pending'}}">
    <text>订单待支付</text>
    <button bindtap="payOrder">立即支付</button>
  </view>

  <view wx:elif="{{orderStatus === 'paid'}}">
    <text>订单已支付，等待发货</text>
  </view>

  <view wx:elif="{{orderStatus === 'shipped'}}">
    <text>订单已发货</text>
    <button bindtap="confirmReceive">确认收货</button>
  </view>

  <view wx:else>
    <text>订单已完成</text>
  </view>

  <!-- block 条件渲染（不产生实际 DOM 节点） -->
  <block wx:if="{{showDetail}}">
    <view>订单号：{{orderNo}}</view>
    <view>下单时间：{{createTime}}</view>
    <view>收货地址：{{address}}</view>
  </block>
</view>
```

```javascript
// pages/order/order.js
Page({
  data: {
    orderStatus: 'paid',
    showDetail: true,
    orderNo: 'WX20240115001',
    createTime: '2024-01-15 10:30:00',
    address: '北京市海淀区中关村大街1号'
  },

  payOrder() {
    this.setData({ orderStatus: 'paid' })
  },

  confirmReceive() {
    this.setData({ orderStatus: 'completed' })
  }
})
```

> **类比理解：条件渲染与 Thymeleaf 的对比**
>
> | WXML | Thymeleaf | Java |
> |------|-----------|------|
> | `wx:if="{{condition}}"` | `th:if="${condition}"` | `if (condition) {}` |
> | `wx:elif="{{condition}}"` | （无直接对应，需嵌套） | `else if (condition) {}` |
> | `wx:else` | `th:unless="${condition}"` | `else {}` |
> | `<block wx:if>` | `th:block th:if` | — |
>
> **`wx:if` vs `hidden` 的选择**：`wx:if` 是真正的条件渲染（节点会被创建或销毁），`hidden` 只是控制显示/隐藏（节点始终存在，类似 CSS 的 `display: none`）。频繁切换时用 `hidden` 性能更好，不常切换时用 `wx:if` 更节省资源。这类似于 Thymeleaf 中 `th:if`（不渲染）与 `th:style="display:none"`（渲染但隐藏）的区别。

### 4.4 列表渲染

`wx:for` 指令用于循环渲染列表数据，功能上等同于 JSTL 的 `<c:forEach>` 或 Thymeleaf 的 `th:each`。

```xml
<!-- pages/news/news.wxml -->
<view class="news-container">
  <!-- 基本列表渲染 -->
  <view class="news-item" wx:for="{{newsList}}" wx:key="id">
    <image src="{{item.coverUrl}}" mode="aspectFill" class="news-cover"></image>
    <view class="news-info">
      <text class="news-title">{{item.title}}</text>
      <text class="news-summary">{{item.summary}}</text>
      <view class="news-meta">
        <text>{{item.author}}</text>
        <text>{{item.publishTime}}</text>
        <text>阅读 {{item.readCount}}</text>
      </view>
    </view>
  </view>

  <!-- 自定义当前项变量名（默认为 item） -->
  <view wx:for="{{categories}}" wx:for-item="category" wx:for-index="idx" wx:key="id">
    <text>{{idx + 1}}. {{category.name}}</text>
  </view>

  <!-- 渲染嵌套列表 -->
  <view wx:for="{{sections}}" wx:key="title">
    <view class="section-title">{{item.title}}</view>
    <view wx:for="{{item.articles}}" wx:for-item="article" wx:key="id">
      <text>{{article.name}}</text>
    </view>
  </view>
</view>
```

```javascript
// pages/news/news.js
Page({
  data: {
    newsList: [
      {
        id: 1,
        title: '微信小程序开发最佳实践',
        summary: '本文总结了小程序开发中的常见问题和解决方案...',
        author: '技术团队',
        publishTime: '2024-01-15',
        readCount: 12580,
        coverUrl: '/images/news1.jpg'
      },
      {
        id: 2,
        title: 'Spring Boot 微服务架构实战',
        summary: '从零搭建一个生产级的微服务架构...',
        author: '架构师',
        publishTime: '2024-01-14',
        readCount: 8920,
        coverUrl: '/images/news2.jpg'
      },
      {
        id: 3,
        title: '前端性能优化深度指南',
        summary: '全面解析前端性能优化的各个方面...',
        author: '前端团队',
        publishTime: '2024-01-13',
        readCount: 6340,
        coverUrl: '/images/news3.jpg'
      }
    ],
    categories: [
      { id: 'cat1', name: '后端开发' },
      { id: 'cat2', name: '前端开发' },
      { id: 'cat3', name: '移动开发' }
    ],
    sections: [
      {
        title: '推荐阅读',
        articles: [
          { id: 'a1', name: 'Java并发编程' },
          { id: 'a2', name: 'JVM调优指南' }
        ]
      },
      {
        title: '热门文章',
        articles: [
          { id: 'a3', name: 'Docker入门' },
          { id: 'a4', name: 'Kubernetes实战' }
        ]
      }
    ]
  }
})
```

> **Java 开发者注意点**
>
> 1. **`wx:key` 的重要性**：`wx:key` 类似于 React 中列表渲染的 `key` 属性，用于帮助框架高效地 diff 和更新列表。建议始终提供唯一标识作为 `wx:key` 的值，这就像数据库表的主键一样重要。
>
> 2. **默认变量名**：`wx:for` 默认使用 `item` 代表当前项，`index` 代表当前索引。可以通过 `wx:for-item` 和 `wx:for-index` 自定义命名，这在嵌套循环中尤为重要（类似 Java 增强for循环中的变量名）。
>
> 3. **与 JSTL forEach 的对比**：
>    ```xml
>    <!-- JSTL 写法 -->
>    <c:forEach items="${newsList}" var="item" varStatus="status">
>      <div>${item.title}</div>
>    </c:forEach>
>
>    <!-- WXML 写法 -->
>    <view wx:for="{{newsList}}" wx:for-item="item" wx:key="id">
>      <text>{{item.title}}</text>
>    </view>
>    ```

### 4.5 模板 template 与引用

WXML 提供了模板（template）机制来实现代码复用，类似于 Thymeleaf 的 `th:fragment` 和 `th:insert`。

**定义模板（templates/news-item.wxml）：**

```xml
<!-- templates/news-item.wxml -->
<template name="newsItem">
  <view class="news-item">
    <image src="{{coverUrl}}" mode="aspectFill" class="news-cover"></image>
    <view class="news-info">
      <text class="news-title">{{title}}</text>
      <text class="news-summary">{{summary}}</text>
      <view class="news-meta">
        <text class="author">{{author}}</text>
        <text class="time">{{publishTime}}</text>
      </view>
    </view>
  </view>
</template>

<!-- 另一个模板示例 -->
<template name="userCard">
  <view class="user-card">
    <image src="{{avatar}}" class="avatar"></image>
    <text class="username">{{name}}</text>
    <text class="bio">{{bio}}</text>
  </view>
</template>
```

**使用 import 引入模板（只能使用 template）：**

```xml
<!-- pages/index/index.wxml -->
<import src="../../templates/news-item.wxml"/>

<view class="container">
  <!-- 使用 is 属性指定模板名称，data 属性传入数据 -->
  <template is="newsItem" data="{{...newsList[0]}}" />
  <template is="newsItem" data="{{...newsList[1]}}" />

  <!-- 循环中使用模板 -->
  <template is="newsItem" wx:for="{{newsList}}" wx:key="id" data="{{...item}}" />
</view>
```

**使用 include 引入（直接嵌入 WXML 片段，不涉及 template）：**

```xml
<!-- templates/header.wxml -->
<view class="header">
  <text>{{title}}</text>
  <text>{{subtitle}}</text>
</view>

<!-- pages/index/index.wxml -->
<include src="../../templates/header.wxml"/>
```

> **类比理解：import vs include**
>
> | WXML | Thymeleaf | 说明 |
> |------|-----------|------|
> | `<import>` + `<template is>` | `th:insert` / `th:replace` | 引入模板定义，按需使用，支持传参 |
> | `<include>` | `th:fragment` 内联展开 | 直接将目标文件的 WXML 片段嵌入当前位置 |
>
> **关键区别**：`import` 有作用域概念，只会导入目标文件中定义的 `template`，不会导入目标文件 `import` 的其他模板（类似 Java 的包导入）；`include` 则是简单的文件内容嵌入（类似 C 语言的 `#include`）。

### 4.6 完整示例：新闻列表页面

下面通过一个完整的新闻列表页面，综合运用本章所学的 WXML 核心语法。

**页面结构（pages/news/index.wxml）：**

```xml
<import src="../../templates/news-item.wxml"/>

<view class="news-page">
  <!-- 顶部搜索栏 -->
  <view class="search-bar">
    <icon type="search" size="14" color="#999"/>
    <input placeholder="搜索新闻" bindinput="onSearch" value="{{keyword}}"/>
  </view>

  <!-- 分类标签 -->
  <scroll-view scroll-x class="category-bar">
    <view
      wx:for="{{categories}}"
      wx:key="id"
      class="category-tag {{currentCategory === item.id ? 'active' : ''}}"
      bindtap="switchCategory"
      data-id="{{item.id}}"
    >
      {{item.name}}
    </view>
  </scroll-view>

  <!-- 新闻列表 -->
  <view class="news-list">
    <!-- 加载状态 -->
    <view wx:if="{{loading}}" class="loading">
      <text>加载中...</text>
    </view>

    <!-- 空状态 -->
    <view wx:elif="{{newsList.length === 0}}" class="empty">
      <text>暂无新闻</text>
    </view>

    <!-- 正常列表 -->
    <block wx:else>
      <template
        is="newsItem"
        wx:for="{{newsList}}"
        wx:key="id"
        data="{{...item}}"
      />
    </block>
  </view>

  <!-- 加载更多 -->
  <view wx:if="{{hasMore}}" class="load-more" bindtap="loadMore">
    <text>{{loadingMore ? '加载中...' : '点击加载更多'}}</text>
  </view>
</view>
```

**页面逻辑（pages/news/index.js）：**

```javascript
Page({
  data: {
    keyword: '',
    categories: [
      { id: 'all', name: '全部' },
      { id: 'tech', name: '科技' },
      { id: 'finance', name: '财经' },
      { id: 'sports', name: '体育' }
    ],
    currentCategory: 'all',
    newsList: [],
    loading: false,
    loadingMore: false,
    hasMore: true,
    page: 1
  },

  onLoad() {
    this.fetchNews()
  },

  // 切换分类
  switchCategory(e) {
    const categoryId = e.currentTarget.dataset.id
    this.setData({
      currentCategory: categoryId,
      newsList: [],
      page: 1,
      hasMore: true
    })
    this.fetchNews()
  },

  // 搜索
  onSearch(e) {
    this.setData({
      keyword: e.detail.value,
      newsList: [],
      page: 1
    })
    this.fetchNews()
  },

  // 获取新闻数据
  fetchNews() {
    this.setData({ loading: true })
    // 模拟网络请求（实际开发中使用 wx.request）
    setTimeout(() => {
      const mockData = [
        {
          id: 1,
          title: '微信小程序开发最佳实践',
          summary: '本文总结了小程序开发中的常见问题...',
          author: '技术团队',
          publishTime: '2024-01-15',
          coverUrl: '/images/news1.jpg'
        }
      ]
      this.setData({
        newsList: mockData,
        loading: false
      })
    }, 500)
  },

  // 加载更多
  loadMore() {
    if (this.data.loadingMore || !this.data.hasMore) return
    this.setData({ loadingMore: true, page: this.data.page + 1 })
    // 模拟分页加载
    setTimeout(() => {
      this.setData({ loadingMore: false, hasMore: false })
    }, 800)
  }
})
```

---

