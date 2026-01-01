## 属性对照速查表（Horizontal TabLayout vs VerticalTabLayout）

> 本表用于快速判断：  
> **某个 TabLayout 属性在 VerticalTabLayout 中是否还能用，以及语义是否发生变化。**

---

### 1️⃣ 尺寸 / 布局相关属性

| 属性 | 官方 TabLayout（横向） | VerticalTabLayout（纵向） | 说明 |
|----|----|----|----|
| `android:layout_width` | 控件宽度 | 控件宽度 | 行为一致 |
| `android:layout_height` | 控件高度 | 控件高度 | 行为一致 |
| `tabMinWidth` | Tab 最小宽度 | **Tab 最小高度** | 语义反转 |
| `tabMaxWidth` | Tab 最大宽度 | **Tab 最大高度** | 语义反转 |
| `tabPadding` | 左右 padding 为主 | 上下 padding 为主 | 方向变化 |
| `tabPaddingStart/End` | 水平方向 padding | 垂直布局中的内边距 | 内部统一处理 |
| `tabContentStart` | 起始 X 偏移 | **顶部 inset（Top padding）** | 关键改造点 |

**记忆规则：**

> 在 VerticalTabLayout 中，**所有 width / X 轴相关的语义，都应当按 height / Y 轴理解。**

---

### 2️⃣ Tab 模式（tabMode）

| 值 | 官方 TabLayout 行为 | VerticalTabLayout 行为 |
|----|----|----|
| `fixed` | 所有 Tab 等宽，不滚动 | **所有 Tab 等高，不滚动** |
| `scrollable` | 横向可滚动 | **纵向可滚动** |
| `auto` | 空间足够时 fixed | 高度足够时 fixed，不够自动 scroll |

---

### 3️⃣ Tab Gravity（tabGravity）

| 值 | 官方语义 | VerticalTabLayout |
|----|----|----|
| `fill` | 填满水平方向 | **填满垂直高度** |
| `center` | 水平居中 | 垂直居中排列 |

---

### 4️⃣ 指示器（Indicator）相关属性（重点）

| 属性 | 官方 TabLayout | VerticalTabLayout | 说明 |
|----|----|----|----|
| `tabIndicator` | 底部横线 drawable | **左 / 中 / 右竖线 drawable** | 本质仍是 drawable |
| `tabIndicatorHeight` | 指示器高度 | **指示器宽度** | 非常关键 |
| `tabIndicatorColor` | 指示器颜色 | 行为一致 | tint 逻辑不变 |
| `tabIndicatorGravity` | bottom / center / top / stretch | left / center / right / stretch | 枚举复用，语义旋转 |
| `tabIndicatorFullWidth` | 填满 Tab 宽度 | 填满 Tab 高度 | 行为等价 |
| `tabIndicatorAnimationDuration` | X 轴动画时长 | **Y 轴动画时长** | 内部实现变化 |

> 本质理解：  
> **Indicator 并未重写，只是整体“旋转了 90°”。**

---

### 5️⃣ 文本 / 图标相关属性

| 属性 | VerticalTabLayout 行为 | 备注 |
|----|----|----|
| `tabTextAppearance` | 行为一致 | 完全复用 |
| `tabTextColor` | 行为一致 | 支持 ColorStateList |
| `tabSelectedTextColor` | 行为一致 | 内部合并状态 |
| `tabIconTint` | 行为一致 | 完全一致 |
| `tabIconTintMode` | 行为一致 | 完全一致 |
| `tabInlineLabel` | ⚠️ 仍然横排 | 不推荐多行文本 |

---

### 6️⃣ Ripple / 背景相关属性

| 属性 | VerticalTabLayout 行为 | 说明 |
|----|----|----|
| `tabBackground` | 行为一致 | 支持 selector |
| `tabRippleColor` | 行为一致 | Material Ripple |
| `tabUnboundedRipple` | 行为一致 | 直接复用 |

---

### 7️⃣ ViewPager / 交互 API

| API | 官方 TabLayout | VerticalTabLayout | 说明 |
|----|----|----|----|
| `setupWithViewPager()` | 支持 | 行为一致 | API 完全一致 |
| `selectTab()` | X 轴滚动 | **Y 轴滚动** | 内部重写 |
| `setScrollPosition()` | X 插值 | **Y 插值** | 动画方向变化 |
| `OnTabSelectedListener` | 回调一致 | 回调一致 | 无行为差异 |

---

### 8️⃣ 不推荐 / 无意义属性（纵向场景）

| 属性 | 原因 |
|----|----|
| `tabIndicatorPadding` | 在纵向模式下几乎无视觉效果 |
| 多行文本 + `tabInlineLabel` | 易导致高度测量异常 |
| `tabMaxWidth`（fixed 模式） | 会被 `weight` 直接覆盖 |

---

### ✅ 一句话总结

> **VerticalTabLayout = TabLayout 的纵向语义实现**  
>  
> 属性几乎全部可用，但：  
> **凡是涉及 width / X 轴的理解，必须转换为 height / Y 轴。**


---
---
---


# VerticalTabLayout 介绍 
- fork from https://github.com/secf4ult/VerticalTabLayout
- ZZH life

- 一个基于 **Material TabLayout** 深度改造的 **纵向 TabLayout** 组件，用于在 Android 中实现「左侧 / 右侧垂直标签栏 + 内容区域」的经典布局形态（如设置页、侧边导航、主从界面）。

---

## 一、项目背景与设计目标

Android 官方 `TabLayout` 仅支持 **横向布局**。在平板、横屏或桌面化 UI 场景下，垂直标签栏是非常常见的交互模式。本项目的目标是：

