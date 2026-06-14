# 微信小程序开发入门教程

> **适用读者**：具有 Java Web 开发经验（Spring Boot / Spring MVC）的后端开发者
> **学习目标**：从零掌握微信小程序开发，能够独立完成内容资讯类小程序项目
> **教程特色**：Java 视角对比、完整实战项目、企业级开发规范

---

## 第7章 JSON配置体系

JSON（JavaScript Object Notation）在小程序开发中扮演着重要的配置角色。小程序通过 JSON 文件来进行全局配置、页面配置、项目配置等。对于 Java Web 开发者来说，JSON 配置文件类似于 Spring Boot 中的 `application.yml` 或 `application.properties`。

### 7.1 app.json 全局配置

`app.json` 是小程序的全局配置文件，位于项目根目录，是整个小程序的"总控配置"。

```json
{
  "pages": [
    "pages/index/index",
    "pages/category/category",
    "pages/cart/cart",
    "pages/user/user",
    "pages/product/detail",
    "pages/order/order",
    "pages/search/search"
  ],

  "window": {
    "navigationBarBackgroundColor": "#ffffff",
    "navigationBarTitleText": "我的商城",
    "navigationBarTextStyle": "black",
    "backgroundColor": "#f5f5f5",
    "backgroundTextStyle": "dark",
    "enablePullDownRefresh": true
  },

  "tabBar": {
    "color": "#999999",
    "selectedColor": "#07c160",
    "backgroundColor": "#ffffff",
    "borderStyle": "black",
    "list": [
      {
        "pagePath": "pages/index/index",
        "text": "首页",
        "iconPath": "images/tab/home.png",
        "selectedIconPath": "images/tab/home-active.png"
      },
      {
        "pagePath": "pages/category/category",
        "text": "分类",
        "iconPath": "images/tab/category.png",
        "selectedIconPath": "images/tab/category-active.png"
      },
      {
        "pagePath": "pages/cart/cart",
        "text": "购物车",
        "iconPath": "images/tab/cart.png",
        "selectedIconPath": "images/tab/cart-active.png"
      },
      {
        "pagePath": "pages/user/user",
        "text": "我的",
        "iconPath": "images/tab/user.png",
        "selectedIconPath": "images/tab/user-active.png"
      }
    ]
  },

  "networkTimeout": {
    "request": 10000,
    "downloadFile": 30000,
    "uploadFile": 30000,
    "connectSocket": 5000
  },

  "debug": true,

  "sitemapLocation": "sitemap.json"
}
```

**核心配置项详解：**

| 配置项 | 类型 | 说明 | Spring Boot 类比 |
|--------|------|------|-----------------|
| `pages` | String[] | 页面路径列表（第一项为首页） | Controller 路由映射 |
| `window` | Object | 全局默认窗口表现 | `spring.mvc.view.prefix` |
| `tabBar` | Object | 底部/顶部 Tab 栏配置 | — |
| `networkTimeout` | Object | 网络请求超时时间（ms） | `spring.mvc.async.request-timeout` |
| `debug` | Boolean | 是否开启调试模式 | `logging.level.root=DEBUG` |

**window 配置项详解：**

| 属性 | 说明 | 默认值 |
|------|------|--------|
| `navigationBarBackgroundColor` | 导航栏背景颜色 | `#000000` |
| `navigationBarTitleText` | 导航栏标题文字 | — |
| `navigationBarTextStyle` | 导航栏标题颜色（仅 `black`/`white`） | `white` |
| `backgroundColor` | 窗口背景色 | `#ffffff` |
| `backgroundTextStyle` | 下拉 loading 样式（仅 `dark`/`light`） | `dark` |
| `enablePullDownRefresh` | 是否开启全局下拉刷新 | `false` |

> **类比理解：app.json = application.yml**
>
> | app.json | application.yml | 说明 |
> |----------|-----------------|------|
> | `pages` | `@RequestMapping` 路由 | 定义可访问的页面/接口 |
> | `window` | 视图解析器配置 | 控制全局 UI 表现 |
> | `tabBar` | 导航菜单配置 | 底部导航栏 |
> | `networkTimeout` | 超时配置 | 网络请求超时设置 |
> | `debug` | `debug: true` | 调试模式开关 |
>
> **pages 数组的特殊规则**：`pages` 数组中的第一个路径就是小程序的默认启动页面（首页），类似于 Spring Boot 中 `@SpringBootApplication` 所在包下的自动扫描起点。新增页面时，必须在 `pages` 数组中注册才能正常访问。

