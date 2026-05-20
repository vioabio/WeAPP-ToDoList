# 专注时钟 (XFocus) — 项目架构文档

> 版本：V 1.2.2 | 平台：微信小程序 | 框架：原生微信小程序 + WeUI

---

## 一、整体架构概览

```mermaid
graph TB
    subgraph "用户层 User Layer"
        A[用户设备<br/>WeChat Mini Program]
    end

    subgraph "视图层 View Layer (WXML + WXSS)"
        TAB[底部Tab导航 - 4页]
        SUB[子页面 - 6页]
    end

    subgraph "逻辑层 Logic Layer (JS)"
        APP[app.js<br/>全局初始化]
        PAGES[各页面 Page JS<br/>业务逻辑]
        UTILS[utils/<br/>工具函数]
    end

    subgraph "数据层 Data Layer (Storage)"
        STORAGE[wx.setStorageSync<br/>本地持久化]
        CACHE[页面 data<br/>运行时缓存]
    end

    subgraph "微信原生能力 WeChat Capabilities"
        API[基础API<br/>振动/分享/画布]
        BG[后台能力<br/>背景音频/定位]
        AD[广告组件<br/>Banner/视频/插屏]
        USER[用户信息<br/>open-data组件]
    end

    subgraph "外部依赖 External Dependencies"
        WXCHARTS[wxcharts.js<br/>图表库]
        WEUI[weui-miniprogram<br/>UI组件库]
        KUWO[酷我音乐API<br/>白噪声源]
    end

    A --> TAB
    A --> SUB
    TAB --> PAGES
    SUB --> PAGES
    PAGES --> APP
    PAGES --> UTILS
    PAGES --> STORAGE
    PAGES --> CACHE
    PAGES --> API
    PAGES --> BG
    PAGES --> AD
    PAGES --> USER
    PAGES --> WXCHARTS
    TAB --> WEUI
    SUB --> WEUI
```

---

## 二、页面导航架构

```mermaid
graph LR
    subgraph "底部Tab（4个主页面）"
        INDEX["📌 专注<br/>pages/index/index"]
        TODOS["📋 待办<br/>pages/todos/todos"]
        LOGS["📊 记录<br/>pages/logs/logs"]
        SETTING["👤 我<br/>pages/setting/setting"]
    end

    subgraph "子页面（6个）"
        RANK["🏆 排行榜<br/>pages/setting/rank/rank"]
        SETMORE["⚙️ 设置<br/>pages/setting/setmore/setmore"]
        ABOUT["ℹ️ 关于<br/>pages/about/about"]
        MORE["🔍 发现<br/>pages/more/more"]
        POSTER["🖼️ 打卡海报<br/>pages/poster/poster"]
        VERSION["📝 版本日志<br/>pages/version/version"]
    end

    subgraph "外部跳转"
        FEEDBACK["💬 反馈小程序<br/>wx8abaf00ee8c3202e"]
        AVATAR["🎭 头像工具小程序<br/>wxe333b834a643f99d"]
    end

    INDEX --> POSTER
    LOGS --> POSTER
    SETTING --> RANK
    SETTING --> SETMORE
    SETTING --> MORE
    SETTING --> ABOUT
    ABOUT --> VERSION
    SETTING --> FEEDBACK
    MORE --> AVATAR
```

---

## 三、数据流架构

```mermaid
flowchart TD
    subgraph "数据存储 (Storage)"
        WT[(workTime<br/>工作时长: Number)]
        RT[(restTime<br/>休息时长: Number)]
        VB[(vibison<br/>振动开关: Boolean)]
        LG[(logs<br/>专注记录: Array)]
        TD[(todo_list<br/>待办列表: Array)]
        TL[(todo_logs<br/>待办日志: Array)]
    end

    subgraph "页面读写"
        APP_INIT["app.js 初始化<br/>workTime=30, restTime=5"]
        INDEX_PAGE["pages/index<br/>番茄钟计时器"]
        TODOS_PAGE["pages/todos<br/>待办清单"]
        LOGS_PAGE["pages/logs<br/>记录图表"]
        SETMORE_PAGE["pages/setting/setmore<br/>设置页"]
        POSTER_PAGE["pages/poster<br/>海报生成"]
    end

    APP_INIT -->|写入默认值| WT
    APP_INIT -->|写入默认值| RT
    SETMORE_PAGE -->|读写| WT
    SETMORE_PAGE -->|读写| RT
    SETMORE_PAGE -->|读写| VB
    INDEX_PAGE -->|读取| WT
    INDEX_PAGE -->|读取| RT
    INDEX_PAGE -->|读取| VB
    INDEX_PAGE -->|写入记录| LG
    TODOS_PAGE -->|读写| TD
    TODOS_PAGE -->|读写| TL
    LOGS_PAGE -->|读取/清空| LG
    POSTER_PAGE -->|读取专注次数| LG
```

---

## 四、核心功能 — 番茄钟计时器流程

