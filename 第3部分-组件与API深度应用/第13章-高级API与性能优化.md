# 微信小程序开发入门教程

> **适用读者**：具有 Java Web 开发经验（Spring Boot / Spring MVC）的后端开发者
> **学习目标**：从零掌握微信小程序开发，能够独立完成内容资讯类小程序项目
> **教程特色**：Java 视角对比、完整实战项目、企业级开发规范

---

## 第13章 高级API与性能优化

当小程序功能逐渐丰富、页面数量增多后，性能优化和代码组织变得至关重要。本章介绍分包加载、全局状态管理和页面间数据传递等高级主题。

### 13.1 分包加载

小程序包大小限制：**主包不超过 2MB**，**单个分包不超过 2MB**，**所有包总大小不超过 20MB**。对于内容资讯类应用，将不同功能模块拆分为独立分包是必要的。

```json
// app.json —— 分包配置
{
  "pages": [
    "pages/index/index",
    "pages/category/category",
    "pages/profile/profile"
  ],
  "subPackages": [
    {
      "root": "packageArticle",
      "name": "article",
      "pages": [
        "pages/detail/detail",
        "pages/comment/comment"
      ]
    },
    {
      "root": "packageUser",
      "name": "user",
      "pages": [
        "pages/settings/settings",
        "pages/history/history",
        "pages/favorites/favorites"
      ]
    },
    {
      "root": "packageSearch",
      "name": "search",
      "pages": [
        "pages/result/result",
        "pages/hot/hot"
      ]
    }
  ]
}
```

> **Java开发者注意点**：分包的概念类似于 Java Web 中的模块化打包（Maven Multi-Module 或 Spring Boot 的分层架构）：
> - **主包（main package）** = 核心模块，包含 TabBar 页面和公共资源
> - **分包（sub package）** = 功能模块，按业务领域拆分
> - 分包之间不能互相引用对方的组件（除非使用分包预下载或分包异步化）
>
> 目录结构对应关系：
> ```
> 主包 pages/          → com.example.app.core
> packageArticle/     → com.example.app.article
> packageUser/        → com.example.app.user
> packageSearch/      → com.example.app.search
> ```

#### 分包预下载

```json
// app.json —— 分包预下载配置
{
  "preloadRule": {
    "pages/index/index": {
      "network": "wifi",
      "packages": ["packageArticle"]
    },
    "pages/profile/profile": {
      "network": "all",
      "packages": ["packageUser"]
    }
  }
}
```

> **说明**：当用户进入首页时，在 WiFi 环境下预下载文章详情分包；进入个人中心时，预下载用户相关分包。这类似于 Web 应用中的 **资源预加载（Preload）** 策略。

### 13.2 预加载与懒加载策略

#### 图片懒加载

```xml
<!-- 使用 lazy-load 属性实现图片懒加载 -->
<image
  wx:for="{{articleList}}"
  wx:key="id"
  src="{{item.coverUrl}}"
  mode="aspectFill"
  lazy-load
  class="article-cover"
/>
```

#### 组件按需引入

```json
// 页面级别的组件按需引入（而非全局引入）
// pages/article/detail.json
{
  "usingComponents": {
    "rich-text-parser": "/components/rich-text-parser/rich-text-parser",
    "comment-list": "/components/comment-list/comment-list",
    "author-card": "/components/author-card/author-card"
  }
}
```

> **Java开发者注意点**：小程序的组件按需引入类似于 Spring Boot 的**条件装配**（`@ConditionalOnProperty`）。只有页面明确声明需要的组件才会被打包进该页面的代码中，减小包体积。

#### 数据预加载

