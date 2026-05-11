# 心流旅人(SoulTraveler) — AI 构建包

> 来源:心流旅人-模块2-文档.md
> 生成时间:2026-05
> 面向工具:Claude Code / Cursor

---

## 0 · 元数据表

| 项 | 值 |
|---|---|
| 产品 ID | soultraveler |
| 产品名 | 心流旅人(SoulTraveler) |
| 版本 | V1.0-MVP |
| 来源模块 2 | 心流旅人-模块2-文档.md |
| 生成时间 | 2026-05 |
| 面向 AI 工具 | Claude Code / Cursor |
| 阶段性质 | MVP |
| 目标平台 | 微信小程序 |

---

## 1 · 技术选型

| 项 | 选型 |
|---|---|
| 目标平台 | 微信小程序 |
| 前端框架 + 版本 | 微信小程序原生(WXML + WXSS + JavaScript) |
| 状态管理 | 全局 App globalData + 页面 data |
| 样式方案 | WXSS,全局变量文件 variables.wxss |
| 后端存储 | 微信云开发(云数据库 + 云函数),本地 wx.setStorageSync 做缓存 |
| 部署形态 | 微信小程序平台发布,云开发自动部署 |
| AI 调用方 | 无运行时 AI 调用(AI 仅用于离线内容生产) |
| 音频方案 | wx.createInnerAudioContext() |
| 动画方案 | CSS 动画 + WXSS keyframes |

---

## 2 · 数据模型

### 实体:User

| 字段名 | 类型 | 必填 | 取值约束 | 说明 |
|---|---|---|---|---|
| userId | string | 是 | 微信 openid | 用户唯一标识 |
| createdAt | date | 是 | ISO 日期 | 注册时间 |
| totalFocusMinutes | number | 是 | 整数,最小 0 | 累计专注分钟数 |
| totalTomatoes | number | 是 | 整数,最小 0 | 累计完成番茄数 |
| todayTomatoes | number | 是 | 整数,最小 0 | 今日完成番茄数 |
| todayDate | string | 是 | YYYY-MM-DD | 用于判断跨天重置 |
| currentStreak | number | 是 | 整数,最小 0 | 连续使用天数 |
| currentCityId | string | 是 | city_01 至 city_10 | 当前所在城市 ID |
| fragments | number | 是 | 整数,0 至 4 | 当前持有碎片数(解锁后清零) |
| unlockedCities | array | 是 | 城市 ID 数组 | 已解锁的城市列表 |
| collectedItems | array | 是 | 收集物 ID 数组 | 已获得的收集物列表 |
| completedEvents | array | 是 | 事件 ID 数组 | 已完成的探索事件列表 |
| focusDuration | number | 是 | enum: 15, 25, 45 | 默认专注时长(分钟) |
| restDuration | number | 是 | 固定值 5 | 休息时长(分钟) |
| soundEnabled | boolean | 是 | true / false | 环境白噪音开关 |
| vibrationEnabled | boolean | 是 | true / false | 振动反馈开关 |

### 实体:City

| 字段名 | 类型 | 必填 | 取值约束 | 说明 |
|---|---|---|---|---|
| cityId | string | 是 | city_01 至 city_10 | 城市唯一 ID |
| name | string | 是 | 中文名 | 城市名称 |
| nameEn | string | 是 | 英文名 | 城市英文名 |
| subtitle | string | 是 | 不超过 10 字 | 城市副标题 |
| description | string | 是 | 2-3 句话 | 城市氛围描写 |
| order | number | 是 | 1 至 10 | 路线顺序 |
| fragmentsNeeded | number | 是 | city_01 为 0,其余为 5 | 解锁所需碎片数 |
| backgroundImage | string | 是 | 图片路径 | 冥想场景背景图 |
| ambientAudio | string | 是 | 音频路径 | 环境白噪音文件 |
| eventIds | array | 是 | 3 个事件 ID | 该城市的探索事件 |
| collectibleIds | array | 是 | 3-4 个收集物 ID | 该城市的收集物 |