### 7.2 页面级 .json 配置

每个页面目录下都可以有一个 `.json` 文件，用于配置当前页面的窗口表现。页面级配置会**覆盖** `app.json` 中 `window` 的同名配置。

```json
// pages/product/detail.json
{
  "navigationBarTitleText": "商品详情",
  "navigationBarBackgroundColor": "#ffffff",
  "enablePullDownRefresh": false,
  "usingComponents": {
    "product-card": "/components/product-card/index",
    "rating-stars": "/components/rating-stars/index"
  }
}
```

```json
// pages/user/user.json
{
  "navigationBarTitleText": "个人中心",
  "navigationBarBackgroundColor": "#07c160",
  "navigationBarTextStyle": "white",
  "enablePullDownRefresh": true,
  "usingComponents": {
    "user-avatar": "/components/user-avatar/index"
  }
}
```

**页面级可配置项：**

| 属性 | 说明 |
|------|------|
| `navigationBarTitleText` | 当前页面导航栏标题 |
| `navigationBarBackgroundColor` | 当前页面导航栏背景色 |
| `navigationBarTextStyle` | 当前页面导航栏标题颜色 |
| `backgroundColor` | 当前页面窗口背景色 |
| `backgroundTextStyle` | 当前页面下拉 loading 样式 |
| `enablePullDownRefresh` | 是否启用当前页面下拉刷新 |
| `onReachBottomDistance` | 页面上拉触底距离（px） |
| `pageOrientation` | 页面旋转配置（`auto`/`portrait`/`landscape`） |
| `usingComponents` | 当前页面引入的自定义组件 |
| `disableScroll` | 是否禁止页面滚动 |

> **Java 开发者注意点**
>
> 页面级 `.json` 配置覆盖全局配置的机制，与 Spring Boot 中**配置优先级**的概念完全一致：
>
> ```
> 页面级 .json  >  app.json  >  框架默认值
> ```
>
> 这类似于：
> ```
> application-local.yml  >  application.yml  >  框架默认配置
> ```
>
> 此外，`usingComponents` 字段用于声明当前页面使用的自定义组件，类似于 Spring 中通过 `@Autowired` 或 `@Resource` 声明依赖注入的 Bean。

### 7.3 sitemap.json 搜索引擎配置

`sitemap.json` 用于配置小程序及其页面是否允许被微信索引，影响小程序在微信搜索中的可见性。

```json
{
  "desc": "关于本文件的更多信息，请参考文档",
  "rules": [
    {
      "action": "allow",
      "page": "pages/index/index",
      "params": ["*"],
      "matching": "exact"
    },
    {
      "action": "allow",
      "page": "pages/product/detail",
      "params": ["id"],
      "matching": "inclusive"
    },
    {
      "action": "disallow",
      "page": "pages/order/*"
    },
    {
      "action": "allow",
      "page": "*"
    }
  ]
}
```

**rules 字段说明：**

| 属性 | 说明 | 可选值 |
|------|------|--------|
| `action` | 是否允许被索引 | `allow`（允许）/ `disallow`（禁止） |
| `page` | 页面路径 | 具体路径或 `*`（所有页面） |
| `params` | 允许被索引的参数 | 参数名数组或 `*`（所有参数） |
| `matching` | 匹配规则 | `exact`（精确）/ `inclusive`（包含）/ `regex`（正则） |

> **类比理解：sitemap.json 与 robots.txt**
>
> `sitemap.json` 的功能类似于传统 Web 开发中的 `robots.txt` 文件，都是用来告诉搜索引擎哪些页面可以被索引，哪些不可以。如果你做过 SEO 优化，对这个概念应该不陌生。

### 7.4 project.config.json 项目工具配置

`project.config.json` 是微信开发者工具的项目配置文件，通常由开发者工具自动生成和维护，一般不需要手动修改。