```javascript
// 在列表页点击时预加载详情页数据
Page({
  data: {
    articles: []
  },

  // 方式一：使用 wx.navigateTo 的 events 参数
  onCardTap(e) {
    const articleId = e.detail.articleId

    // 预请求数据
    const detailPromise = http.get(`/api/articles/${articleId}`)

    wx.navigateTo({
      url: `/packageArticle/pages/detail/detail?id=${articleId}`,
      events: {
        // 详情页通过 eventChannel 获取预加载数据
        getData: (callback) => {
          detailPromise.then(res => callback(res.data))
        }
      }
    })
  }
})

// 详情页接收预加载数据
Page({
  onLoad(options) {
    const eventChannel = this.getOpenerEventChannel()

    // 尝试获取预加载数据
    eventChannel.on('getData', async (callback) => {
      const data = await callback
      if (data) {
        this.setData({ article: data, loading: false })
      }
    })

    // 如果没有预加载数据，则自行请求
    if (!this.data.article) {
      this.loadArticle(options.id)
    }
  }
})
```

### 13.3 getApp() 全局状态管理

`getApp()` 用于获取小程序全局实例，可以在不同页面间共享数据和方法。

```javascript
// app.js —— 全局状态管理
App({
  globalData: {
    userInfo: null,
    token: null,
    systemInfo: null
  },

  onLaunch() {
    // 获取系统信息
    const systemInfo = wx.getSystemInfoSync()
    this.globalData.systemInfo = systemInfo

    // 检查登录状态
    const token = wx.getStorageSync('token')
    if (token) {
      this.globalData.token = token
      this.globalData.userInfo = wx.getStorageSync('userInfo')
    }
  },

  // 全局方法：更新用户信息
  updateUserInfo(userInfo) {
    this.globalData.userInfo = userInfo
    wx.setStorageSync('userInfo', userInfo)
  },

  // 全局方法：检查是否登录
  isLoggedIn() {
    return !!this.globalData.token
  }
})
```

```javascript
// 在页面中使用全局状态
const app = getApp()

Page({
  onLoad() {
    if (app.isLoggedIn()) {
      this.setData({ userInfo: app.globalData.userInfo })
    }
  },

  onShow() {
    // 每次显示页面时同步最新状态
    if (app.globalData.userInfo) {
      this.setData({ userInfo: app.globalData.userInfo })
    }
  }
})
```

> **Java开发者注意点**：`getApp()` 的 `globalData` 类似于 Spring 中的 **Application Scope Bean**（`@Scope("application")`）。但需要注意：
> 1. **不要在 globalData 中存储大量数据**，它常驻内存，会影响小程序性能。
> 2. **页面获取 globalData 后不会自动更新**，需要手动同步。如果需要响应式全局状态，建议使用小程序的 `Behavior` 或第三方状态管理库（如 `mobx-miniprogram`）。
> 3. **globalData 在小程序冷启动时初始化，热启动时保持不变**，这与 Spring Boot 应用启动时初始化 Application Bean 的行为一致。

### 13.4 页面间数据传递方式对比

| 方式 | 适用场景 | 优点 | 缺点 | 类比 Java |
|------|---------|------|------|-----------|
| **URL 参数** | 传递简单参数（ID、关键词等） | 简单直接，支持刷新 | 数据量有限，仅支持字符串 | `HttpServletRequest.getParameter()` |
| **全局变量** (globalData) | 多页面共享的用户信息、配置 | 任何页面都可访问 | 不响应式，需手动同步 | `ServletContext.getAttribute()` |
| **本地缓存** (Storage) | 需要持久化的数据 | 小程序重启后仍在 | 同步读写会阻塞 | 数据库 / Redis |
| **EventChannel** | 页面间传递复杂数据或对象 | 支持双向通信，数据量大 | 仅适用于页面跳转场景 | `RedirectAttributes.addAttribute()` |

#### 各方式代码示例

**URL 参数传递：**

```javascript
// 发送方
wx.navigateTo({
  url: `/packageArticle/pages/detail/detail?id=${articleId}&from=home`
})

// 接收方
Page({
  onLoad(options) {
    const id = options.id          // "123"
    const from = options.from      // "home"
  }
})
```

**本地缓存传递：**