### 实体:ExploreEvent

| 字段名 | 类型 | 必填 | 取值约束 | 说明 |
|---|---|---|---|---|
| eventId | string | 是 | event_XX_YY 格式 | 事件唯一 ID |
| cityId | string | 是 | 对应城市 ID | 所属城市 |
| title | string | 是 | 不超过 10 字 | 事件标题 |
| description | string | 是 | 3-5 句叙事 | 事件叙事文字 |
| funFact | string | 是 | 1-2 句 | 趣味冷知识 |
| rewardCollectibleId | string | 是 | 收集物 ID | 完成后获得的收集物 |

### 实体:Collectible

| 字段名 | 类型 | 必填 | 取值约束 | 说明 |
|---|---|---|---|---|
| collectibleId | string | 是 | col_XX_YY 格式 | 收集物唯一 ID |
| cityId | string | 是 | 对应城市 ID | 所属城市 |
| name | string | 是 | 不超过 8 字 | 收集物名称 |
| type | string | 是 | enum: scenery, food, scroll | 类型:风景 / 美食 / 卷轴 |
| image | string | 是 | 图片路径 | 收集物图片 |
| description | string | 是 | 1-2 句 | 收集物描述 |
| scrollContent | string | 否 | 仅 type=scroll 时必填 | 名言内容 |
| scrollAuthor | string | 否 | 仅 type=scroll 时必填 | 名言作者 |

### 实体:FocusSession

| 字段名 | 类型 | 必填 | 取值约束 | 说明 |
|---|---|---|---|---|
| sessionId | string | 是 | 自动生成 | 本次冥想唯一 ID |
| userId | string | 是 | 用户 openid | 所属用户 |
| startTime | date | 是 | ISO 日期时间 | 开始时间 |
| endTime | date | 否 | ISO 日期时间 | 结束时间 |
| duration | number | 是 | enum: 15, 25, 45 | 选择的时长(分钟) |
| cityId | string | 是 | 城市 ID | 冥想时所在城市 |
| status | string | 是 | enum: completed, abandoned | 完成 / 中途放弃 |
| fragmentsEarned | number | 是 | 0, 1, 2, 4 | 本次获得碎片数 |
| abandonedAtSecond | number | 否 | 仅 status=abandoned 时填 | 放弃时剩余秒数 |

### 实体:AnalyticsEvent

| 字段名 | 类型 | 必填 | 取值约束 | 说明 |
|---|---|---|---|---|
| analyticsId | string | 是 | 自动生成 | 埋点事件 ID |
| userId | string | 是 | 用户 openid | 所属用户 |
| eventType | string | 是 | enum: focus_start, focus_complete, focus_abandon, city_unlock, explore_click, journal_view, map_view | 埋点事件类型 |
| timestamp | date | 是 | ISO 日期时间 | 事件发生时间 |
| payload | object | 否 | 自由结构 | 附加数据(城市 ID、时长等) |

---

## 3 · 页面结构

### 页面:冥想主页(首页)

| 项 | 值 |
|---|---|
| 路由 | /pages/meditation/meditation |
| 读取实体 | User, City |
| 写入实体 | User, FocusSession, AnalyticsEvent |
| 用户身份限制 | 登录(微信自动登录) |

主要区域:
- **顶部信息栏**:当前城市名称 + 碎片进度条(当前碎片数 / 所需碎片数)
- **场景区**:占屏幕 60% 高度,展示当前城市冥想背景图 + 云宝角色动画
- **计时区**:倒计时数字(大号等宽字体)+ 圆形进度环
- **操作区**:未开始时显示时长选择(15/25/45)+ 开始按钮;冥想中显示暂停 + 放弃按钮
- **底部导航栏**:冥想 / 地图 / 手账 / 我的 四个 Tab

### 页面:世界地图

| 项 | 值 |
|---|---|
| 路由 | /pages/map/map |
| 读取实体 | User, City |
| 写入实体 | User, AnalyticsEvent |
| 用户身份限制 | 登录 |

