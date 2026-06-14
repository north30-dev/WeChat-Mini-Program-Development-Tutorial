# 微信小程序开发入门教程

> **适用读者**：具有 Java Web 开发经验（Spring Boot / Spring MVC）的后端开发者
> **学习目标**：从零掌握微信小程序开发，能够独立完成内容资讯类小程序项目
> **教程特色**：Java 视角对比、完整实战项目、企业级开发规范

---

## 附录B：小程序 API 速查表

### B.1 网络请求 API

#### wx.request -- 发起网络请求

```javascript
wx.request({
  url: 'https://api.example.com/users',  // 请求地址（必须 HTTPS）
  method: 'GET',                         // 请求方法：GET/POST/PUT/DELETE/HEAD/OPTIONS/PATCH
  data: {                                // 请求参数
    page: 1,
    size: 10
  },
  header: {                              // 请求头
    'Content-Type': 'application/json',
    'Authorization': 'Bearer ' + token
  },
  dataType: 'json',                      // 返回数据格式：json（默认）
  responseType: 'text',                  // 响应类型：text（默认）/ arraybuffer
  timeout: 60000,                        // 超时时间（毫秒）
  enableHttpDNS: false,                  // 是否启用 HTTPDNS
  enableQuic: false,                     // 是否开启 QUIC 协议
  success(res) {
    console.log(res.statusCode);  // HTTP 状态码
    console.log(res.data);        // 返回数据
    console.log(res.header);      // 响应头
    console.log(res.cookies);     // 响应 cookies
  },
  fail(err) {
    console.error('请求失败', err);
  },
  complete() {
    // 无论成功失败都执行
  }
});
```

> **注意：** 需要在小程序管理后台配置域名白名单（request 合法域名）。

#### wx.uploadFile -- 上传文件

```javascript
wx.uploadFile({
  url: 'https://api.example.com/upload',
  filePath: tempFilePath,          // 本地文件路径
  name: 'file',                    // 文件对应的 key
  formData: {                      // 额外的表单数据
    'user': 'test'
  },
  header: {
    'Authorization': 'Bearer ' + token
  },
  success(res) {
    const data = JSON.parse(res.data);
    console.log(data.url);  // 服务器返回的文件地址
  }
});
```

#### wx.downloadFile -- 下载文件

```javascript
wx.downloadFile({
  url: 'https://example.com/file.pdf',
  header: {},
  success(res) {
    if (res.statusCode === 200) {
      console.log(res.tempFilePath);  // 临时文件路径
      // 打开文档
      wx.openDocument({
        filePath: res.tempFilePath,
        fileType: 'pdf',
        success() { console.log('打开成功'); }
      });
    }
  }
});
```

#### 网络状态监听

```javascript
wx.onNetworkStatusChange(function(res) {
  console.log(res.isConnected);  // 是否有网络连接
  console.log(res.networkType);  // 网络类型：wifi/2g/3g/4g/unknown
});

wx.getNetworkType({
  success(res) {
    console.log(res.networkType);
  }
});
```

---

### B.2 数据缓存 API

> 小程序缓存上限为 **10MB**，数据以 key-value 形式存储。

#### 同步 API（推荐，代码更简洁）

```javascript
// ========= 存储 =========
try {
  wx.setStorageSync('key', 'value');           // 存储字符串
  wx.setStorageSync('userInfo', { name: 'Tom' }); // 存储对象（自动序列化）
  wx.setStorageSync('token', 'abc123');
} catch (e) {
  console.error('存储失败', e);
}

// ========= 读取 =========
const value = wx.getStorageSync('key');         // 读取字符串
const userInfo = wx.getStorageSync('userInfo'); // 读取对象（自动反序列化）
const token = wx.getStorageSync('token');

// ========= 删除 =========
wx.removeStorageSync('key');     // 删除指定 key

// ========= 清空 =========
wx.clearStorageSync();            // 清空所有缓存

// ========= 获取缓存信息 =========
const res = wx.getStorageInfoSync();
console.log(res.keys);       // 所有 key 数组
console.log(res.currentSize); // 当前占用空间（字节）
console.log(res.limitSize);   // 限制空间（字节，通常为 10485760 = 10MB）
```

