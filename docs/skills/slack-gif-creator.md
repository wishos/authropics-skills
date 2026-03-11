# Slack GIF 创建器 (Slack GIF Creator) 指南

## 触发条件

当用户请求为 Slack 创建动画 GIF 时使用此 Skill，如"为我制作一个 X 做 Y 的 GIF 用于 Slack"。提供约束、验证工具和动画概念的知识和实用程序。

---

## 概述

一个工具包，提供为 Slack 优化创建动画 GIF 的实用程序和知识。

---

## Slack 要求

**尺寸：**
- Emoji GIF：128x128（推荐）
- 消息 GIF：480x480

**参数：**
- FPS：10-30（越低文件越小）
- 颜色：48-128（越少文件越小）
- 持续时间：Emoji GIF 保持在 3 秒以下

---

## 核心工作流

```python
from core.gif_builder import GIFBuilder
from PIL import Image, ImageDraw

# 1. 创建构建器
builder = GIFBuilder(width=128, height=128, fps=10)

# 2. 生成帧
for i in range(12):
    frame = Image.new('RGB', (128, 128), (240, 248, 255))
    draw = ImageDraw.Draw(frame)

    # 使用 PIL 原语绘制动画
    # （圆形、多边形、线条等）

    builder.add_frame(frame)

# 3. 保存并优化
builder.save('output.gif', num_colors=48, optimize_for_emoji=True)
```

---

## 绘制图形

### 处理用户上传的图像
如果用户上传图像，考虑他们是否想：
- **直接使用**（例如"动画这个"、"将其分成帧"）
- **用作灵感**（例如"像这个一样做点什么"）

使用 PIL 加载和处理图像：
```python
from PIL import Image

uploaded = Image.open('file.png')
# 直接使用，或者仅作为颜色/风格的参考
```

### 从头绘制
从头绘制图形时，使用 PIL ImageDraw 原语：

```python
from PIL import ImageDraw

draw = ImageDraw.Draw(frame)

# 圆形/椭圆
draw.ellipse([x1, y1, x2, y2], fill=(r, g, b), outline=(r, g, b), width=3)

# 星星、三角形、任何多边形
points = [(x1, y1), (x2, y2), (x3, y3), ...]
draw.polygon(points, fill=(r, g, b), outline=(r, g, b), width=3)

# 线条
draw.line([(x1, y1), (x2, y2)], fill=(r, g, b), width=5)

# 矩形
draw.rectangle([x1, y1, x2, y2], fill=(r, g, b), outline=(r, g, b), width=3)
```

**不要使用：** Emoji 字体（跨平台不可靠）或假设此技能中包含预打包图形。

### 让图形看起来更好

图形应该看起来精致和创意，而不是基本的。以下是方法：

**使用更粗的线条** - 始终为轮廓和线条设置 `width=2` 或更高。细线（width=1）看起来粗糙和业余。

**添加视觉深度：**
- 为背景使用渐变（`create_gradient_background`）
- 使用多层形状增加复杂性（例如，内部带有小星星的星星）

**让形状更有趣：**
- 不要只画一个普通圆——添加高光、环或图案
- 星星可以发光（在后面绘制更大、半透明版本）
- 组合多个形状（星星 + 闪烁、圆 + 环）

**注意颜色：**
- 使用充满活力、互补的颜色
- 添加对比（浅形状上的深轮廓、深形状上的浅轮廓）
- 考虑整体构图

**对于复杂形状**（心、雪花等）：
- 结合多边形和椭圆的组合
- 小心计算对称点
- 添加细节（心可以有高光曲线、雪花有复杂的分支）

有创意和详细！一个好的 Slack GIF 应该看起来精致，而不是占位符图形。

---

## 可用实用程序

### GIFBuilder (`core.gif_builder`)
组装帧并为 Slack 优化：
```python
builder = GIFBuilder(width=128, height=128, fps=10)
builder.add_frame(frame)  # 添加 PIL Image
builder.add_frames(frames)  # 添加帧列表
builder.save('out.gif', num_colors=48, optimize_for_emoji=True, remove_duplicates=True)
```

