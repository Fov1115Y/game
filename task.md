# 大便超人网页小游戏 Tasks

## 文件清单

| 操作 | 文件 | 职责 |
|------|------|------|
| 新建 | `index.html` | 单文件网页小游戏入口，包含 HTML 结构、CSS 样式和 JavaScript 逻辑 |
| 已有 | `spec.md` | 已批准的需求说明，作为实现边界 |
| 已有 | `plan.md` | 已批准的技术设计，作为实现方案 |
| 新建 | `task.md` | 本任务拆解文档 |
| 新建 | `checklist.md` | 后续验收清单 |

## T1: 搭建单文件页面骨架

**文件：** `index.html`  
**依赖：** 无  
**步骤：**
1. 创建标准 HTML 文档结构，设置中文语言、字符集、视口和页面标题。
2. 添加主容器、游戏舞台、角色区域、立牌区域、名字输入区、发射方式区、发射按钮、重置按钮和命中计数区域。
3. 为关键元素设置稳定的 `id` 或 class，匹配 `plan.md` 中的 Markup 模块接口。
4. 添加默认立牌名字和默认命中计数，保证页面无脚本时也有基础可见结构。

**验证：** 用浏览器打开 `index.html`，期望看到页面标题、角色区域、立牌区域、输入框、发射按钮、重置按钮和命中计数。

## T2: 实现卡通视觉样式

**文件：** `index.html`  
**依赖：** T1  
**步骤：**
1. 在 `<style>` 中定义整体页面布局，让游戏舞台居中显示。
2. 使用 CSS 绘制红色紧身衣卡通角色、立牌、背景、地面和操作面板。
3. 定义按钮、输入框、发射方式卡片、计数显示的小游戏风格样式。
4. 定义弹幕、污渍、短文本提示和命中抖动的基础 class 与动画。
5. 确保不引用外部图片、字体、音频或网络资源。

**验证：** 用浏览器打开 `index.html`，期望看到卡通化红衣角色、对面立牌、清晰操作区，开发者工具 Network 面板无外部资源请求。

## T3: 实现状态与渲染逻辑

**文件：** `index.html`  
**依赖：** T1、T2  
**步骤：**
1. 在 `<script>` 中定义 `GameState` 初始状态，包括 `targetName`、`attackMode`、`hitCount`、`stainCount`、`isAnimating`。
2. 定义三种 `AttackMode` 配置：单发直线、连续多发、抛物线。
3. 缓存关键 DOM 元素引用。
4. 实现 `setTargetName(name)`、`setAttackMode(modeId)`、`addHit(amount)`、`resetGame()`。
5. 实现 `renderModeCards()`、`renderState()`、`clearVisualEffects()`。

**验证：** 在浏览器控制台调用 `setTargetName("测试目标")`、`setAttackMode("burst")`、`addHit(1)`、`resetGame()`，期望立牌名字、发射方式高亮、命中计数和视觉状态正确更新。

## T4: 绑定玩家操作事件

**文件：** `index.html`  
**依赖：** T3  
**步骤：**
1. 实现 `bindEvents()`，统一绑定名字确认、发射方式选择、发射按钮和重置按钮。
2. 玩家点击确认改名或在输入框按 Enter 时，读取输入值并调用 `setTargetName(name)`。
3. 玩家点击发射方式卡片时，调用 `setAttackMode(modeId)` 并刷新高亮状态。
4. 玩家点击重置按钮时，调用 `resetGame()`、`clearVisualEffects()` 和 `renderState()`。
5. 页面加载后调用 `renderModeCards()`、`renderState()`、`bindEvents()` 完成初始化。

**验证：** 用浏览器操作页面，期望改名、切换发射方式、重置按钮都能即时生效。

## T5: 实现发射动画

**文件：** `index.html`  
**依赖：** T3、T4  
**步骤：**
1. 实现 `fireAttack()`，根据当前 `attackMode` 找到对应发射配置。
2. 使用 `isAnimating` 限制发射期间重复触发。
3. 根据 `projectileCount` 循环调用 `createProjectile(options)` 创建弹幕元素。
4. 在 `createProjectile(options)` 中根据 `trajectory` 设置直线、散射或抛物线的 CSS 变量、延迟和动画 class。
5. 在动画结束后调用 `finishProjectile(projectile)` 清理弹幕。

**验证：** 逐一选择三种发射方式并点击发射，期望每种方式都能看到从角色到立牌方向的飞行动画，且视觉表现有差异。

## T6: 实现命中反馈

**文件：** `index.html`  
**依赖：** T5  
**步骤：**
1. 实现 `applyHitEffect(amount)`，统一处理单次或多次命中后的反馈。
2. 实现 `addStain()`，在立牌区域生成随机位置的卡通污渍。
3. 实现 `showToast(text)`，在立牌附近展示短文本提示并自动消失。
4. 在 `finishProjectile(projectile)` 中调用 `applyHitEffect(1)`。
5. 命中时给立牌添加 `hit-shake` class，动画结束后移除。
6. 确保 `hitCount` 增加、`stainCount` 累积、页面计数同步更新。

**验证：** 点击发射并等待命中，期望立牌抖动、出现污渍、显示短提示，命中计数增加；多次发射后污渍可以累积。

## T7: 完成首版自检与离线验证

**文件：** `index.html`  
**依赖：** T1、T2、T3、T4、T5、T6  
**步骤：**
1. 检查页面是否只依赖 `index.html`，不引用外部图片、音频、字体、脚本或样式。
2. 检查三种发射方式是否都能触发不同动画。
3. 检查改名、命中计数、污渍累积和重置是否符合 `spec.md`。
4. 检查快速点击发射按钮时是否不会造成明显状态混乱。
5. 用浏览器直接打开本地 `index.html` 完成一次端到端试玩。

**验证：** 断网或忽略网络的本地浏览器直接打开 `index.html`，期望完成“改名 → 选择三种方式之一 → 发射 → 命中反馈 → 多次累积 → 重置”的完整流程。

## 执行顺序

```text
T1 → T2 → T3 → T4 → T5 → T6 → T7
```
