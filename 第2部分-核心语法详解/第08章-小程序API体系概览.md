# 微信小程序开发入门教程

> **适用读者**：具有 Java Web 开发经验（Spring Boot / Spring MVC）的后端开发者
> **学习目标**：从零掌握微信小程序开发，能够独立完成内容资讯类小程序项目
> **教程特色**：Java 视角对比、完整实战项目、企业级开发规范

---

## 第8章 小程序API体系概览

微信小程序提供了丰富的原生 API，涵盖了网络请求、数据存储、媒体操作、位置服务、设备信息等各个方面。这些 API 都以 `wx.` 开头，采用回调或 Promise 的方式处理异步操作。对于 Java Web 开发者来说，这些 API 类似于 Spring Boot 中各种 `RestTemplate`、`RedisTemplate` 等工具类的客户端封装。

### 8.1 网络请求：wx.request

`wx.request` 是小程序中最常用的 API 之一，用于发起 HTTP 网络请求，功能上等同于浏览器端的 `XMLHttpRequest` / `fetch` 或 Java 后端的 `RestTemplate` / `HttpClient`。

**基本用法：**

```javascript
// GET 请求
Page({
  data: {
    productList: []
  },

  onLoad() {
    this.fetchProducts()
  },

  // 回调方式
  fetchProducts() {
    wx.request({
      url: 'https://api.example.com/products',
      method: 'GET',
      data: {
        page: 1,
        size: 10,
        category: 'electronics'
      },
      header: {
        'Content-Type': 'application/json',
        'Authorization': 'Bearer ' + wx.getStorageSync('token')
      },
      success(res) {
        console.log('请求成功：', res.statusCode, res.data)
        if (res.statusCode === 200) {
          this.setData({ productList: res.data.list })
        }
      },
      fail(err) {
        console.error('请求失败：', err)
        wx.showToast({ title: '网络错误', icon: 'none' })
      },
      complete() {
        // 无论成功失败都会执行
        wx.hideLoading()
      }
    })
  },

  // Promise 封装（推荐）
  async fetchProductDetail(id) {
    try {
      wx.showLoading({ title: '加载中' })
      const res = await new Promise((resolve, reject) => {
        wx.request({
          url: 'https://api.example.com/products/' + id,
          method: 'GET',
          success: resolve,
          fail: reject
        })
      })

      if (res.statusCode === 200) {
        this.setData({ product: res.data })
        return res.data
      } else {
        throw new Error('HTTP ' + res.statusCode)
      }
    } catch (error) {
      console.error('获取商品详情失败：', error)
      wx.showToast({ title: '加载失败', icon: 'none' })
    } finally {
      wx.hideLoading()
    }
  },

  // POST 请求
  submitOrder(orderData) {
    wx.request({
      url: 'https://api.example.com/orders',
      method: 'POST',
      data: orderData,
      header: {
        'Content-Type': 'application/json'
      },
      success(res) {
        if (res.statusCode === 201) {
          wx.showToast({ title: '下单成功' })
          wx.navigateTo({
            url: '/pages/order/detail?id=' + res.data.orderId
          })
        }
      },
      fail(err) {
        wx.showToast({ title: '下单失败', icon: 'none' })
      }
    })
  }
})
```

> **类比理解：wx.request 与 Ajax/Axios 的对比**
>
> | 特性 | Axios (Web) | wx.request (小程序) | RestTemplate (Java) |
> |------|-------------|---------------------|---------------------|
> | 请求方式 | `axios.get(url)` | `wx.request({ url, method })` | `restTemplate.getForObject(url)` |
> | 响应处理 | `.then(res => {})` | `success(res) {}` | 直接返回对象 |
> | 错误处理 | `.catch(err => {})` | `fail(err) {}` | `try-catch` / `ResponseEntity` |
> | 请求头 | `headers: { ... }` | `header: { ... }` | `HttpHeaders` |
> | 超时配置 | `timeout: 5000` | 在 `app.json` 中配置 | `requestFactory.setConnectTimeout()` |
> | 拦截器 | `axios.interceptors` | 需要自行封装 | `ClientHttpRequestInterceptor` |
> | 跨域 | 需要处理 CORS | 不存在跨域问题 | 不存在跨域问题 |
>
> **重要限制**：
> 1. **域名白名单**：小程序只能请求在微信公众平台配置的合法域名（HTTPS），开发阶段可以在开发者工具中关闭域名校验。
> 2. **无 Cookie**：小程序不支持 Cookie，需要通过 `header` 手动传递 Token。
> 3. **并发限制**：小程序同时最多发起 10 个 `wx.request` 请求。