```javascript
// 发送方
wx.setStorageSync('tempArticle', articleData)
wx.navigateTo({ url: '/packageArticle/pages/detail/detail' })

// 接收方
Page({
  onLoad() {
    const article = wx.getStorageSync('tempArticle')
    // 使用完毕后清除
    wx.removeStorageSync('tempArticle')
  }
})
```

**EventChannel 传递（推荐用于复杂对象）：**

```javascript
// 发送方
wx.navigateTo({
  url: '/packageArticle/pages/detail/detail',
  success(res) {
    // 向目标页面发送数据
    res.eventChannel.emit('sendArticleData', {
      article: fullArticleData,
      fromPage: 'home'
    })
  }
})

// 接收方
Page({
  onLoad() {
    const eventChannel = this.getOpenerEventChannel()
    eventChannel.on('sendArticleData', (data) => {
      this.setData({
        article: data.article,
        fromPage: data.fromPage
      })
    })
  }
})
```

> **类比理解**：EventChannel 类似于 Java 中的 **Spring ApplicationEvent** 机制：
> - `eventChannel.emit()` = `applicationEventPublisher.publishEvent()`
> - `eventChannel.on()` = `@EventListener`
> - 它是页面间的点对点通信通道，比全局广播更精准。

### 13.5 性能优化清单

> **Java开发者注意点**：小程序的性能优化思路与 Java Web 应用的性能优化有很多共通之处，以下是核心优化策略对照：

| 优化策略 | 小程序实现 | Java Web 类比 |
|---------|-----------|--------------|
| 减少包体积 | 分包加载、按需引入组件 | Maven 依赖裁剪、Tree Shaking |
| 减少网络请求 | 数据预加载、请求合并 | GraphQL、批量查询接口 |
| 减少渲染压力 | `wx:if` vs `hidden` 合理使用、虚拟列表 | 服务端分页、懒加载 |
| 缓存策略 | `wx.setStorage` 缓存热点数据 | Redis 缓存、HTTP Cache |
| 图片优化 | 懒加载、CDN + WebP 格式、缩略图 | 图片压缩、CDN 加速 |
| 启动速度 | 分包预下载、首屏数据预请求 | Spring Boot 懒加载、预热 |

**setData 优化建议：**

```javascript
// ❌ 不好的做法：频繁调用 setData，每次传递大量数据
for (let i = 0; i < list.length; i++) {
  this.setData({ [`list[${i}].loaded`]: true })  // 触发 N 次渲染
}

// ✅ 好的做法：合并 setData 调用
const updates = {}
list.forEach((item, index) => {
  updates[`list[${index}].loaded`] = true
})
this.setData(updates)  // 只触发 1 次渲染

// ✅ 好的做法：只传递变化的数据，而非整个对象
// ❌ this.setData({ article: newArticle })  // 传递整个对象
// ✅ this.setData({ 'article.title': newTitle })  // 只传递变化的字段
```

> **关键原则**：`setData` 是小程序渲染层与逻辑层通信的桥梁，每次调用都会有序列化和跨线程传输的开销。这与 Java Web 中频繁的数据库查询或 RPC 调用类似 —— 应尽量减少调用次数和数据量。

---

## 本部分小结

通过第9-13章的学习，我们掌握了微信小程序开发中组件与 API 的核心知识：

| 章节 | 核心内容 | Java 类比 |
|------|---------|-----------|
| 第9章 基础组件 | view/scroll-view/swiper/rich-text 等内置组件 | Thymeleaf 标签库 / JSP 标签 |
| 第10章 自定义组件 | Component 构造器、properties、slot、组件通信 | Spring Bean / 自定义 Fragment |
| 第11章 网络请求 | wx.request 封装、拦截器、与 RESTful API 对接 | RestTemplate / Feign Client |
| 第12章 登录体系 | wx.login、JWT 鉴权、Token 刷新 | Spring Security OAuth2 |
| 第13章 性能优化 | 分包加载、全局状态、页面通信、setData 优化 | 模块化打包 / Application Scope |

在下一部分中，我们将进入实战项目阶段，综合运用以上知识构建一个完整的内容资讯类小程序。

---

---