主要区域:
- **地图区**:横向可滚动的丝绸之路路线图,10 个城市节点用虚线连接
- **城市节点**:已到达(金色亮起)/ 当前位置(脉动标记)/ 下一城(碎片提示)/ 未到达(迷雾覆盖)
- **城市详情卡片**:点击已到达城市弹出城市介绍卡片
- **进度指示**:顶部显示 X/10 城市已探索
- **底部导航栏**:同上

### 页面:地点探索

| 项 | 值 |
|---|---|
| 路由 | /pages/explore/explore |
| 读取实体 | City, ExploreEvent, Collectible |
| 写入实体 | User, AnalyticsEvent |
| 用户身份限制 | 登录 |

主要区域:
- **城市介绍区**:场景插画 + 城市名 + 副标题 + 氛围描写
- **事件列表**:2-3 个可点击的探索事件卡片
- **事件详情弹层**:叙事文字 + 冷知识 + 收集物获得动画
- **返回按钮**:返回冥想页或地图页

### 页面:旅行手账

| 项 | 值 |
|---|---|
| 路由 | /pages/journal/journal |
| 读取实体 | User, City, Collectible |
| 写入实体 | AnalyticsEvent |
| 用户身份限制 | 登录 |

主要区域:
- **顶部进度**:收集物总进度(X/35)
- **城市分页**:按城市顺序排列,每个城市展示其收集物
- **收集物卡片**:已收集(图片 + 名称 + 描述)/ 未收集(问号占位)
- **空态**:新用户无收集物时显示引导("开始冥想,踏上旅程吧")
- **底部导航栏**:同上

### 页面:个人中心

| 项 | 值 |
|---|---|
| 路由 | /pages/profile/profile |
| 读取实体 | User |
| 写入实体 | User |
| 用户身份限制 | 登录 |

主要区域:
- **用户信息**:微信头像 + 昵称
- **数据统计卡片**:累计专注分钟数、完成番茄数、连续使用天数
- **旅程状态**:当前城市、旅行进度百分比、收集物完成度
- **设置区**:专注时长选择、声音开关、振动开关
- **底部导航栏**:同上

---

## 4 · 交互流程

### 流程 A · 开始冥想并获得碎片

**前置条件**:用户已登录,处于冥想主页,番茄钟状态为 idle

| 步骤 | 触发 | 系统动作 | 涉及实体 | 失败态处理 |
|---|---|---|---|---|
| 1 | 用户点击时长选择按钮(15/25/45) | 更新 User.focusDuration 为所选值,高亮选中按钮 | User | 无 |
| 2 | 用户点击"开始冥想" | 创建 FocusSession(status=focusing, duration=所选值, cityId=User.currentCityId);启动倒计时;播放当前城市 ambientAudio(fadeIn 1 秒);设置屏幕常亮;云宝切换为 focusing 状态;隐藏时长选择和开始按钮,显示暂停和放弃按钮;上报 AnalyticsEvent(focus_start) | User, FocusSession, City, AnalyticsEvent | 音频播放失败:静默处理,不影响倒计时 |
| 3 | 每秒 tick | 更新 remainingSeconds - 1;更新计时显示(MM:SS);更新圆形进度环 | FocusSession | 后台恢复:计算时间差补偿 remainingSeconds |
| 4 | 用户点击暂停 | 暂停倒计时;音频 fadeOut;暂停按钮变为"继续";云宝切换为 idle 状态 | FocusSession | 无 |
| 5 | 用户点击继续 | 恢复倒计时;音频 fadeIn;继续按钮变为"暂停";云宝切换为 focusing 状态 | FocusSession | 无 |
| 6 | 用户点击放弃 | 弹出确认对话框"灵魂还没准备好远行,确定要中断吗?" | 无 | 无 |
| 6a | 用户确认放弃 | 停止倒计时;音频 fadeOut;FocusSession.status = abandoned;FocusSession.abandonedAtSecond = remainingSeconds;fragmentsEarned = 0;云宝切换为 idle;恢复未开始状态;上报 AnalyticsEvent(focus_abandon) | FocusSession, AnalyticsEvent | 无 |
| 6b | 用户取消放弃 | 关闭对话框,继续计时 | 无 | 无 |
| 7 | 倒计时归零 | 播放钵声提示音;wx.vibrateShort();FocusSession.status = completed;计算 fragmentsEarned(15 分钟=1, 25 分钟=2, 45 分钟=4);User.fragments += fragmentsEarned;User.totalFocusMinutes += duration;User.totalTomatoes += 1;User.todayTomatoes += 1;弹出完成卡片("冥想完成!获得 X 个钥匙碎片");碎片飞入进度条动画;若 User.fragments >= 5,显示"可以解锁新地点了!去看看→";音频 fadeOut;上报 AnalyticsEvent(focus_complete) | User, FocusSession, AnalyticsEvent | 无 |
| 8 | 完成卡片关闭 | 自动进入 5 分钟休息倒计时;云宝切换为 resting;显示"探索当前城市"按钮 | User | 无 |
| 9 | 休息倒计时归零 | 提示"是否开始下一个番茄?";恢复未开始状态 | 无 | 无 |