### 8.2 数据缓存：wx.setStorage

小程序提供了本地缓存 API，功能上类似于浏览器的 `localStorage`，也类似于 Redis 的本地缓存模式。

**同步与异步 API：**

```javascript
Page({
  // ===== 异步 API（推荐） =====

  // 存储数据
  saveUserInfo() {
    wx.setStorage({
      key: 'userInfo',
      data: {
        id: '123',
        name: '张三',
        avatar: 'https://example.com/avatar.jpg'
      },
      success() {
        console.log('存储成功')
      },
      fail(err) {
        console.error('存储失败：', err)
      }
    })
  },

  // 读取数据
  getUserInfo() {
    wx.getStorage({
      key: 'userInfo',
      success(res) {
        console.log('读取成功：', res.data)
        // res.data = { id: '123', name: '张三', avatar: '...' }
      },
      fail() {
        console.log('数据不存在')
      }
    })
  },

  // 删除数据
  removeUserInfo() {
    wx.removeStorage({
      key: 'userInfo',
      success() {
        console.log('删除成功')
      }
    })
  },

  // 清空所有缓存
  clearAllCache() {
    wx.clearStorage({
      success() {
        console.log('缓存已清空')
      }
    })
  },

  // 获取缓存信息
  getStorageInfo() {
    wx.getStorageInfo({
      success(res) {
        console.log('当前缓存信息：', res.keys)       // 所有 key
        console.log('当前占用大小：', res.currentSize) // 字节
        console.log('限制大小：', res.limitSize)       // 字节（通常为 10485760 = 10MB）
      }
    })
  },

  // ===== 同步 API（简单场景可用） =====

  syncExample() {
    try {
      // 同步存储
      wx.setStorageSync('token', 'abc123')
      wx.setStorageSync('cartList', [
        { productId: 'p1', quantity: 2 },
        { productId: 'p2', quantity: 1 }
      ])

      // 同步读取
      const token = wx.getStorageSync('token')           // 'abc123'
      const cartList = wx.getStorageSync('cartList')     // [{...}, {...}]

      // 同步删除
      wx.removeStorageSync('token')

      // 同步获取信息
      const info = wx.getStorageInfoSync()
      console.log(info.keys, info.currentSize)
    } catch (e) {
      console.error('同步操作失败：', e)
    }
  }
})
```

> **类比理解：wx.setStorage 与 Redis/Session 的对比**
>
> | 特性 | wx.setStorage | Redis | HttpSession |
> |------|--------------|-------|-------------|
> | 存储位置 | 客户端（手机本地） | 服务端（内存/磁盘） | 服务端（内存） |
> | 容量上限 | 单个 key 最大 1MB，总上限 10MB | 取决于服务器配置 | 通常无硬性限制 |
> | 数据类型 | Object / String / Number 等 | String / Hash / List / Set 等 | Object |
> | 持久性 | 持久化存储（除非用户清理） | 可配置（RDB/AOF） | 会话结束即销毁 |
> | 安全性 | 用户可查看/修改（不安全） | 服务端控制（安全） | 服务端控制（安全） |
> | 适用场景 | Token、用户偏好设置、购物车 | 热点数据、缓存、分布式锁 | 用户登录状态 |
>
> **安全警告**：由于 `wx.setStorage` 的数据存储在客户端，用户可以通过工具查看和修改。因此，**绝对不能将敏感信息**（如密码、支付密钥等）存储在本地缓存中。Token 可以存储，但需要配合服务端验证机制（如 Token 过期、刷新机制）。这类似于浏览器中不应该在 `localStorage` 中存储敏感信息。