#### 异步 API

```javascript
// 存储
wx.setStorage({
  key: 'key',
  data: 'value',
  encrypt: true,  // 是否加密存储（基础库 2.21.3+）
  success() { console.log('存储成功'); },
  fail(err) { console.error('存储失败', err); }
});

// 读取
wx.getStorage({
  key: 'key',
  success(res) {
    console.log(res.data);  // 读取到的数据
  }
});

// 删除
wx.removeStorage({
  key: 'key',
  success() { console.log('删除成功'); }
});

// 清空
wx.clearStorage({
  success() { console.log('清空成功'); }
});

// 获取缓存信息
wx.getStorageInfo({
  success(res) {
    console.log(res.keys);
    console.log(res.currentSize);
    console.log(res.limitSize);
  }
});
```

> **最佳实践：**
> - 敏感数据不要明文存储，建议加密
> - 存储前检查数据大小，避免超出 10MB 限制
> - 读取时做好异常处理（key 不存在返回空字符串）

---

### B.3 界面交互 API

#### wx.showToast -- 消息提示框

```javascript
wx.showToast({
  title: '操作成功',       // 提示文字（最多两行，约14-16个汉字）
  icon: 'success',         // 图标类型：success / error / loading / none
  image: '/images/custom.png', // 自定义图标（优先级高于 icon）
  duration: 1500,          // 显示时长（毫秒），默认 1500
  mask: false,             // 是否显示透明蒙层，防止触摸穿透
  success() {},
  complete() {}
});
```

#### wx.showModal -- 模态对话框

```javascript
wx.showModal({
  title: '提示',            // 标题
  content: '确定要删除吗？', // 内容
  showCancel: true,         // 是否显示取消按钮（默认 true）
  cancelText: '取消',       // 取消按钮文字（最多 4 个字）
  confirmText: '确定',      // 确认按钮文字（最多 4 个字）
  cancelColor: '#000000',   // 取消按钮颜色
  confirmColor: '#576B95',  // 确认按钮颜色
  editable: false,          // 是否显示输入框（基础库 2.17.1+）
  placeholderText: '',      // 输入框占位符
  success(res) {
    if (res.confirm) {
      console.log('用户点击确定');
    } else if (res.cancel) {
      console.log('用户点击取消');
    }
  }
});
```

#### wx.showLoading -- 加载提示

```javascript
// 显示
wx.showLoading({
  title: '加载中...',
  mask: true  // 防止触摸穿透（推荐开启）
});

// 隐藏
wx.hideLoading();
```

> **注意：** `wx.showLoading` 和 `wx.showToast` 不能同时显示。

#### wx.showActionSheet -- 操作菜单

```javascript
wx.showActionSheet({
  itemList: ['拍照', '从相册选择', '取消'],
  itemColor: '#333333',
  success(res) {
    console.log(res.tapIndex);  // 用户点击的按钮序号（从 0 开始）
    if (res.tapIndex === 0) {
      // 拍照
    } else if (res.tapIndex === 1) {
      // 从相册选择
    }
  },
  fail(res) {
    console.log(res.errMsg);  // 包含 "cancel" 表示用户点击了取消
  }
});
```

#### 页面导航 API

```javascript
// ========= 保留当前页面，跳转到新页面（可返回） =========
wx.navigateTo({
  url: '/pages/detail/detail?id=123',
  events: {
    // 接收被打开页面通过 EventChannel 传回的数据
    acceptDataFromOpenedPage(data) {
      console.log(data);
    }
  },
  success(res) {
    // 通过 EventChannel 向被打开页面传送数据
    res.eventChannel.emit('acceptDataFromOpenerPage', { id: 123 });
  }
});

// ========= 关闭当前页面，跳转到新页面（不可返回） =========
wx.redirectTo({
  url: '/pages/login/login'
});

// ========= 关闭所有页面，打开到新页面 =========
wx.reLaunch({
  url: '/pages/index/index'
});

// ========= 跳转到 tabBar 页面（不能带参数） =========
wx.switchTab({
  url: '/pages/home/home'
});

// ========= 返回上一页或多级页面 =========
wx.navigateBack({
  delta: 1,  // 返回的层数，默认 1
  success() {
    // 可通过 getCurrentPages() 获取页面栈
    const pages = getCurrentPages();
    const prevPage = pages[pages.length - 2]; // 上一个页面
    prevPage.setData({ refreshed: true });     // 直接修改上一页数据
  }
});

// ========= 页面栈（最多 10 层） =========
const pages = getCurrentPages();
console.log(pages.length);       // 当前页面栈层数
console.log(pages[pages.length - 1].route); // 当前页面路径
```