### 验证器 (`core.validators`)
检查 GIF 是否满足 Slack 要求：
```python
from core.validators import validate_gif, is_slack_ready

# 详细验证
passes, info = validate_gif('my.gif', is_emoji=True, verbose=True)

# 快速检查
if is_slack_ready('my.gif'):
    print("Ready!")
```

### 缓动函数 (`core.easing`)
平滑运动而不是线性：
```python
from core.easing import interpolate

# 从 0.0 到 1.0 的进度
t = i / (num_frames - 1)

# 应用缓动
y = interpolate(start=0, end=400, t=t, easing='ease_out')

# 可用：linear, ease_in, ease_out, ease_in_out,
#           bounce_out, elastic_out, back_out
```

### 帧辅助 (`core.frame_composer`)
常用需求的便捷函数：
```python
from core.frame_composer import (
    create_blank_frame,         # 纯色背景
    create_gradient_background,  # 垂直渐变
    draw_circle,                # 圆形辅助
    draw_text,                 # 简单文本渲染
    draw_star                   # 五角星
)
```

---

## 动画概念

### 摇动/振动
使用振荡偏移对象位置：
- 使用 `math.sin()` 或 `math.cos()` 与帧索引
- 添加小随机变化以获得自然感觉
- 应用于 x 和/或 y 位置

### 脉冲/心跳
节奏性缩放对象大小：
- 使用 `math.sin(t * frequency * 2 * math.pi)` 进行平滑脉冲
- 对于心跳：两个快速脉冲然后暂停（调整正弦波）
- 在基本大小的 0.8 到 1.2 之间缩放

### 弹跳
对象下落和弹跳：
- 落地使用 `interpolate()` 和 `easing='bounce_out'`
- 下落使用 `easing='ease_in'`
- 通过每帧增加 y 速度来应用重力

### 旋转/旋转
绕中心旋转对象：
- PIL：`image.rotate(angle, resample=Image.BICUBIC)`
- 对于摆动：使用正弦波代替线性角度

### 淡入/淡出
逐渐出现或消失：
- 创建 RGBA 图像，调整 alpha 通道
- 或使用 `Image.blend(image1, image2, alpha)`
- 淡入：alpha 从 0 到 1
- 淡出：alpha 从 1 到 0

### 滑动
将对象从屏幕外移动到位置：
- 起始位置：画布边界外
- 结束位置：目标位置
- 使用 `interpolate()` 和 `easing='ease_out'` 进行平滑停止
- 对于超调：使用 `easing='back_out'`

### 缩放
缩放和定位以获得缩放效果：
- 放大：从 0.1 缩放到 2.0，裁剪中心
- 缩小：从 2.0 缩放到 1.0
- 可以添加运动模糊增加戏剧效果（PIL 滤镜）

### 爆炸/粒子爆发
创建向外辐射的粒子：
- 生成具有随机角度和速度的粒子
- 更新每个粒子：`x += vx`, `y += vy`
- 添加重力：`vy += gravity_constant`
- 随时间淡出粒子（减少 alpha）

---

## 优化策略

只有当要求使文件更小时，实现以下几种方法：

1. **更少帧** - 降低 FPS（10 而不是 20）或更短持续时间
2. **更少颜色** - `num_colors=48` 而不是 128
3. **更小尺寸** - 128x128 而不是 480x480
4. **删除重复帧** - `remove_duplicates=True` 保存时
5. **Emoji 模式** - `optimize_for_emoji=True` 自动优化

```python
# Emoji 最大优化
builder.save(
    'emoji.gif',
    num_colors=48,
    optimize_for_emoji=True,
    remove_duplicates=True
)
```

---

## 哲学

此技能提供：
- **知识**：Slack 要求和动画概念
- **实用程序**：GIFBuilder、验证器、缓动函数
- **灵活性**：使用 PIL 原语创建动画逻辑

它不提供：
- 僵硬的动画模板或预制函数
- Emoji 字体渲染（跨平台不可靠）
- 预打包图形库内置于技能中

**关于用户上传的注意**：此技能不包含预建图形，但如果用户上传图像，使用 PIL 加载和处理它——根据他们的请求解释他们是想直接使用还是仅用作灵感。

有创意！组合概念（弹跳 + 旋转、脉冲 + 滑动等）并使用 PIL 的全部功能。
