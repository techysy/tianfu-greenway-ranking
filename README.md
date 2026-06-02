# 天府绿道ITT排行榜 - HTML架构文档

## 文件概述
- **文件名**: tianfu-itt.html
- **用途**: 展示成都绕城绿道逆时针单飞ITT排行榜
- **技术栈**: 纯HTML + CSS + JavaScript

---

## 1. HTML结构

### 1.1 Head部分 (L3-L439)
- **meta标签**: 字符集、视口设置
- **title**: 页面标题
- **style标签**: 完整的CSS样式定义

### 1.2 Body部分 (L440-L767)
```
body
├── .hero-section (英雄区)
│   ├── .hero-title (标题)
│   ├── .hero-subtitle (副标题)
│   └── .stage-info (赛段信息卡片)
│       └── .stage-stats (赛段数据统计)
│
├── .ranking-section (排行榜区)
│   ├── .section-header (头部区域)
│   │   ├── .section-title (标题)
│   │   ├── #statsCount (统计信息)
│   │   ├── 提示文字
│   │   ├── 排序按钮 (用时/速度/心率/功率)
│   │   ├── 仅显示认证记录 复选框
│   │   └── 仅显示女子组 复选框
│   └── .cards-container (#cardsContainer, 骑手卡片容器)
│
└── .footer (页脚)
    ├── 认证人信息
    └── 维护者信息
```

---

## 2. CSS样式结构

### 2.1 全局样式
- **重置样式** (L8-L12): margin、padding、box-sizing
- **body样式** (L14-L19): 字体、背景、颜色

### 2.2 英雄区样式 (.hero-section)
- 渐变背景、标题、赛段信息卡片、统计数据展示

### 2.3 排行榜区样式
- **.ranking-section**: 最大宽度、居中布局
- **.section-header**: 白色背景、圆角、阴影
- **.cards-container**: Grid布局、卡片间距

### 2.4 骑手卡片样式 (.rider-card)
- **基础样式**: 白色背景、圆角、阴影、Flex布局
- **奖牌样式**: 
  - .gold: 金牌渐变
  - .silver: 银牌渐变  
  - .bronze: 铜牌渐变
- **排名徽章** (.rank-badge): 圆形、颜色区分
- **头像** (.avatar): 圆形、渐变背景
- **骑手信息** (.rider-info): 名称、框高、体重
- **时间徽章** (.time-badge): 等宽字体、快速/普通样式
- **统计数据** (.stats-inline): 速度、心率、功率
- **卡片底部** (.card-footer): 日期、备注
- **备注提示** (.remark-tooltip): 悬停显示

### 2.5 其他组件样式
- **.sort-btn**: 排序按钮样式
- **.highlight-link**: 高亮链接样式
- **.date-link**: 日期链接样式
- **.empty-state**: 空状态样式

### 2.6 响应式样式 (@media)
- 移动端适配: 字体大小、间距、卡片布局调整

---

## 3. JavaScript数据结构

### 3.1 状态变量 (L507-L509)
```javascript
let showPowerPerKg = false;  // 是否显示功率/体重比
let showQzOnly = false;      // 是否仅显示认证记录
let showFemaleOnly = false;  // 是否仅显示女子组
```

### 3.2 骑手基础数据 (ridersData, L511-L537)
```javascript
{
    name: "骑手名称",
    stravaId: 数字ID,
    avatar: "头像链接",
    female: false/true
}
```

### 3.3 ITT活动数据 (ittData, L539-L565)
```javascript
{
    stravaId: 骑手ID (关联用),
    activityId: 活动ID,
    time: "用时 (HH:MM:SS)",
    speed: 速度 (km/h),
    heart: 心率 (bpm),
    power: 功率 (W),
    date: "日期",
    remark: "备注",
    qz: false/true,  // 是否认证
    weight: 体重 (kg),
    heightFront: 前轮框高,
    heightBack: 后轮框高
}
```

---

## 4. JavaScript功能函数

### 4.1 数据处理函数
- **getBestIttDataBySpeed()** (L567-L576)
  - 功能: 每个骑手取速度最快的记录
  - 返回: 优化后的ITT数据数组

- **compareTime(timeA, timeB)** (L578-L584)
  - 功能: 比较两个时间字符串
  - 返回: 时间差（秒）

- **isFastTime(time)** (L586-L591)
  - 功能: 判断是否为快速记录（≤ 2:30:59）
  - 返回: boolean

- **getRankClass(index)** (L593-L598)
  - 功能: 根据排名获取样式类
  - 返回: 'gold' | 'silver' | 'bronze' | 'other'

### 4.2 交互控制函数
- **toggleQzFilter()** (L600-L605)
  - 功能: 切换"仅显示认证记录"筛选

- **toggleFemaleFilter()** (L607-L612)
  - 功能: 切换"仅显示女子组"筛选

- **togglePowerDisplay()** (L614-L619)
  - 功能: 切换功率显示（W / W/kg）

### 4.3 核心渲染函数
- **renderCards(sortBy = 'time')** (L621-L750)
  - 功能: 渲染骑手卡片列表
  - 参数: 'time' | 'speed' | 'heart' | 'power'
  - 流程:
    1. 设置排序按钮状态
    2. 获取最佳ITT数据
    3. 合并骑手和ITT数据
    4. 应用筛选条件
    5. 排序数据
    6. 渲染卡片到DOM
    7. 更新统计信息

- **updateDate()** (L752-L761)
  - 功能: 更新页面日期显示

### 4.4 初始化代码 (L763-L764)
```javascript
updateDate();
renderCards('time');
```

---

## 5. 数据关联方式
- 通过 `stravaId` 字段关联 `ridersData` 和 `ittData`
- 使用 `Map` 数据结构实现高效关联查找
- 过滤无活动ID的记录

---

## 6. 功能特性
1. **多种排序方式**: 用时、速度、心率、功率
2. **筛选功能**: 认证记录、女子组
3. **功率切换**: 可在W和W/kg之间切换
4. **快速记录高亮**: ≤ 2:30:59 的记录红色高亮
5. **奖牌标识**: 前三名特殊样式
6. **可跳转链接**: 头像 → Strava主页；日期 → Strava活动
7. **备注提示**: 悬停显示完整备注
8. **响应式设计**: 适配移动端

---

## 7. 文件大小统计
- **总行数**: 767行
- **CSS样式**: ~400行 (L7-L438)
- **HTML结构**: ~60行 (L440-L505)
- **JavaScript**: ~260行 (L506-L766)
- **数据数组**: ~70行 (L511-L565)

---

## 8. 更新记录
- 2024-10-01: 分离ridersData和ittData，按stravaId关联
- 2024-10-02: 添加女子组筛选功能
- 持续添加新骑手数据