#### 导航栏与标题栏

```javascript
// 动态设置导航栏标题
wx.setNavigationBarTitle({
  title: '页面标题'
});

// 动态设置导航栏颜色
wx.setNavigationBarColor({
  frontColor: '#ffffff',   // 前景颜色（仅 #ffffff 和 #000000）
  backgroundColor: '#ff0000' // 背景颜色
});

// 显示/隐藏加载动画（标题栏右侧）
wx.showNavigationBarLoading();
wx.hideNavigationBarLoading();
```

---

### B.4 媒体 API

#### 图片相关

```javascript
// ========= 选择图片 =========
wx.chooseImage({
  count: 9,                          // 最多可选图片数量（默认 9）
  sizeType: ['original', 'compressed'], // 原图 / 压缩图
  sourceType: ['album', 'camera'],     // 相册 / 相机
  success(res) {
    const tempFilePaths = res.tempFilePaths;  // 图片临时路径数组
    const tempFiles = res.tempFiles;          // 包含 size 和 path 的数组
  }
});

// ========= 预览图片 =========
wx.previewImage({
  current: 'https://example.com/2.jpg',  // 当前显示图片的链接
  urls: [                                 // 所有需要预览的图片链接列表
    'https://example.com/1.jpg',
    'https://example.com/2.jpg',
    'https://example.com/3.jpg'
  ],
  indicator: 'default',   // 图片指示器样式：default / number / none
  loop: false             // 是否可循环预览
});

// ========= 获取图片信息 =========
wx.getImageInfo({
  src: '/images/photo.jpg',
  success(res) {
    console.log(res.width);       // 图片宽度（px）
    console.log(res.height);      // 图片高度（px）
    console.log(res.path);        // 图片本地路径
    console.log(res.orientation); // 图片方向
    console.log(res.type);        // 图片格式
  }
});

// ========= 压缩图片 =========
wx.compressImage({
  src: tempFilePath,
  quality: 80,   // 压缩质量 0-100
  success(res) {
    console.log(res.tempFilePath); // 压缩后的临时路径
  }
});

// ========= 保存图片到相册 =========
wx.saveImageToPhotosAlbum({
  filePath: tempFilePath,
  success() {
    wx.showToast({ title: '保存成功' });
  }
});
```

#### 视频相关

```javascript
// ========= 选择视频 =========
wx.chooseVideo({
  sourceType: ['album', 'camera'],
  compressed: true,          // 是否压缩
  maxDuration: 60,           // 拍摄视频最长时长（秒）
  camera: 'back',            // 默认拉起的前置/后置摄像头
  success(res) {
    console.log(res.tempFilePath);  // 视频临时路径
    console.log(res.duration);      // 视频时长（秒）
    console.log(res.size);          // 视频大小（字节）
    console.log(res.height);        // 视频高度
    console.log(res.width);         // 视频宽度
  }
});

// ========= 视频缩略图 =========
wx.getVideoInfo({
  src: videoPath,
  success(res) {
    console.log(res.width, res.height);
    console.log(res.duration);
    console.log(res.fps);
    console.log(res.size);
  }
});
```

#### 音频相关