### 8.3 媒体操作 API

小程序提供了丰富的媒体操作 API，涵盖图片、音频、视频等方面。

**图片操作：**

```javascript
Page({
  data: {
    imageUrl: '',
    imageList: []
  },

  // 从相册选择图片
  chooseImage() {
    wx.chooseImage({
      count: 9,                          // 最多选择9张
      sizeType: ['original', 'compressed'], // 原图和压缩图
      sourceType: ['album', 'camera'],     // 相册和相机
      success: (res) => {
        const tempFilePaths = res.tempFilePaths
        console.log('选择的图片路径：', tempFilePaths)
        this.setData({
          imageList: tempFilePaths
        })
        // 上传图片到服务器
        this.uploadImages(tempFilePaths)
      }
    })
  },

  // 预览图片
  previewImage(e) {
    const current = e.currentTarget.dataset.src
    wx.previewImage({
      current: current,                    // 当前显示的图片链接
      urls: this.data.imageList            // 所有需要预览的图片链接列表
    })
  },

  // 上传图片到服务器
  uploadImages(filePathList) {
    filePathList.forEach((filePath) => {
      wx.uploadFile({
        url: 'https://api.example.com/upload',
        filePath: filePath,
        name: 'file',                      // 服务端接收的文件字段名
        header: {
          'Authorization': 'Bearer ' + wx.getStorageSync('token')
        },
        formData: {
          'type': 'avatar'                 // 额外的表单数据
        },
        success(res) {
          const data = JSON.parse(res.data)
          console.log('上传成功，URL：', data.url)
        },
        fail(err) {
          console.error('上传失败：', err)
        }
      })
    })
  },

  // 保存图片到相册
  saveImageToAlbum() {
    wx.downloadFile({
      url: 'https://example.com/image.jpg',
      success(res) {
        wx.saveImageToPhotosAlbum({
          filePath: res.tempFilePath,
          success() {
            wx.showToast({ title: '已保存到相册' })
          },
          fail() {
            wx.showToast({ title: '保存失败', icon: 'none' })
          }
        })
      }
    })
  }
})
```

**音频与视频操作：**

```javascript
// 音频录制与播放
Page({
  data: {
    isRecording: false,
    recordingTime: 0,
    audioPath: ''
  },

  // 开始录音
  startRecord() {
    this.setData({ isRecording: true })
    wx.startRecord({
      success: (res) => {
        this.setData({ audioPath: res.tempFilePath })
        console.log('录音完成：', res.tempFilePath)
      }
    })
  },

  // 停止录音
  stopRecord() {
    wx.stopRecord()
    this.setData({ isRecording: false })
  },

  // 播放录音
  playAudio() {
    const innerAudioContext = wx.createInnerAudioContext()
    innerAudioContext.src = this.data.audioPath
    innerAudioContext.play()
    innerAudioContext.onEnded(() => {
      console.log('播放结束')
    })
  }
})
```

### 8.4 位置服务：地图与定位

```javascript
Page({
  data: {
    latitude: 39.9042,
    longitude: 116.4074,
    markers: [],
    locationAddress: ''
  },

  // 获取当前位置
  getLocation() {
    wx.getLocation({
      type: 'gcj02',  // 国测局坐标（适用于微信地图）
      success: (res) => {
        this.setData({
          latitude: res.latitude,
          longitude: res.longitude
        })
        console.log('当前位置：', res.latitude, res.longitude)
        // 逆地理编码获取地址
        this.reverseGeocode(res.latitude, res.longitude)
      },
      fail(err) {
        console.error('获取位置失败：', err)
        // 提示用户开启定位权限
        wx.showModal({
          title: '提示',
          content: '需要获取您的位置信息，请授权',
          success(res) {
            if (res.confirm) {
              wx.openSetting()
            }
          }
        })
      }
    })
  },

  // 打开地图选择位置
  chooseLocation() {
    wx.chooseLocation({
      success: (res) => {
        this.setData({
          latitude: res.latitude,
          longitude: res.longitude,
          locationAddress: res.address + res.name
        })
        console.log('选择的位置：', res.name, res.address)
      }
    })
  },

  // 打开内置地图导航
  openMap() {
    wx.openLocation({
      latitude: this.data.latitude,
      longitude: this.data.longitude,
      scale: 18,
      name: '目标位置',
      address: this.data.locationAddress
    })
  },

  // 使用地图组件显示标记点
  onReady() {
    this.setData({
      markers: [
        {
          id: 1,
          latitude: 39.9042,
          longitude: 116.4074,
          title: '天安门',
          iconPath: '/images/marker.png',
          width: 30,
          height: 30,
          callout: {
            content: '天安门广场',
            display: 'ALWAYS',
            fontSize: 14,
            borderRadius: 4,
            padding: 8,
            bgColor: '#ffffff'
          }
        }
      ]
    })
  }
})
```