**终点**:用户完成一次冥想,获得碎片,可选进入休息或查看地图

**禁区**:
- 冥想中不允许切换到其他 Tab(提示"冥想中无法离开")
- 不允许同时运行多个计时器

### 流程 B · 解锁新城市

**前置条件**:User.fragments >= 5,当前城市不是最后一个(city_10)

| 步骤 | 触发 | 系统动作 | 涉及实体 | 失败态处理 |
|---|---|---|---|---|
| 1 | 用户点击碎片进度条"可以出发了"或地图页下一城市节点 | 检查 User.fragments >= 5 | User | 碎片不足:显示"还需要 X 个碎片才能出发,继续冥想吧" |
| 2 | 验证通过 | 计算 nextCityId = 当前城市 order + 1 对应的城市;User.fragments -= 5;User.unlockedCities.push(nextCityId);User.currentCityId = nextCityId;播放解锁动画(迷雾散去,城市名和场景逐渐显现);上报 AnalyticsEvent(city_unlock, payload={cityId: nextCityId}) | User, City, AnalyticsEvent | 无 |
| 3 | 解锁动画完成 | 冥想页场景自动切换为新城市的 backgroundImage 和 ambientAudio;更新碎片进度条显示 | User, City | 无 |

**终点**:新城市解锁,冥想场景更新

**禁区**:
- 不允许跳过城市解锁(必须按顺序)
- 不允许碎片不足时强制解锁

### 流程 C · 探索城市收集物品

**前置条件**:用户已到达该城市(cityId 在 User.unlockedCities 中)

| 步骤 | 触发 | 系统动作 | 涉及实体 | 失败态处理 |
|---|---|---|---|---|
| 1 | 用户进入探索页(从休息按钮或地图点击) | 加载 City 数据(name, subtitle, description, backgroundImage);加载该城市的 ExploreEvent 列表;标记已完成事件 | City, ExploreEvent, User | 无 |
| 2 | 用户点击一个未完成的事件卡片 | 展开事件详情弹层(description + funFact);上报 AnalyticsEvent(explore_click, payload={eventId}) | ExploreEvent, AnalyticsEvent | 无 |
| 3 | 用户阅读完事件,系统发放收集物 | User.completedEvents.push(eventId);User.collectedItems.push(rewardCollectibleId);弹出收集物卡片(image + name + description);收集物飞入动画;事件卡片标记为已完成(灰色 + 勾号) | User, Collectible, ExploreEvent | 无 |
| 4 | 所有事件完成 | 显示"这座城市已经被你探索透啦,出发去下一站吧" | 无 | 无 |

**终点**:用户收集到物品,存入手账

**禁区**:
- 同一事件不可重复完成
- 不可探索未解锁城市的事件