```javascript
// ========= 获取全局唯一的音频管理器 =========
const audioManager = wx.getBackgroundAudioManager();

audioManager.title = '歌曲名';
audioManager.epname = '专辑名';
audioManager.singer = '歌手';
audioManager.coverImgUrl = '封面URL';
audioManager.src = '音频URL';  // 设置 src 后自动播放

audioManager.onPlay(() => { console.log('开始播放'); });
audioManager.onPause(() => { console.log('暂停播放'); });
audioManager.onStop(() => { console.log('停止播放'); });
audioManager.onEnded(() => { console.log('播放结束'); });

// ========= 录音 =========
const recorderManager = wx.getRecorderManager();

recorderManager.onStart(() => { console.log('开始录音'); });
recorderManager.onStop((res) => {
  console.log(res.tempFilePath);  // 录音临时路径
  console.log(res.duration);      // 录音时长（毫秒）
  console.log(res.fileSize);      // 文件大小（字节）
});

recorderManager.start({
  duration: 60000,     // 最长录音时间（毫秒）
  format: 'mp3',       // 音频格式：mp3 / aac
});
```

---

### B.5 位置 API

```javascript
// ========= 获取当前位置 =========
wx.getLocation({
  type: 'gcj02',   // 坐标类型：wgs84 / gcj02（国测局坐标，推荐）
  altitude: true,  // 获取高度信息
  success(res) {
    console.log(res.latitude);    // 纬度
    console.log(res.longitude);   // 经度
    console.log(res.speed);       // 速度
    console.log(res.accuracy);    // 位置的精确度
    console.log(res.altitude);    // 高度
  }
});

// ========= 打开地图查看位置 =========
wx.openLocation({
  latitude: 39.9042,
  longitude: 116.4074,
  scale: 18,              // 缩放级别 5-18
  name: '天安门',
  address: '北京市东城区'
});

// ========= 使用地图选择位置 =========
wx.chooseLocation({
  latitude: 39.9042,
  longitude: 116.4074,
  success(res) {
    console.log(res.name);       // 位置名称
    console.log(res.address);    // 详细地址
    console.log(res.latitude);   // 纬度
    console.log(res.longitude);  // 经度
  }
});

// ========= 持续监听位置变化 =========
wx.startLocationUpdate({
  success() {
    console.log('开始监听位置');
  }
});

wx.onLocationChange(function(res) {
  console.log(res.latitude, res.longitude);
});

// 停止监听
wx.stopLocationUpdate();

// ========= 使用内置地图组件 =========
// <map latitude="{{latitude}}" longitude="{{longitude}}" markers="{{markers}}" />
```

---

### B.6 用户授权 API

```javascript
// ========= 登录（获取 code） =========
wx.login({
  success(res) {
    if (res.code) {
      // 将 code 发送到后端，后端调用微信接口换取 openid 和 session_key
      wx.request({
        url: 'https://api.example.com/login',
        data: { code: res.code },
        success(loginRes) {
          const token = loginRes.data.token;
          wx.setStorageSync('token', token);
        }
      });
    }
  }
});

// ========= 获取用户信息（需要用户主动触发） =========
// 注意：wx.getUserProfile 已废弃，推荐使用头像昵称填写能力
// 新版方案：使用 button 组件的 open-type="chooseAvatar" 和 input 的 type="nickname"

// 获取头像
<button open-type="chooseAvatar" bindchooseavatar="onChooseAvatar">
  获取头像
</button>

Page({
  onChooseAvatar(e) {
    console.log(e.detail.avatarUrl);  // 临时头像路径
    // 需要上传到自己的服务器
  }
});

// 获取昵称
<input type="nickname" bindblur="onNicknameBlur" />

// ========= 获取用户授权设置 =========
wx.getSetting({
  success(res) {
    console.log(res.authSetting);
    // res.authSetting['scope.userLocation']  → 是否授权了位置
    // res.authSetting['scope.camera']         → 是否授权了相机
    // true = 已授权, false = 已拒绝, undefined = 未询问
  }
});

// ========= 提前发起授权请求 =========
wx.authorize({
  scope: 'scope.userLocation',
  success() {
    // 用户同意授权
    wx.getLocation();
  },
  fail() {
    // 用户拒绝授权，引导到设置页手动开启
    wx.showModal({
      title: '提示',
      content: '需要获取您的位置信息，请在设置中开启',
      success(res) {
        if (res.confirm) {
          wx.openSetting();
        }
      }
    });
  }
});

// ========= 打开设置页 =========
wx.openSetting({
  success(res) {
    console.log(res.authSetting);
  }
});
```