```json
{
  "description": "项目配置文件",
  "packOptions": {
    "ignore": [],
    "include": []
  },
  "setting": {
    "bundle": false,
    "userConfirmedBundleSwitch": false,
    "urlCheck": true,
    "scopeDataCheck": false,
    "coverView": true,
    "es6": true,
    "postcss": true,
    "compileHotReLoad": false,
    "lazyloadPlaceholderEnable": false,
    "preloadBackgroundData": false,
    "minified": true,
    "autoAudits": false,
    "newFeature": false,
    "uglifyFileName": false,
    "uploadWithSourceMap": true,
    "useIsolateContext": true,
    "nodeModules": false,
    "enhance": true,
    "useMultiFrameRuntime": true,
    "useApiHook": true,
    "useApiHostProcess": true,
    "showShadowRootInWxmlPanel": true,
    "packNpmManually": false,
    "enableEngineNative": false,
    "packNpmRelationList": [],
    "minifyWXSS": true,
    "showES6CompileOption": false,
    "minifyWXML": true,
    "babelSetting": {
      "ignore": [],
      "disablePlugins": [],
      "outputPath": ""
    }
  },
  "compileType": "miniprogram",
  "libVersion": "3.3.4",
  "appid": "wx1234567890abcdef",
  "projectname": "my-miniprogram",
  "condition": {},
  "editorSetting": {
    "tabIndent": "insertSpaces",
    "tabSize": 2
  }
}
```

**常用配置项说明：**

| 配置项 | 说明 | 建议值 |
|--------|------|--------|
| `appid` | 小程序 AppID | 在微信公众平台申请 |
| `projectname` | 项目名称 | 自定义 |
| `setting.urlCheck` | 是否检查安全域名 | 开发阶段可设为 `false` |
| `setting.es6` | 是否转译 ES6 为 ES5 | `true` |
| `setting.minified` | 是否压缩代码 | 生产环境 `true` |
| `compileType` | 编译类型 | `miniprogram` |

> **Java 开发者注意点**
>
> `project.config.json` 类似于 Java 项目中的 `.idea`（IntelliJ IDEA）或 `.classpath`（Eclipse）等 IDE 配置文件。它记录了开发者工具的个性化设置，通常应该被纳入版本控制（`.gitignore` 中不应排除此文件），以便团队成员共享一致的开发环境配置。
>
> **`urlCheck` 配置**：开发阶段建议将 `urlCheck` 设为 `false`，这样可以请求任意域名的 API（不受合法域名限制）。生产环境必须设为 `true`，并在微信公众平台配置合法域名。这类似于 Spring Boot 中开发环境和生产环境使用不同的配置 profile。

### 7.5 配置文件完整关系图

```
小程序项目根目录/
├── app.js              # 小程序入口文件（逻辑）
├── app.json            # 全局配置（★ 核心配置）
├── app.wxss            # 全局样式
├── sitemap.json        # 搜索引擎索引配置
├── project.config.json # 开发者工具配置
├── pages/
│   ├── index/
│   │   ├── index.js
│   │   ├── index.json  # 页面级配置（覆盖全局）
│   │   ├── index.wxml
│   │   └── index.wxss
│   └── ...
└── ...
```

> **类比理解：配置层级关系**
>
> ```
> ┌─────────────────────────────────────────┐
> │  project.config.json  (IDE配置，不参与运行)  │
> ├─────────────────────────────────────────┤
> │  app.json            (全局配置，最高优先级)   │
> ├─────────────────────────────────────────┤
> │  页面级 .json        (页面配置，覆盖全局)     │
> ├─────────────────────────────────────────┤
> │  sitemap.json        (SEO配置，独立运行)     │
> └─────────────────────────────────────────┘
> ```
>
> 这与 Spring Boot 的多层级配置体系非常相似：
> - `project.config.json` ≈ IDE 配置文件（`.idea/workspace.xml`）
> - `app.json` ≈ `application.yml`（全局配置）
> - 页面级 `.json` ≈ `@ConfigurationProperties`（局部配置）
> - `sitemap.json` ≈ `robots.txt`（SEO 配置）

---