```xml
<!-- 地图组件使用 -->
<map
  latitude="{{latitude}}"
  longitude="{{longitude}}"
  markers="{{markers}}"
  scale="14"
  show-location
  style="width: 750rpx; height: 600rpx;"
/>
```

### 8.5 设备信息获取

```javascript
Page({
  data: {
    systemInfo: {},
    networkType: '',
    batteryLevel: 0,
    isCharging: false,
    screenBrightness: 0
  },

  onLoad() {
    this.getSystemInfo()
    this.getNetworkType()
    this.getBatteryInfo()
  },

  // 获取系统信息
  getSystemInfo() {
    const systemInfo = wx.getSystemInfoSync()
    this.setData({ systemInfo })
    console.log('设备品牌：', systemInfo.brand)           // 'iPhone' / 'Huawei' 等
    console.log('设备型号：', systemInfo.model)            // 'iPhone 14 Pro'
    console.log('系统版本：', systemInfo.system)           // 'iOS 16.0'
    console.log('微信版本：', systemInfo.version)          // '8.0.30'
    console.log('屏幕宽度：', systemInfo.screenWidth)      // 375 (px)
    console.log('屏幕高度：', systemInfo.screenHeight)     // 812 (px)
    console.log('像素比：', systemInfo.pixelRatio)         // 3
    console.log('平台：', systemInfo.platform)             // 'ios' / 'android'
    console.log('SDK版本：', systemInfo.SDKVersion)        // '3.0.0'
  },

  // 获取网络类型
  getNetworkType() {
    wx.getNetworkType({
      success: (res) => {
        this.setData({ networkType: res.networkType })
        // 'wifi' / '2g' / '3g' / '4g' / '5g' / 'unknown' / 'none'
        console.log('当前网络类型：', res.networkType)
      }
    })
  },

  // 获取电池信息
  getBatteryInfo() {
    wx.getBatteryInfo({
      success: (res) => {
        this.setData({
          batteryLevel: res.level,       // 电量百分比 0-100
          isCharging: res.isCharging      // 是否正在充电
        })
      }
    })
  },

  // 监听网络状态变化
  watchNetworkChange() {
    wx.onNetworkStatusChange((res) => {
      console.log('网络状态变化：', res.isConnected, res.networkType)
      if (!res.isConnected) {
        wx.showToast({ title: '网络已断开', icon: 'none' })
      } else {
        wx.showToast({ title: '网络已恢复：' + res.networkType })
      }
    })
  },

  // 设置屏幕亮度
  setScreenBrightness(value) {
    wx.setScreenBrightness({
      value: value  // 0-1 之间
    })
  },

  // 震动反馈
  vibrate() {
    wx.vibrateShort({ type: 'medium' })   // 短震动
    // wx.vibrateLong()                    // 长震动
  }
})
```