```mermaid
stateDiagram-v2
    [*] --> Idle: 进入首页
    
    Idle --> Working: 点击"工作"
    Idle --> Resting: 点击"休息"
    
    Working --> Working_Ticking: setInterval 启动<br/>每秒更新倒计时
    Working_Ticking --> Working_Ticking: 更新UI<br/>更新进度环角度
    Working_Ticking --> Working_Complete: 倒计时归零<br/>振动反馈(viblong)
    Working_Complete --> Idle: 显示完成图标
    
    Resting --> Resting_Ticking: setInterval 启动<br/>每秒更新倒计时
    Resting_Ticking --> Resting_Ticking: 更新UI<br/>更新进度环角度
    Resting_Ticking --> Resting_Complete: 倒计时归零<br/>振动反馈(viblong)
    Resting_Complete --> Idle: 显示完成图标

    note right of Working_Ticking: Date.now() 计算差值<br/>CSS rotate 控制进度环
    note left of Working: 写入 log 到 Storage
```

---

## 五、目录结构

```
WXminiprogram-Focus-clock/
│
├── app.js                  # 应用入口 — 初始化默认配置
├── app.json                # 全局配置 — 页面路由/TabBar/权限
├── app.wxss                # 全局样式
├── project.config.json     # 微信开发者工具配置
├── project.private.config.json  # 私有项目配置
├── sitemap.json            # 站点地图
├── package.json            # 依赖管理 (weui-miniprogram)
├── ARCHITECTURE.md         # 本架构文档
│
├── utils/                  # 工具模块
│   ├── util.js             # formatTime 时间格式化
│   └── wxcharts.js         # 图表绘制库 (72KB)
│
├── image/                  # 图标与图片资源 (~30个文件)
│   ├── yu.png / yu2.png    # Tab: 专注图标
│   ├── todo.png / todo2.png# Tab: 待办图标
│   ├── jilu.png / jilu2.png# Tab: 记录图标
│   ├── shezhi.png / shezhi2.png  # Tab: 我图标
│   ├── poster.jpg          # 打卡海报背景图
│   └── ...
│
└── pages/                  # 业务页面
    ├── index/              # [专注] 番茄钟计时器
    │   ├── index.js        # 核心计时逻辑
    │   ├── index.wxml      # 计时器UI
    │   ├── index.wxss      # 进度环/按钮样式
    │   └── index.json      # 页面配置
    │
    ├── todos/              # [待办] 待办清单
    │   ├── todos.js        # CRUD操作
    │   ├── todos.wxml      # 待办列表UI
    │   ├── todos.wxss      # 列表样式
    │   └── todos.json
    │
    ├── logs/               # [记录] 专注记录
    │   ├── logs.js         # 图表渲染 + 记录管理
    │   ├── logs.wxml       # 图表 + 列表UI
    │   ├── logs.wxss       # 图表样式
    │   └── logs.json
    │
    ├── setting/            # [我] 个人中心
    │   ├── setting.js      # 用户信息/白噪声控制
    │   ├── setting.wxml    # 个人中心UI
    │   ├── setting.wxss
    │   └── setting.json
    │   │
    │   ├── rank/           # 排行榜子页面
    │   ├── setmore/        # 设置子页面
    │
    ├── about/              # 关于页面
    ├── more/               # 发现(推荐)页面
    ├── poster/             # 打卡海报生成
    └── version/            # 版本日志
```

---

## 六、技术选型与设计决策

| 决策项 | 选择 | 说明 |
|--------|------|------|
| **框架** | 原生微信小程序 | 无第三方框架，直接使用微信JS-SDK |
| **UI组件** | WeUI (weui-miniprogram) | 微信官方UI库，风格统一 |
| **图表** | wxcharts.js | 轻量第三方库，绘制环形分析图 |
| **状态管理** | 无 | 纯 `setData` + `Storage`，无全局状态库 |
| **数据持久化** | wx.setStorageSync | 全本地存储，无后端服务 |
| **后台能力** | 背景音频 + 定位 | 支持白噪声后台播放 |
| **广告** | Banner/视频/插屏 | 接入微信流量主广告 |
| **外部API** | 酷我音乐 | 白噪声音频源 (mp3) |
| **云开发** | 未使用 | cloudfunctionRoot 已配但目录为空 |

---

## 七、关键数据模型

### 专注记录 (logs)
```json
{
  "startTime": "2022-01-15 14:30:00",
  "action": "work",         // "work" | "rest"
  "name": "编写代码",       // 自定义任务名
  "date": "2022-01-15"     // 日期，用于海报展示
}
```

### 待办事项 (todo_list)
```json
[
  {
    "id": "1642233600000",
    "context": "完成架构文档",
    "end": false             // true=已完成, false=未完成
  }
]
```

### 配置项
```json
{
  "workTime": 30,   // 工作时长 (1-60分钟)
  "restTime": 5,    // 休息时长 (1-30分钟)
  "vibison": false  // 振动反馈开关
}
```

---

## 八、广告组件分布

```mermaid
graph LR
    BANNER["Banner广告<br/>adunit-4547..."] --> LOGS["pages/logs (已注释)"]
    BANNER --> SETTING["pages/setting"]
    VIDEO["视频广告<br/>adunit-8dfc..."] --> MORE["pages/more"]
    INTERSTITIAL["插屏广告<br/>adunit-720b..."] --> ABOUT["pages/about"]
```

---

> 文档维护者：REALY | 最后更新：2026-05-20