### 流程 D · 查看旅行手账

**前置条件**:用户已登录

| 步骤 | 触发 | 系统动作 | 涉及实体 | 失败态处理 |
|---|---|---|---|---|
| 1 | 用户点击底部导航"手账" | 加载 User.collectedItems;按城市分组加载 Collectible 列表;计算总进度(collectedItems.length / 35);上报 AnalyticsEvent(journal_view) | User, Collectible, City, AnalyticsEvent | 空态:collectedItems 为空时显示引导"开始冥想,踏上旅程吧" |
| 2 | 用户点击已收集的物品卡片 | 展示收集物详情(大图 + 描述;卷轴类额外显示 scrollContent + scrollAuthor) | Collectible | 无 |
| 3 | 用户滚动浏览不同城市 | 切换城市分页;未收集的物品显示问号占位 | City, Collectible | 未解锁城市:城市名显示但收集物全部锁定 |

**终点**:用户浏览手账,获得收藏满足感

**禁区**:
- 不可删除已收集的物品

### 流程 E · 跨天重置

**前置条件**:用户打开小程序

| 步骤 | 触发 | 系统动作 | 涉及实体 | 失败态处理 |
|---|---|---|---|---|
| 1 | 小程序 onLaunch 或 onShow | 获取当前日期(YYYY-MM-DD),与 User.todayDate 对比 | User | 无 |
| 2 | 日期不同 | User.todayTomatoes = 0;若昨日 todayTomatoes > 0,User.currentStreak += 1;否则 User.currentStreak = 0;User.todayDate = 当前日期;保存数据 | User | 无 |
| 2a | 日期相同 | 不做任何操作 | 无 | 无 |

**终点**:每日数据正确重置

**禁区**:
- 不可重置 totalFocusMinutes 和 totalTomatoes(这是累计值)

---

## 5 · AI 决策点

本产品运行时无 AI 决策点。AI 仅用于离线内容生产阶段:

- 城市场景插画:Midjourney 生成,统一水彩风格 Prompt,人工筛选和后期统一色调
- 探索文案:GPT-4 / Claude 按模板生成初稿,人工做文化准确性审核和情感润色
- 角色"云宝"立绘:AI 图像生成多状态(idle / focus / complete / sleep / awake),人工确保风格一致性
- 趣味知识:AI 生成,人工 fact-check

所有 AI 生产的内容在开发阶段完成,打包为静态数据存入小程序,运行时直接读取,不调用 AI API。

---

## 6 · 规则判定点

### 规则 R1 · 碎片奖励计算

| 项 | 值 |
|---|---|
| 触发 | 冥想倒计时归零(FocusSession.status = completed) |
| 输入 | FocusSession.duration |
| 判定逻辑 | if duration == 15: fragmentsEarned = 1; elif duration == 25: fragmentsEarned = 2; elif duration == 45: fragmentsEarned = 4 |
| 输出 | fragmentsEarned(写入 FocusSession 和 User.fragments) |
| 优先级 | 1 |

### 规则 R2 · 城市解锁判定

| 项 | 值 |
|---|---|
| 触发 | 用户点击解锁城市按钮 |
| 输入 | User.fragments, User.currentCityId |
| 判定逻辑 | nextCity = Cities[currentCity.order + 1]; if User.fragments >= nextCity.fragmentsNeeded AND nextCity.order <= 10: allow_unlock = true; else: allow_unlock = false |
| 输出 | allow_unlock(boolean),若 true 则执行流程 B |
| 优先级 | 2 |

### 规则 R3 · 放弃冥想处理

| 项 | 值 |
|---|---|
| 触发 | 用户确认放弃冥想 |
| 输入 | FocusSession(当前会话) |
| 判定逻辑 | FocusSession.status = "abandoned"; FocusSession.fragmentsEarned = 0; 不增加 User.totalTomatoes; 不增加 User.totalFocusMinutes |
| 输出 | 放弃的会话记录(用于埋点分析) |
| 优先级 | 3 |

