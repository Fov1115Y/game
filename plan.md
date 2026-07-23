# 大便超人网页小游戏 Plan

## 架构概览
首版采用单文件网页架构，所有结构、样式和脚本都放在 `index.html` 中。页面由 HTML 提供固定游戏区域和操作控件，CSS 负责卡通视觉、角色造型、立牌、弹幕、污渍和抖动动画，JavaScript 负责状态管理、玩家输入、发射方式切换、弹幕生成、命中反馈和重置逻辑。

游戏拆为四个逻辑层：

1. **界面层**
   负责渲染主画面、角色、立牌、名字输入框、发射方式选择、发射按钮、重置按钮和命中计数，对应 F1、F2、F6、F7、F8。

2. **状态层**
   维护立牌名字、当前发射方式、命中次数、污渍数量和临时动画元素，保证玩家改名、发射、计数和重置行为一致，对应 F2、F3、F5、F6、F7。

3. **交互层**
   绑定输入确认、发射方式选择、发射按钮和重置按钮事件，将玩家操作转换为状态更新和动画触发，对应 F2、F3、F4、F7。

4. **表现层**
   通过 CSS 动画和动态 DOM 元素实现飞行动画、命中抖动、污渍累积、短文本提示和不同发射方式的视觉差异，对应 F3、F4、F5、F6。

## 核心数据结构

### GameState
```js
{
  targetName: string,
  attackMode: "single" | "burst" | "arc",
  hitCount: number,
  stainCount: number,
  isAnimating: boolean
}
```

**字段说明：**
- `targetName`: 当前显示在立牌上的名字。
- `attackMode`: 当前选择的发射方式。
  - `"single"`: 单发直线攻击。
  - `"burst"`: 连续多发攻击。
  - `"arc"`: 抛物线攻击。
- `hitCount`: 当前命中次数或得分。
- `stainCount`: 当前立牌上的污渍数量。
- `isAnimating`: 当前是否处于发射动画中，用于避免按钮被过快连点造成状态混乱。

### AttackMode
```js
{
  id: "single" | "burst" | "arc",
  label: string,
  description: string,
  projectileCount: number,
  trajectory: "line" | "spread" | "arc"
}
```

**字段说明：**
- `id`: 发射方式标识。
- `label`: 显示给玩家看的方式名称。
- `description`: 简短说明该方式的表现差异。
- `projectileCount`: 单次点击产生的弹幕数量。
- `trajectory`: 弹幕运动轨迹类型。

### ProjectileOptions
```js
{
  mode: AttackMode,
  index: number,
  total: number
}
```

**字段说明：**
- `mode`: 当前发射方式配置。
- `index`: 当前弹幕在本次发射中的序号。
- `total`: 本次发射的弹幕总数，用于计算延迟、偏移和视觉差异。

## 模块设计

### Markup 模块
**职责：** 提供页面静态结构，包括游戏舞台、角色、立牌、名字输入区、发射方式选择区、操作按钮和计数显示。  
**对外接口：** DOM 元素标识，例如 `targetNameText`、`nameInput`、`modeSelect`、`fireButton`、`resetButton`、`hitCountText`、`stage`、`standee`。  
**依赖：** 无。  

### Style 模块
**职责：** 定义整体布局、卡通风格、角色造型、立牌样式、按钮状态、弹幕样式、污渍样式、命中抖动和提示动画。  
**对外接口：** CSS class，例如 `projectile`、`stain`、`hit-shake`、`toast`、`mode-card`、`active`。  
**依赖：** Markup 模块提供的 DOM 结构。  

### State 模块
**职责：** 保存并更新 `GameState`，提供状态初始化、改名、切换发射方式、增加命中、重置游戏等行为。  
**对外接口：**
- `state`
- `setTargetName(name)`
- `setAttackMode(modeId)`
- `addHit(amount)`
- `resetGame()`

**依赖：** 无。  

### Render 模块
**职责：** 根据 `GameState` 更新页面显示，包括立牌名字、命中计数、当前发射方式高亮、重置后的视觉清理。  
**对外接口：**
- `renderState()`
- `renderModeCards()`
- `clearVisualEffects()`