> **Java 开发者注意点**
>
> 1. **API 设计模式**：小程序的 API 遵循统一的命名规范 `wx.动词名词()`（如 `wx.getLocation`、`wx.setStorage`），这与 Java 中 `对象.方法名()` 的调用方式一致。
>
> 2. **异步处理**：大部分 API 都是异步的，支持回调函数和 Promise 两种方式。推荐使用 `async/await` 语法来处理异步操作，代码可读性更好：
>    ```javascript
>    // 回调方式（传统）
>    wx.request({ url, success(res) { ... }, fail(err) { ... } })
>
>    // Promise + async/await（推荐）
>    const res = await new Promise((resolve, reject) => {
>      wx.request({ url, success: resolve, fail: reject })
>    })
>    ```
>
> 3. **权限管理**：部分 API（如定位、相册、录音等）需要用户授权。如果用户拒绝授权，需要引导用户到设置页面手动开启。这类似于 Android 的运行时权限机制。
>
> 4. **API 限制**：部分 API 有调用频率限制，例如 `wx.login` 每天有调用次数限制。开发时需要关注官方文档中的调用限制说明。

### 8.6 API 速查表

| 分类 | API | 说明 |
|------|-----|------|
| **网络** | `wx.request` | 发起 HTTP 请求 |
| | `wx.uploadFile` | 上传文件 |
| | `wx.downloadFile` | 下载文件 |
| | `wx.connectSocket` | 创建 WebSocket 连接 |
| **数据缓存** | `wx.setStorage` / `wx.setStorageSync` | 存储数据 |
| | `wx.getStorage` / `wx.getStorageSync` | 读取数据 |
| | `wx.removeStorage` / `wx.removeStorageSync` | 删除数据 |
| | `wx.clearStorage` / `wx.clearStorageSync` | 清空数据 |
| **界面** | `wx.showToast` | 显示消息提示框 |
| | `wx.showLoading` | 显示加载提示框 |
| | `wx.showModal` | 显示模态对话框 |
| | `wx.showActionSheet` | 显示操作菜单 |
| | `wx.navigateTo` | 保留当前页，跳转到新页面 |
| | `wx.redirectTo` | 关闭当前页，跳转到新页面 |
| | `wx.navigateBack` | 关闭当前页，返回上一页 |
| | `wx.switchTab` | 跳转到 Tab 页 |
| **媒体** | `wx.chooseImage` | 从相册/相机选择图片 |
| | `wx.previewImage` | 预览图片 |
| | `wx.saveImageToPhotosAlbum` | 保存图片到相册 |
| | `wx.startRecord` / `wx.stopRecord` | 录音 |
| | `wx.createInnerAudioContext` | 创建音频上下文 |
| **位置** | `wx.getLocation` | 获取当前位置 |
| | `wx.chooseLocation` | 打开地图选择位置 |
| | `wx.openLocation` | 打开内置地图 |
| **设备** | `wx.getSystemInfo` | 获取系统信息 |
| | `wx.getNetworkType` | 获取网络类型 |
| | `wx.getBatteryInfo` | 获取电池信息 |
| | `wx.vibrateShort` / `wx.vibrateLong` | 震动反馈 |
| | `wx.setScreenBrightness` | 设置屏幕亮度 |
| | `wx.scanCode` | 扫描二维码 |
| | `wx.makePhoneCall` | 拨打电话 |
| **登录** | `wx.login` | 获取登录凭证 code |
| | `wx.getUserProfile` | 获取用户信息（已废弃，使用头像昵称填写能力） |
| | `wx.checkSession` | 检查登录态是否过期 |
| **支付** | `wx.requestPayment` | 发起微信支付 |

---

> **本部分小结**
>
> 通过第4-8章的学习，我们全面掌握了微信小程序的核心语法体系：
>
> - **WXML**（第4章）：小程序的视图层语言，通过数据绑定和指令实现动态页面渲染
> - **WXSS**（第5章）：小程序的样式语言，在 CSS 基础上新增了 `rpx` 响应式单位
> - **JavaScript**（第6章）：小程序的逻辑层语言，通过 Page 对象管理页面生命周期和数据
> - **JSON**（第7章）：小程序的配置体系，从全局到页面级别的分层配置
> - **API**（第8章）：小程序的能力接口，涵盖网络、存储、媒体、位置、设备等
>
> 作为 Java Web 开发者，你已经具备了良好的编程基础和架构思维。将这些知识与小程序的开发模式进行类比映射，可以大大加速学习过程。在接下来的第三部分中，我们将进入实战环节，通过构建一个完整的小程序项目来巩固所学知识。

---