### 规则 R4 · 跨天判定

| 项 | 值 |
|---|---|
| 触发 | 小程序 onLaunch 或 onShow |
| 输入 | User.todayDate, 系统当前日期 |
| 判定逻辑 | if todayDate != currentDate: todayTomatoes = 0; if 昨日 todayTomatoes > 0: currentStreak += 1; else: currentStreak = 0; todayDate = currentDate |
| 输出 | 更新后的 User 数据 |
| 优先级 | 4 |

### 规则 R5 · 探索事件去重

| 项 | 值 |
|---|---|
| 触发 | 用户点击探索事件 |
| 输入 | ExploreEvent.eventId, User.completedEvents |
| 判定逻辑 | if eventId in User.completedEvents: show "已探索" 标记,不可重复触发; else: 允许探索 |
| 输出 | 是否允许探索(boolean) |
| 优先级 | 5 |

### 规则 R6 · 首城免费解锁

| 项 | 值 |
|---|---|
| 触发 | 新用户首次进入 |
| 输入 | User.unlockedCities |
| 判定逻辑 | if User.unlockedCities is empty: unlockedCities = ["city_01"]; currentCityId = "city_01"; fragments = 0 |
| 输出 | 初始化后的用户数据 |
| 优先级 | 0(最高) |

---

## 7 · 合规红线

### 红线 CL1 · 用户数据最小收集

| 项 | 值 |
|---|---|
| 触发 | 用户首次使用,获取用户信息时 |
| 必须做 | 仅收集微信 openid 和使用行为数据(专注时长、收集进度);首次使用时展示隐私政策弹窗,用户同意后方可使用 |
| 植入位置 | 流程 E 步骤 1(小程序初始化) |
| 绝对禁止 | 收集用户手机号、真实姓名、通讯录、位置信息;未经同意获取微信用户信息 |

### 红线 CL2 · 内容文化准确性

| 项 | 值 |
|---|---|
| 触发 | 内容生产阶段(离线) |
| 必须做 | 所有城市介绍、探索事件文案、哲思卷轴名言经过人工审核,确保文化准确性和尊重性;名人名言来源经过 fact-check |
| 植入位置 | 数据模型 City / ExploreEvent / Collectible 的静态数据填充阶段 |
| 绝对禁止 | 使用未经审核的 AI 生成文案直接上线;编造名人名言或历史事实;包含对任何民族、宗教、文化的歧视或不尊重内容 |

### 红线 CL3 · 插画版权合规

| 项 | 值 |
|---|---|
| 触发 | 内容生产阶段(离线) |
| 必须做 | AI 生成的插画需确认 AI 工具(Midjourney 等)的商用授权条款;白噪音素材需购买有商用授权的素材 |
| 植入位置 | 数据模型 City.backgroundImage 和 City.ambientAudio 的素材准备阶段 |
| 绝对禁止 | 使用未授权的图片或音频素材;使用可识别的真人照片 |

### 红线 CL4 · 免责声明展示

| 项 | 值 |
|---|---|
| 触发 | 用户首次使用或访问"关于"页面 |
| 必须做 | 在产品内展示免责声明:本产品提供专注辅助和放松体验,不构成医学冥想治疗或心理健康建议 |
| 植入位置 | 页面:个人中心 的设置区 |
| 绝对禁止 | 暗示本产品具有医疗或心理治疗效果 |

---

## 8 · 执行顺序与依赖

