# 微信小程序开发入门教程

> **适用读者**：具有 Java Web 开发经验（Spring Boot / Spring MVC）的后端开发者
> **学习目标**：从零掌握微信小程序开发，能够独立完成内容资讯类小程序项目
> **教程特色**：Java 视角对比、完整实战项目、企业级开发规范

---

## 第6章 JavaScript逻辑层开发

JavaScript 是微信小程序逻辑层的核心语言。对于 Java 开发者来说，JavaScript 的语法学习并不困难，真正需要理解的是小程序的**运行模型**和**数据驱动机制**。本章将深入讲解小程序的页面生命周期、数据管理、事件处理和模块化开发。

### 6.1 Page 对象结构与生命周期函数

每个小程序页面都通过 `Page(Object)` 函数来注册，其中 `Object` 是一个包含页面初始数据、生命周期函数和事件处理函数的对象。这类似于 Spring MVC 中 `@Controller` 注解的类——它定义了请求的处理逻辑和数据的准备方式。

```javascript
// pages/user/user.js
Page({
  // ========== 1. 初始数据 ==========
  data: {
    userInfo: null,
    hasLogin: false,
    orderCount: 0
  },

  // ========== 2. 生命周期函数 ==========

  // 页面加载时触发（只执行一次）
  // 类比：Spring Controller 的 @PostConstruct 初始化方法
  onLoad(options) {
    console.log('页面加载，参数：', options)
    // options 是页面跳转时传递的参数
    // 例如从其他页面跳转到 /pages/user/user?id=123
    // 则 options = { id: '123' }
    const userId = options.id || ''
    if (userId) {
      this.fetchUserInfo(userId)
    }
  },

  // 页面初次渲染完成时触发（只执行一次）
  // 类比：页面 DOM 加载完成，可以安全操作视图
  onReady() {
    console.log('页面初次渲染完成')
    // 可以在这里获取节点信息（wx.createSelectorQuery）
  },

  // 页面显示时触发（每次进入页面都会触发）
  // 类比：Spring MVC 的 @RequestMapping 方法每次被调用
  onShow() {
    console.log('页面显示')
    // 适合在这里刷新数据，例如从其他页面返回后更新
    this.refreshData()
  },

  // 页面隐藏时触发（跳转到其他页面或小程序切到后台）
  // 类比：Activity 的 onPause
  onHide() {
    console.log('页面隐藏')
    // 适合在这里暂停定时器、保存临时数据等
  },

  // 页面卸载时触发（使用 wx.navigateBack 或 redirectTo）
  // 类比：Spring Bean 的 @PreDestroy 销毁方法
  onUnload() {
    console.log('页面卸载')
    // 适合在这里清理资源
  },

  // ========== 3. 页面事件处理函数 ==========

  // 下拉刷新
  onPullDownRefresh() {
    console.log('用户下拉刷新')
    this.refreshData().then(() => {
      wx.stopPullDownRefresh()
    })
  },

  // 上拉触底
  onReachBottom() {
    console.log('用户上拉触底')
    this.loadMore()
  },

  // 页面滚动
  onPageScroll(e) {
    // e.scrollTop 为当前滚动位置
  },

  // 用户分享
  onShareAppMessage() {
    return {
      title: '用户中心',
      path: '/pages/user/user?id=123'
    }
  },

  // ========== 4. 自定义方法 ==========

  fetchUserInfo(userId) {
    wx.request({
      url: 'https://api.example.com/user/' + userId,
      success: (res) => {
        this.setData({
          userInfo: res.data,
          hasLogin: true
        })
      }
    })
  },

  refreshData() {
    return new Promise((resolve) => {
      // 刷新数据逻辑
      resolve()
    })
  },

  loadMore() {
    // 加载更多逻辑
  }
})
```

