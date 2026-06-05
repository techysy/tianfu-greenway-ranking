# `tianfu-itt.html` 架构分析

该文件是一个**单文件 Web 应用**（991 行），由三层构成：HTML 结构、CSS 样式、JavaScript 逻辑。整体是一个"天府绿道单飞 ITT 排行榜"展示页面。

---

## 1. HTML 结构层

页面分为三个主要区域：

| 区域 | 类名/选择器 | 功能 |
|------|------------|------|
| **Hero 区域** | `.hero-section` | 标题、副标题、赛段信息统计（96.29km / 593m爬升 / 平均坡度 / 海拔范围） |
| **排行榜区域** | `.ranking-section` | 排序按钮（用时/速度/心率/功率）+ 三个过滤器（QZ认证 / 女子组 / 团骑）+ 排行榜卡片容器 |
| **页脚** | `.footer` | 数据认证信息和页面维护者链接 |

```html
<div class="hero-section">
    <h1 class="hero-title">🚴 天府绿道单飞ITT排行榜</h1>
    <p class="hero-subtitle">成都绕城绿道 逆时针方向 | 0公里出发</p>
    <div class="stage-info">
        <h3>赛段信息</h3>
        <div class="stage-stats">
            <!-- 距离、爬升、坡度、海拔等统计 -->
        </div>
    </div>
</div>

<div class="ranking-section">
    <div class="section-header">
        <!-- 排序按钮 + 过滤标签 -->
    </div>
    <div class="cards-container" id="cardsContainer">
        <!-- JS 动态渲染排行榜卡片 -->
    </div>
</div>

<div class="footer">
    <!-- 数据认证 + 维护者信息 -->
</div>
```

---

## 2. CSS 样式层

### 设计系统

- 紫蓝渐变主题色（`#667eea → #764ba2`）
- 奖牌排名系统：前三名使用 gold/silver/bronze 样式（金色边框、银色边框、古铜色边框）
- 卡片悬停效果：`translateX(4px)` 左移 + 阴影加深
- 纯 CSS 实现的备注 tooltip（`.remark-tooltip`），hover 时显示
- 团队成员头像默认隐藏（`.group-members.hidden`），点击卡片切换显示

### 响应式设计（`@media max-width: 768px`）

- 标题字号缩小、卡片内元素换行（`flex-wrap: wrap`）
- 头像/徽章尺寸缩小、统计间距压缩

### 关键交互状态类

| 类名 | 作用 |
|------|------|
| `.sort-btn.active` | 排序按钮激活态（紫色渐变） |
| `.filter-tag.active` | 过滤标签激活态 |
| `.rider-card.gold` | 第1名金色边框+金色渐变背景 |
| `.rider-card.silver` | 第2名银色边框+灰色渐变背景 |
| `.rider-card.bronze` | 第3名古铜色边框+暖色渐变背景 |
| `.time-badge.fast` | 用时 ≤2:30:59 的记录红色高亮 |
| `.group-members.hidden` | 团队成员头像默认隐藏 |

### 布局方式

- 卡片容器：`display: grid; grid-template-columns: 1fr`（单列布局）
- 卡片内部：`display: flex` 横向排列，各部分通过 `flex-shrink: 0` 保证不压缩
- 统计行：`display: flex; gap` 横向排列

---

## 3. JavaScript 逻辑层

### 数据层（三个核心数组）

| 数组 | 条数 | 说明 | 关键字段 |
|------|------|------|---------|
| `ridersData` | 42条 | 骑手基本信息 | `name`, `stravaId`, `avatar`, `female` |
| `groupData` | 6条 | 团骑分组定义 | `groupId`, `name`, `members[]` |
| `activityData` | 48条 | 活动记录 | `stravaId`, `activityId`, `time`, `speed`, `heart`, `power`, `date`, `remark`, `qz`, `itt`, `groupId` |

### 状态管理（全局变量）

```javascript
showQzOnly        // QZ认证过滤
showFemaleOnly    // 女子组过滤
showGroupRideOnly // 团骑模式过滤
showPowerPerKg    // 功率显示切换（W ↔ W/kg）
```

### 核心函数调用链

```
页面加载
  └─ renderCards('time')          ← 默认按时排序渲染
       ├─ getBestActivityDataBySpeed()  ← 数据去重/筛选
       │    ├─ 按 stravaId + 模式(itt/group/solo) 分组
       │    ├─ QZ模式下优先选取认证记录
       │    └─ 取每组最快记录
       ├─ 过滤逻辑
       │    ├─ itt模式：只显示 itt=true 的单飞记录
       │    ├─ 团骑模式：只显示 groupId != null 的团骑记录
       │    ├─ 女子组：只显示 female=true 的骑手
       │    └─ QZ认证：只显示 qz=true 的记录
       ├─ 排序（按 sortBy 参数）
       │    ├─ time：compareTime() 字符串解析比较
       │    └─ speed/heart/power：降序排列
       └─ DOM 生成
            ├─ 排名徽章（gold/silver/bronze/other）
            ├─ 头像（带 Strava 链接 + QZ认证闪电标识）
            ├─ 骑手信息（名称链接 + 轮组/体重 + 队员头像）
            ├─ 时间徽章（≤2:30:59 红色高亮）
            ├─ 速度/心率/功率 统计
            ├─ 日期链接 + 备注 tooltip
            └─ 点击事件：切换团队成员显示
```

### 关键辅助函数

| 函数 | 作用 |
|------|------|
| `compareTime(timeA, timeB)` | 将 `H:MM:SS` 解析为秒数比较 |
| `isFastTime(time)` | 判断用时是否 ≤ 2:30:59 |
| `getRankClass(index)` | 根据索引返回 gold/silver/bronze/other |
| `getFemaleGroupIds()` | 筛选全员为女性的团骑组ID |
| `getGroupMembers(groupId, excludeId)` | 获取某团骑组的其他成员 |
| `updateDate()` | 更新页脚日期 |

---

## 4. 架构特点总结

### 优点

- **零依赖**：纯静态 HTML，无框架、无构建工具，直接部署即可运行
- **数据与视图分离**：数据数组与渲染逻辑独立，结构清晰
- **CSS 语义化**：类名命名直观（`.rider-card`, `.time-badge`, `.rank-badge`），易于维护
- **响应式友好**：移动端适配合理，卡片在窄屏下自动换行

### 可改进点

- **数据硬编码**：骑手和活动数据直接写在 JS 中，新增记录需手动编辑代码，建议迁移到独立 JSON 文件或后端 API
- **`renderCards` 函数过长**：约 170 行，包含过滤、排序、DOM 生成等多种职责，可拆分为独立函数
- **排序状态判断脆弱**：通过查询 DOM 按钮文本内容判断当前排序方式，建议改为参数显式传递
- **缺少错误处理**：头像加载失败、数据异常等情况没有兜底逻辑

### 数据流

```
数组数据 → 过滤/去重 → 排序 → DOM 渲染
```

单向数据流，无框架，整体逻辑简单直观。