1. **建立数据模型** → 在 utils/data.js 中定义 City、ExploreEvent、Collectible 的静态数据;在 utils/storage.js 中定义 User 数据结构和存取方法
2. **建立页面骨架** → 创建 5 个页面的 WXML + WXSS 基础结构,配置 app.json 路由和自定义 tabBar
3. **实现交互流程** → 按流程 A → B → C → D → E 顺序实现:先做冥想计时器,再做城市解锁,再做探索收集,再做手账查看,最后做跨天重置
4. **接入 AI 决策点** → 本产品无运行时 AI 决策点,跳过
5. **植入规则判定** → 实现 R1-R6 规则,分别嵌入对应流程的对应步骤
6. **植入合规红线** → 实现 CL1(隐私弹窗)、CL4(免责声明);确认 CL2 和 CL3 的内容审核已完成
7. **自检** → 逐页面走通流程 A-E,确认碎片计算、城市解锁、收集物获取、手账展示、跨天重置均正常;确认四态(正常 / 失败 / 空 / 加载)在每个页面都有对应展示

---

## 自检结论

### 命名统一表

| 模块 2 原叫法 | 统一命名 |
|---|---|
| 用户 / 使用者 | User |
| 城市 / 地点 / 目的地 | City |
| 探索事件 / 随机事件 / 小事件 | ExploreEvent |
| 收集物 / 物件 / 小物件 | Collectible |
| 冥想 / 番茄钟 / 专注 | FocusSession |
| 碎片 / 钥匙碎片 | fragments(User 字段) |
| 手账 / 旅行手账 / 剪贴簿 | Journal(页面名) |
| 云宝 / 角色 / 小旅人 | character(组件名) |

### 字段唯一性扫描结果

全局唯一。所有字段名在实体间无重复。

### 模糊词扫描结果

零命中。

### 合规红线植入位置列表

| 红线 | 植入位置 |
|---|---|
| CL1 用户数据最小收集 | 流程 E 步骤 1(小程序初始化) |
| CL2 内容文化准确性 | 静态数据填充阶段(开发前) |
| CL3 插画版权合规 | 素材准备阶段(开发前) |
| CL4 免责声明展示 | 页面:个人中心 |

### 待产品方补充项清单

| 项 | 说明 |
|---|---|
| 10 个城市的完整探索事件文案 | 需按 ExploreEvent 数据模型逐条撰写,共 30 条 |
| 10 个城市的完整收集物文案 | 需按 Collectible 数据模型逐条撰写,共 35 条 |
| 10 张城市冥想场景图 | 需 Midjourney 生成并人工筛选 |
| 10 段城市环境白噪音 | 需购买授权素材 |
| 云宝角色多状态立绘 | idle / focus / complete / sleep / awake 共 5 个状态 |
| 隐私政策文本终稿 | 需法务审核 |
| 微信小程序 AppID | 需在微信公众平台注册获取 |

### 来源映射表

| 模块 2 章节 / 原始名词 | AI 构建包对应位置 |
|---|---|
| 2.4 成功指标 → 地点解锁率、走完全程率 | 实体 AnalyticsEvent.eventType(city_unlock) |
| 3.3 Journey A → 首次冥想 | 流程 A(开始冥想并获得碎片) |
| 3.3 Journey B → 解锁新城市 | 流程 B(解锁新城市) |
| 3.3 Journey C → 探索收集 | 流程 C(探索城市收集物品) |
| 4.3.1 番茄钟冥想 → 碎片奖励规则 | 规则 R1(碎片奖励计算) |
| 4.3.2 城市解锁 → 消耗碎片 | 规则 R2(城市解锁判定) |
| 8.1 当前城市 → 10 个可选值 | 实体 City(cityId: city_01 至 city_10) |
| 8.2 专注时长选择 | 实体 User.focusDuration(enum: 15, 25, 45) |
| 8.2 开始冥想 | 流程 A 步骤 2 |
| 8.2 解锁城市 | 流程 B 步骤 1 |
| 8.3 碎片进度条 | 页面:冥想主页 → 顶部信息栏 |
| 8.3 旅行手账 | 页面:旅行手账 |
| 8.4 冥想中 / 已暂停 / 休息中 / 冥想完成 | 流程 A 步骤 2-9 |
| 8.4 城市已解锁 / 城市未解锁 | 流程 B |
| 8.4 空态(首次使用) | 流程 D 步骤 1 失败态 |
| 8.5 技术选型 | 第 1 章 技术选型表 |