> **类比理解：Page 生命周期与 Spring MVC 的对比**
>
> | 小程序生命周期 | 触发时机 | Spring MVC 类比 |
> |--------------|---------|----------------|
> | `onLoad(options)` | 页面首次加载 | `@PostConstruct` + `@PathVariable` 参数注入 |
> | `onReady()` | 页面首次渲染完成 | JSP/Thymeleaf 渲染完成 |
> | `onShow()` | 每次页面显示 | 每次 HTTP 请求到达 Controller 方法 |
> | `onHide()` | 页面隐藏 | 请求处理完成，线程释放 |
> | `onUnload()` | 页面销毁 | Servlet 销毁 / Bean 销毁 |
>
> **关键理解**：`onLoad` 只在页面首次创建时执行一次（类似构造函数），而 `onShow` 在每次进入页面时都会执行（类似方法调用）。这是初学者最容易混淆的地方。例如，从 A 页面跳转到 B 页面，再从 B 返回 A 时，A 页面的 `onShow` 会再次触发，但 `onLoad` 不会。

**页面跳转与参数传递：**

```javascript
// 页面 A 中跳转到页面 B，并传递参数
wx.navigateTo({
  url: '/pages/detail/detail?id=123&type=article'
})

// 页面 B 的 onLoad 中接收参数
Page({
  onLoad(options) {
    // options = { id: '123', type: 'article' }
    const id = options.id          // '123'（注意：所有参数都是字符串类型）
    const type = options.type      // 'article'
    // 如果需要数字类型，需要手动转换
    const numId = parseInt(id)     // 123
  }
})
```

> **Java 开发者注意点**
>
> 页面跳转传参类似于 Spring MVC 中的 URL 路径参数：
>
> ```java
> // Spring MVC 写法
> @GetMapping("/detail/{id}")
> public String detail(@PathVariable Long id, @RequestParam String type) {
>     // id 和 type 会自动进行类型转换
> }
> ```
>
> 但小程序中 `options` 的所有值都是**字符串类型**，需要手动转换类型，这一点与 Java 的自动类型转换不同，需要特别注意。

### 6.2 数据管理：data 对象与 setData() 方法

`data` 对象是页面的初始数据，用于 WXML 的数据绑定。`setData()` 是更新 `data` 并驱动视图更新的唯一方法。

```javascript
Page({
  data: {
    message: '初始消息',
    count: 0,
    user: {
      name: '张三',
      age: 25
    },
    items: ['苹果', '香蕉', '橙子'],
    isLoading: false
  },

  // ===== setData 基本用法 =====

  updateMessage() {
    // 更新简单属性
    this.setData({
      message: '更新后的消息'
    })
  },

  incrementCount() {
    // 注意：不能直接修改 data 中的值来触发视图更新
    // 错误写法：this.data.count++（数据会变，但视图不会更新）
    // 正确写法：使用 setData
    this.setData({
      count: this.data.count + 1
    })
  },

  updateNestedProperty() {
    // 更新对象中的某个属性（使用路径语法）
    this.setData({
      'user.name': '李四',
      'user.age': 30
    })
  },

  updateArrayItem() {
    // 更新数组中的某个元素（使用路径语法）
    this.setData({
      'items[0]': '草莓'
    })
    // items 变为 ['草莓', '香蕉', '橙子']
  },

  // ===== setData 高级用法 =====

  batchUpdate() {
    // 批量更新多个属性（一次 setData 调用）
    this.setData({
      message: '批量更新',
      count: 100,
      isLoading: true,
      'user.name': '王五'
    })
  },

  // ===== setData 性能优化 =====

  // 反模式：频繁调用 setData
  badUpdate() {
    // 不好的做法：循环中多次调用 setData
    for (let i = 0; i < 100; i++) {
      this.setData({ count: i })  // 触发100次视图更新！
    }
  },

  // 推荐模式：合并为一次 setData 调用
  goodUpdate() {
    // 好的做法：先计算好数据，一次性更新
    const newData = {}
    for (let i = 0; i < 100; i++) {
      newData['items[' + i + ']'] = 'item-' + i
    }
    this.setData(newData)  // 只触发1次视图更新
  },

  // ===== setData 与数据传递 =====

  handleInput(e) {
    // 从事件对象获取值
    const value = e.detail.value
    this.setData({
      message: value
    })
  }
})
```