#### 常用 scope 列表

| scope | 说明 | 对应 API |
|-------|------|----------|
| `scope.userInfo` | 用户信息 | `wx.getUserInfo`（已废弃） |
| `scope.userLocation` | 地理位置 | `wx.getLocation`, `wx.chooseLocation` |
| `scope.userLocationBackground` | 后台定位 | `wx.startLocationUpdateBackground` |
| `scope.record` | 录音功能 | `RecorderManager.start` |
| `scope.camera` | 摄像头 | `<camera>` 组件 |
| `scope.writePhotosAlbum` | 保存到相册 | `wx.saveImageToPhotosAlbum` |
| `scope.album` | 选择相册 | `wx.chooseImage`（sourceType 含 album） |

---

### B.7 设备信息 API

```javascript
// ========= 获取系统信息 =========
wx.getSystemInfo({
  success(res) {
    console.log(res.brand);           // 设备品牌
    console.log(res.model);           // 设备型号
    console.log(res.system);          // 操作系统及版本
    console.log(res.platform);        // 客户端平台
    console.log(res.pixelRatio);      // 设备像素比
    console.log(res.screenWidth);     // 屏幕宽度（px）
    console.log(res.screenHeight);    // 屏幕高度（px）
    console.log(res.windowWidth);     // 可使用窗口宽度（px）
    console.log(res.windowHeight);    // 可使用窗口高度（px）
    console.log(res.statusBarHeight); // 状态栏高度（px）
    console.log(res.safeArea);        // 安全区域
    console.log(res.safeArea.top);    // 安全区域上边界
    console.log(res.safeArea.bottom); // 安全区域下边界
    console.log(res.SDKVersion);      // 基础库版本
  }
});

// 同步方式（推荐）
const systemInfo = wx.getSystemInfoSync();
console.log(systemInfo.platform);

// ========= 获取基础库版本 =========
const accountInfo = wx.getAccountInfoSync();
console.log(accountInfo.miniProgram.version);     // 当前版本号
console.log(accountInfo.miniProgram.envVersion);  // 环境：develop/trial/release

// ========= 获取设备网络状态 =========
wx.getNetworkType({
  success(res) {
    console.log(res.networkType);  // wifi / 2g / 3g / 4g / unknown / none
  }
});

// ========= 监听网络状态变化 =========
wx.onNetworkStatusChange(function(res) {
  console.log(res.isConnected);
  console.log(res.networkType);
});

// ========= 设置屏幕亮度 =========
wx.setScreenBrightness({
  value: 0.5  // 亮度值 0-1
});

// ========= 获取屏幕亮度 =========
wx.getScreenBrightness({
  success(res) {
    console.log(res.value);  // 亮度值 0-1
  }
});

// ========= 震动反馈 =========
wx.vibrateShort({ type: 'medium' });  // 短震动
wx.vibrateLong();                       // 长震动

// ========= 剪贴板 =========
wx.setClipboardData({
  data: '要复制的内容',
  success() {
    wx.showToast({ title: '已复制' });
  }
});

wx.getClipboardData({
  success(res) {
    console.log(res.data);  // 剪贴板内容
  }
});

// ========= 拨打电话 =========
wx.makePhoneCall({
  phoneNumber: '10086',
  success() {},
  fail() {}
});

// ========= 扫码 =========
wx.scanCode({
  onlyFromCamera: false,  // 是否只允许相机扫码（不允许相册）
  scanType: ['qrCode', 'barCode'],  // 扫码类型
  success(res) {
    console.log(res.result);     // 扫码结果
    console.log(res.scanType);   // 扫码类型
    console.log(res.charSet);    // 字符集
    console.log(res.path);       // 二维码路径
  }
});
```

---