* **最大程度复用 Material TabLayout 能力**
* 支持 **ViewPager / PagerAdapter**
* 完整支持 **Material Attributes（tabIndicator / ripple / textAppearance 等）**
* 在 API 设计层面尽量保持与 `TabLayout` 一致，降低迁移成本

---

## 二、核心文件结构

```
verticaltablayout/
├── VerticalTabLayout.java   // 核心控件实现
└── attrs.xml                // 属性声明（复用 TabLayout styleable）
```

---

## 三、attrs.xml 设计分析（属性层）

### 1. 设计策略

`attrs.xml` **并没有新建 VerticalTabLayout 专属的 styleable**，而是：

> **直接复用 `declare-styleable name="TabLayout"`**

这样做的核心好处是：

* 可直接使用 `Widget.MaterialComponents.TabLayout` 相关主题
* XML 使用方式与官方 TabLayout 完全一致
* 主题 / 夜间模式 / 动态换肤天然兼容

---

### 2. 关键属性说明（与纵向改造相关）

| 属性                          | 作用                        | 纵向适配说明                                  |
| --------------------------- | ------------------------- | --------------------------------------- |
| `tabIndicatorHeight`        | 指示器尺寸                     | 在纵向模式中 **作为指示器宽度使用**                    |
| `tabIndicatorGravity`       | 指示器位置                     | 映射为 **LEFT / CENTER / RIGHT / STRETCH** |
| `tabContentStart`           | 内容起始偏移                    | 转换为 **Top inset**                       |
| `tabMinWidth / tabMaxWidth` | Tab 尺寸                    | 映射为 **高度约束**                            |
| `tabMode`                   | FIXED / SCROLLABLE / AUTO | 完整支持，语义变为“纵向排列”                         |

⚠️ **关键点**：

> 在 VerticalTabLayout 中，所有“width 语义”都被重新解释为 **height 语义**。

---

## 四、VerticalTabLayout.java 深度解析（实现层）

### 1. 类结构概览

```java
public class VerticalTabLayout extends ScrollView
```

核心设计点：

* 使用 `ScrollView` 替代 `HorizontalScrollView`
* 内部真正承载 Tab 的是 `SlidingTabIndicator`（LinearLayout）
* 所有滚动、动画 **全部改为 Y 轴方向**

---

### 2. 关键改造点总览

| 模块           | 官方 TabLayout         | VerticalTabLayout |
| ------------ | -------------------- | ----------------- |
| 容器           | HorizontalScrollView | ScrollView        |
| 滚动方向         | X 轴                  | Y 轴               |
| Indicator 移动 | left / right         | top / bottom      |
| Tab 尺寸       | width 主导             | height 主导         |

---

### 3. 指示器系统（Indicator）

#### 3.1 Gravity 定义

```java
INDICATOR_GRAVITY_LEFT
INDICATOR_GRAVITY_CENTER
INDICATOR_GRAVITY_RIGHT
INDICATOR_GRAVITY_STRETCH
```

对应行为：

* **LEFT / CENTER / RIGHT**：

  * 使用 indicator drawable 的 intrinsicWidth
* **STRETCH**：

  * 忽略 drawable 尺寸，填满整个 Tab

---

### 4. 滚动与动画机制（核心）

#### 4.1 滚动位置计算

```java
private int calculateScrollYForTab(int position, float offset)
```

计算逻辑：

1. 取当前 Tab 的中心点
2. 对齐到父容器中心
3. 根据 offset 插值到下一个 Tab

> 本质是 **“让选中 Tab 永远尽量居中显示”**

---

#### 4.2 动画实现

```java
ValueAnimator
scrollTo(0, animatedValue)
```

* 使用 `FAST_OUT_SLOW_IN_INTERPOLATOR`
* 所有动画均作用于 **ScrollView 的 Y 轴**

---

### 5. MODE 行为解析

| Mode            | 行为说明                         |
| --------------- | ---------------------------- |
| MODE_FIXED      | 所有 Tab 平分高度，不可滚动             |
| MODE_SCROLLABLE | 高度自适应，允许滚动                   |
| MODE_AUTO       | 高度够用时 FIXED，不够时自动 SCROLLABLE |

实现关键：

```java
lp.height = 0;
lp.weight = 1;
```

---

### 6. ViewPager 支持

完全复刻官方 TabLayout 行为：

* `setupWithViewPager()`
* 自动监听 Adapter 变化
* 支持 PageChangeListener

**唯一差异**：

> ViewPager 仍然是横向滑动，但 Tab 是纵向排列

---

## 五、使用示例

### 1. XML

```xml
<io.secf4ult.verticaltablayout.VerticalTabLayout
    android:layout_width="wrap_content"
    android:layout_height="match_parent"
    app:tabMode="scrollable"
    app:tabIndicatorGravity="stretch"
    app:tabIndicatorColor="@color/colorAccent" />
```

---

### 2. Java / Kotlin

```kotlin
verticalTabLayout.setupWithViewPager(viewPager)
```

或手动添加：

```kotlin
verticalTabLayout.addTab(
    verticalTabLayout.newTab().setText("Home")
)
```

---

## 六、适用场景

* 平板 / 横屏应用
* 设置页 / 分类页
* TV / 桌面模式 UI
* Material 风格侧边导航替代方案

---

## 七、设计总结

**VerticalTabLayout 的核心价值在于：**

* 不是“重写一个 TabLayout”
* 而是 **“把 TabLayout 旋转了 90°，并保证所有行为语义仍然成立”**

这使得它：

* 易维护
* 易升级 Material 版本
* 易与现有项目集成

---

## 八、License

基于原始 VerticalTabLayout 改造版本，遵循原项目 License。

---