> **Java 开发者注意点：setData 与 Vue 响应式系统的对比**
>
> 如果你了解 Vue.js，`setData()` 的行为与 Vue 的 `this.xxx = newValue` 非常相似。但有一个关键区别：
>
> | 特性 | Vue | 微信小程序 |
> |------|-----|-----------|
> | 数据更新方式 | `this.xxx = newValue` | `this.setData({ xxx: newValue })` |
> | 响应式原理 | Object.defineProperty / Proxy | setData + 差异比较 |
> | 批量更新 | 自动合并（nextTick） | 需要手动合并 |
> | 性能优化 | 虚拟 DOM | 差异比较（只更新变化的节点） |
>
> **setData 的核心规则**：
> 1. **不要直接修改 `this.data`**：直接修改 `this.data` 不会触发视图更新，必须使用 `setData()`。
> 2. **减少 setData 的调用频率**：每次 `setData` 都会触发视图层的 diff 计算，频繁调用会导致性能问题。
> 3. **减少 setData 传输的数据量**：只传递需要更新的字段，不要传递整个 `data` 对象。
> 4. **setData 是异步的**：如果需要在 setData 完成后执行操作，可以使用回调函数：
>    ```javascript
>    this.setData({ count: 1 }, function() {
>      // setData 完成后的回调
>      console.log('视图已更新')
>    })
>    ```

### 6.3 事件处理系统

小程序的事件处理与 HTML DOM 事件类似，但命名和机制有所不同。事件绑定以 `bind` 或 `catch` 开头，后跟事件类型。

**常用事件类型：**

| 事件名 | 说明 | 对应 HTML 事件 |
|--------|------|---------------|
| `tap` | 手指触摸后离开 | `click` |
| `longpress` | 手指触摸后超过 350ms 再离开 | — |
| `touchstart` | 手指触摸动作开始 | `touchstart` |
| `touchmove` | 手指触摸后移动 | `touchmove` |
| `touchend` | 手指触摸动作结束 | `touchend` |
| `input` | 输入框内容变化 | `input` |
| `confirm` | 输入框确认（回车） | — |
| `change` | 表单值改变 | `change` |

**bind 与 catch 的区别：**

```xml
<!-- pages/event/event.wxml -->

<!-- bindtap：允许事件冒泡（父节点也会收到事件） -->
<view class="outer" bindtap="onOuterTap">
  <view class="middle" bindtap="onMiddleTap">
    <view class="inner" bindtap="onInnerTap">
      点击我（事件会冒泡到 middle 和 outer）
    </view>
  </view>
</view>

<!-- catchtap：阻止事件冒泡（父节点不会收到事件） -->
<view class="outer" bindtap="onOuterTap">
  <view class="middle" catchtap="onMiddleTap">
    <view class="inner" bindtap="onInnerTap">
      点击我（事件不会冒泡到 outer，被 middle 拦截）
    </view>
  </view>
</view>

<!-- 事件传参：通过 data-* 属性传递 -->
<view class="item" wx:for="{{list}}" wx:key="id"
  bindtap="onItemTap"
  data-id="{{item.id}}"
  data-name="{{item.name}}"
  data-index="{{index}}"
>
  {{item.name}}
</view>

<!-- 表单事件 -->
<input bindinput="onInput" placeholder="请输入" />
<button bindtap="onSubmit" type="primary">提交</button>

<!-- 阻止事件冒泡的同时绑定事件 -->
<view catchtap="onCatchTap">
  这个点击事件不会冒泡
</view>
```

```javascript
// pages/event/event.js
Page({
  data: {
    list: [
      { id: 1, name: '项目A' },
      { id: 2, name: '项目B' },
      { id: 3, name: '项目C' }
    ]
  },

  // 事件冒泡示例
  onInnerTap(e) {
    console.log('inner 被点击')
  },
  onMiddleTap(e) {
    console.log('middle 被点击')
  },
  onOuterTap(e) {
    console.log('outer 被点击')
  },

  // 通过 dataset 获取传递的参数
  onItemTap(e) {
    // e.currentTarget.dataset 包含所有 data-* 属性
    const id = e.currentTarget.dataset.id       // '1'（字符串）
    const name = e.currentTarget.dataset.name    // '项目A'
    const index = e.currentTarget.dataset.index  // '0'（字符串）

    console.log('点击了：', name, '，索引：', index)
  },

  // 输入框事件
  onInput(e) {
    // e.detail.value 获取输入框的值
    const value = e.detail.value
    this.setData({ inputValue: value })
  },

  // 提交事件
  onSubmit() {
    const inputValue = this.data.inputValue
    console.log('提交的值：', inputValue)
  }
})
```