**依赖：** State 模块、Markup 模块。  

### Attack 模块
**职责：** 根据当前发射方式生成弹幕动画，处理多发、散射和抛物线视觉差异，并在动画结束时触发命中反馈。  
**对外接口：**
- `fireAttack()`
- `createProjectile(options)`
- `finishProjectile(projectile)`

**依赖：** State 模块、Render 模块、Effects 模块、Markup 模块。  

### Effects 模块
**职责：** 生成命中反馈，包括立牌抖动、污渍累积、短文本提示和命中计数变化。  
**对外接口：**
- `applyHitEffect(amount)`
- `addStain()`
- `showToast(text)`

**依赖：** State 模块、Render 模块、Markup 模块。  

### Events 模块
**职责：** 统一绑定玩家操作事件，包括确认改名、切换发射方式、点击发射和点击重置。  
**对外接口：**
- `bindEvents()`

**依赖：** State 模块、Render 模块、Attack 模块、Markup 模块。

## 模块交互
1. 页面加载后，Markup 模块提供静态 DOM，State 模块初始化默认状态。
2. Events 模块调用 `renderModeCards()` 生成三种发射方式选项，并调用 `renderState()` 同步初始画面。
3. 玩家输入名字并确认时，Events 模块读取输入值，调用 `setTargetName(name)` 更新状态，再调用 `renderState()` 更新立牌文字。
4. 玩家选择发射方式时，Events 模块调用 `setAttackMode(modeId)` 更新状态，再调用 `renderState()` 更新当前方式高亮。
5. 玩家点击发射时，Events 模块调用 `fireAttack()`。
6. Attack 模块读取当前 `attackMode`，按配置调用 `createProjectile(options)` 创建一个或多个弹幕元素。
7. 每个弹幕动画结束后，Attack 模块调用 `finishProjectile(projectile)` 移除弹幕，并调用 Effects 模块的 `applyHitEffect(amount)`。
8. Effects 模块调用 `addHit(amount)` 增加命中计数，调用 `addStain()` 添加污渍，调用 `showToast(text)` 显示短提示，再调用 `renderState()` 更新计数。
9. 玩家点击重置时，Events 模块调用 `resetGame()` 和 `clearVisualEffects()`，再调用 `renderState()` 回到可继续游玩的状态。

## 文件组织
```text
myskill/
├── spec.md        — 已批准的需求说明
├── plan.md        — 技术设计
├── task.md        — 后续任务拆解
├── checklist.md   — 后续验收清单
└── index.html     — 单文件网页小游戏，包含 HTML、CSS、JavaScript
```

首版只创建 `index.html` 作为运行入口，不拆分额外资源文件，满足离线打开和单文件 MVP 的要求。

## 技术决策

| 决策点 | 选择 | 理由 |
|--------|------|------|
| 运行形式 | 单文件网页 `index.html` | 满足无需安装依赖、离线打开、快速可玩，符合首版 MVP 范围。 |
| 技术栈 | 原生 HTML + CSS + JavaScript | 项目功能简单，不需要框架；可减少依赖和构建步骤。 |
| 视觉实现 | CSS 绘制 + 文本符号 + DOM 元素动画 | 不依赖外部图片、音频、字体或网络资源，满足离线运行。 |
| 发射方式 | 单发直线、连续多发、抛物线三种 | 覆盖“至少三种发射方式”，且实现复杂度适合首版。 |
| 状态管理 | 单个 `GameState` 对象 | 状态规模小，集中管理更直接，避免引入复杂架构。 |
| 动画方式 | CSS transition/animation + JavaScript 动态创建元素 | 可实现飞行动画、抖动、污渍和提示，同时保持轻量。 |
| 连点处理 | `isAnimating` 限制发射期间重复触发 | 避免快速点击造成弹幕、计数或清理逻辑混乱。 |
| 内容边界 | 卡通恶搞、不写实、不针对受保护群体 | 保持小游戏风格，同时避免不必要的伤害性表达。 |