> **Java 开发者注意点**
>
> 1. **bind vs catch**：`bind` 相当于 Java Swing 中的普通事件监听（允许事件传递），`catch` 相当于消费了事件不再传递（类似 `e.consume()`）。
>
> 2. **事件传参方式**：小程序通过 `data-*` 属性传参，而不是像 HTML 那样可以在事件处理函数中直接传参。这与 Spring MVC 中通过 `@RequestParam` 接收参数的思想类似——参数都封装在请求对象中。
>
> 3. **dataset 的类型**：`e.currentTarget.dataset` 中的所有值都是**字符串类型**，如果需要数字或布尔值，需要手动转换：
>    ```javascript
>    const id = parseInt(e.currentTarget.dataset.id)   // 字符串转数字
>    const isActive = e.currentTarget.dataset.active === 'true'  // 字符串转布尔
>    ```

### 6.4 模块化开发

小程序支持 CommonJS 模块化规范，通过 `module.exports` 导出模块，通过 `require()` 引入模块。这与 Java 的包管理和类导入机制概念相通。

**定义工具模块（utils/request.js）：**

```javascript
// utils/request.js - 网络请求封装
const BASE_URL = 'https://api.example.com'

// 封装 GET 请求
function get(url, params = {}) {
  return new Promise((resolve, reject) => {
    wx.request({
      url: BASE_URL + url,
      method: 'GET',
      data: params,
      header: {
        'Content-Type': 'application/json',
        'Authorization': wx.getStorageSync('token') || ''
      },
      success(res) {
        if (res.statusCode === 200) {
          resolve(res.data)
        } else {
          reject(new Error('请求失败：' + res.statusCode))
        }
      },
      fail(err) {
        reject(err)
      }
    })
  })
}

// 封装 POST 请求
function post(url, data = {}) {
  return new Promise((resolve, reject) => {
    wx.request({
      url: BASE_URL + url,
      method: 'POST',
      data: data,
      header: {
        'Content-Type': 'application/json',
        'Authorization': wx.getStorageSync('token') || ''
      },
      success(res) {
        if (res.statusCode === 200 || res.statusCode === 201) {
          resolve(res.data)
        } else {
          reject(new Error('请求失败：' + res.statusCode))
        }
      },
      fail(err) {
        reject(err)
      }
    })
  })
}

// 导出模块
module.exports = {
  get,
  post,
  BASE_URL
}
```

**定义业务模块（services/user.js）：**

```javascript
// services/user.js - 用户相关业务逻辑
const { get, post } = require('../utils/request')

// 获取用户信息
function getUserInfo(userId) {
  return get('/api/user/' + userId)
}

// 更新用户信息
function updateUserInfo(userId, data) {
  return post('/api/user/' + userId + '/update', data)
}

// 登录
function login(phone, password) {
  return post('/api/auth/login', { phone, password })
}

module.exports = {
  getUserInfo,
  updateUserInfo,
  login
}
```

**在页面中使用模块：**

```javascript
// pages/user/user.js
const userService = require('../../services/user')
const { formatDate } = require('../../utils/util')

Page({
  data: {
    userInfo: null,
    loading: false
  },

  async onLoad(options) {
    try {
      this.setData({ loading: true })
      const userInfo = await userService.getUserInfo(options.id)
      this.setData({
        userInfo: {
          ...userInfo,
          createTime: formatDate(userInfo.createTime)
        },
        loading: false
      })
    } catch (error) {
      console.error('获取用户信息失败：', error)
      wx.showToast({ title: '加载失败', icon: 'none' })
      this.setData({ loading: false })
    }
  }
})
```

**定义常量模块（utils/constants.js）：**

```javascript
// utils/constants.js
module.exports = {
  // API 状态码
  HTTP_STATUS: {
    SUCCESS: 200,
    CREATED: 201,
    BAD_REQUEST: 400,
    UNAUTHORIZED: 401,
    FORBIDDEN: 403,
    NOT_FOUND: 404,
    SERVER_ERROR: 500
  },

  // 订单状态
  ORDER_STATUS: {
    PENDING: 'pending',
    PAID: 'paid',
    SHIPPED: 'shipped',
    COMPLETED: 'completed',
    CANCELLED: 'cancelled'
  },

  // 本地存储键名
  STORAGE_KEYS: {
    TOKEN: 'auth_token',
    USER_INFO: 'user_info',
    CART: 'shopping_cart'
  }
}
```

> **类比理解：模块化与 Java 包管理的对比**
>
> | JavaScript (CommonJS) | Java | 说明 |
> |-----------------------|------|------|
> | `module.exports = { ... }` | `public class` | 导出模块/定义类 |
> | `require('./utils/request')` | `import com.example.utils.Request` | 引入模块/导入类 |
> | `const { get, post } = require(...)` | `import static` | 解构导入/静态导入 |
> | 一个 `.js` 文件 | 一个 `.java` 文件 | 模块/编译单元 |
> | `utils/` 目录 | `com.example.utils` 包 | 目录/包命名空间 |
>
> **关键区别**：
> 1. JavaScript 的 `require` 是运行时加载，Java 的 `import` 是编译时确定。
> 2. JavaScript 使用相对路径（`./`、`../`），Java 使用全限定包名。
> 3. JavaScript 的模块粒度更灵活，一个文件可以导出多个函数/对象，而 Java 一个文件通常只有一个 public 类。

### 6.5 数据驱动：不能直接操作 DOM

这是小程序开发中最重要的理念转变。在传统的 Web 开发中（包括使用 jQuery 的时代），我们可以直接通过 JavaScript 操作 DOM 元素：

```javascript
// 传统 Web 开发 - 直接操作 DOM（jQuery）
$('#title').text('新标题')
$('#content').html('<p>新内容</p>')
$('#list').append('<li>新项目</li>')
document.getElementById('title').style.color = 'red'
```

但在小程序中，**以上所有 DOM 操作都是不允许的**。小程序没有提供 `document`、`window` 等 DOM API，所有的视图更新都必须通过 `setData()` 来驱动：

```javascript
// 小程序开发 - 数据驱动视图
Page({
  data: {
    title: '旧标题',
    content: '旧内容',
    list: ['项目1', '项目2'],
    titleColor: '#333333'
  },

  updateView() {
    // 通过 setData 更新数据，框架自动更新视图
    this.setData({
      title: '新标题',          // 对应 <text>{{title}}</text>
      content: '新内容',        // 对应 <text>{{content}}</text>
      titleColor: 'red'         // 对应 style="color: {{titleColor}}"
    })

    // 添加列表项
    this.setData({
      list: this.data.list.concat(['新项目'])
    })
  }
})
```

```xml
<!-- 对应的 WXML -->
<view>
  <text style="color: {{titleColor}}">{{title}}</text>
  <text>{{content}}</text>
  <view wx:for="{{list}}" wx:key="*this">
    <text>{{item}}</text>
  </view>
</view>
```

> **Java 开发者注意点**
>
> 如果你用过 Thymeleaf，可以这样理解：小程序的数据驱动模式就像是**Thymeleaf 的实时版本**。在 Thymeleaf 中，你通过修改 Model 中的数据来改变页面渲染结果，但每次都需要完整的 HTTP 请求-响应周期。而在小程序中，`setData()` 可以在不发起网络请求的情况下，实时更新页面上的数据绑定。
>
> | 操作方式 | 传统 Web (jQuery) | Thymeleaf | 微信小程序 |
> |---------|-------------------|-----------|-----------|
> | 更新文本 | `$('#el').text('new')` | 修改 Model 属性 | `this.setData({ key: 'new' })` |
> | 更新样式 | `$('#el').css('color', 'red')` | `th:style` | `this.setData({ color: 'red' })` |
> | 添加元素 | `$('#list').append(html)` | 循环渲染 | `this.setData({ list: [...] })` |
> | 显示/隐藏 | `$('#el').show()/hide()` | `th:if` | `this.setData({ show: true })` |
> | 更新时机 | 即时 | 下次请求 | 即时（异步） |

---

