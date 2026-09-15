---
name: svg-cleaner
description: 清洗和标准化 SVG 图标源码，移除设计工具、iconfont、Figma、Sketch 等生成的冗余元数据、无用元素、固定尺寸与固定颜色，并在不改变路径数据和视觉形状的前提下输出支持 currentColor 的前端可用 SVG。用户粘贴 SVG、要求压缩或清理 SVG、统一项目图标、移除 fill/stroke 固定色、让图标跟随 CSS color、处理 iconfont 导出代码时，应使用此技能。
---

# SVG Cleaner

当用户提供 SVG 源代码时，严格按照以下规则清洗，输出干净、规范、可直接用于前端项目的 SVG。

## 核心目标

- 删除设计工具或 iconfont 产生的垃圾元数据和无用结构。
- 将固定颜色改为 `currentColor`，使图标可通过 CSS `color` 控制。
- 保持视觉形状完全不变：不修改路径数据、不合并路径、不改变精度。
- 输出体积小且易维护的 SVG。

## 清洗规则

按以下优先级执行。

### 1. 删除声明与命名空间冗余

- 删除 `<?xml ... ?>` 声明。
- 删除 `<!DOCTYPE ...>`。
- 删除 `version="1.1"` 属性。
- 未使用 `xlink:href` 或 `xlink:title` 时，删除 `xmlns:xlink` 属性。

### 2. 删除设计工具生成的垃圾元素

- 删除 `<defs>` 及其所有子元素，除非其中包含图标实际依赖的渐变或滤镜。
- 删除 `<clipPath>` 及所有 `clip-path` 属性。
- 仅当 `<g>` 没有 `transform` 或其他必要属性时，删除这一无意义包装，同时保留其子元素及原有顺序。
- 删除 `fill-opacity="1"`、`stroke-opacity="1"` 等默认值属性。
- 删除 `style="mix-blend-mode:passthrough"` 以及任何 `mix-blend-mode` 声明。
- 删除 `class`、`p-id`、`t` 等第三方元数据属性。

### 3. 处理固定尺寸

- 默认删除 `width` 和 `height`，让尺寸由外部 CSS 控制。
- 保留原始 `viewBox`，不得删除或改写其值。规范的图标通常使用 `0 0 <width> <height>` 格式。
- 仅当用户明确要求保留尺寸时，才添加 `width="1em" height="1em"`。
- 如果没有 `viewBox`，但存在可解析的数值型 `width` 和 `height`，先生成 `viewBox="0 0 <width> <height>"`，再删除固定尺寸。
- 如果 `viewBox`、`width` 和 `height` 都不可用，向用户询问缺失的画布尺寸，不猜测坐标范围。

### 4. 处理颜色

- 找出 `fill` 和 `stroke` 属性中的固定颜色值，例如 `#FFFFFF`、`#333333`、`#2475FF`、`rgb(...)` 或具名颜色，并替换为 `currentColor`。
- 保留语义值，例如 `none`、`currentColor`、`inherit`、`transparent` 和 `url(...)` 引用。
- 对纯单色填充图标，在 `<svg>` 根节点设置 `fill="currentColor"`，删除子元素上等价的 `fill` 属性。
- 对以描边为主的图标，在 `<svg>` 根节点设置 `stroke="currentColor"` 并确保 `fill="none"`；保留必要的 `stroke-width`、`stroke-linecap`、`stroke-linejoin`、`stroke-miterlimit` 等描边属性。
- 删除转换后不再需要的 `fill-opacity` 和 `stroke-opacity` 属性。
- 如果存在多个不同的固定颜色或明显的背景层，先判断它是否为多色图标：
  - 对明确不应跟随主题的彩色背景，保留背景固定色，仅将前景颜色改为 `currentColor`。
  - 如无法可靠区分背景与前景，保留必要颜色，输出可安全完成的清洗结果，并用一句话说明歧义和建议；只有无法产出安全结果时才询问用户。

### 5. 无障碍

- 在 `<svg>` 根节点添加 `aria-hidden="true"`，将图标视为纯装饰内容。
- 如果原 SVG 已包含明确的可访问名称或用户要求保留可访问语义，不要强行改为装饰图标；保留相关 `title`、`aria-label` 或 `aria-labelledby`。

### 6. 精度与路径数据

- 严禁修改任何 `d` 属性的值，包括数字、精度、命令字母、空格和换行。
- 严禁合并路径、简化路径、重新排序元素或改变坐标。
- 对路径之外的普通标记，只能删除不影响渲染的无意义空白。

### 7. 格式化输出

- 使用 2 空格缩进。
- 将 `<svg>` 根节点的属性逐行排列。
- 子元素属性较少时可保持单行；长 `path` 的 `d` 属性保持单行，不强制换行。
- 只输出最终结果，并包裹在语言标识为 `svg` 的 Markdown 代码块中。
- 除非存在歧义、风险或无法自动处理的结构，否则不添加额外解释。

## 特殊情况

### `<style>` 块

内联样式可能覆盖元素属性并阻止主题色生效。若样式规则能明确、安全地转换，则转换相关颜色；否则保留原样，并简洁提示用户需要人工确认。不要猜测选择器语义。

### 渐变与滤镜

- 保留实际被引用的渐变或滤镜定义及其所在 `<defs>`。
- 只删除未被引用的定义、`clipPath` 和无用包装。
- 谨慎处理渐变色标：只有在图标明确应整体跟随主题色时，才将固定色改为 `currentColor`；否则保留原色并说明。

### 引用与依赖

删除任何元素或属性前，检查 `url(#...)`、`href`、`xlink:href`、`aria-labelledby` 等引用，避免留下悬空引用或误删真实依赖。

## 示例

输入：

```svg
<?xml version="1.0" standalone="no"?>
<!DOCTYPE svg>
<svg t="123456" class="icon" viewBox="0 0 24 24" width="24" height="24" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink">
  <defs><clipPath id="a"><rect x="0" y="0" width="24" height="24"/></clipPath></defs>
  <g clip-path="url(#a)">
    <path fill="#333333" d="M12 2a10 10 0 1 0 0 20 10 10 0 0 0 0-20z"/>
    <path fill="#333333" d="M12 6v6l4 2"/>
  </g>
</svg>
```

输出：

```svg
<svg
  xmlns="http://www.w3.org/2000/svg"
  viewBox="0 0 24 24"
  fill="currentColor"
  aria-hidden="true"
>
  <path d="M12 2a10 10 0 1 0 0 20 10 10 0 0 0 0-20z"/>
  <path d="M12 6v6l4 2"/>
</svg>
```

## 禁止事项

- 不删除或改写已有 `viewBox`。
- 不修改 `d` 属性中的任何内容。
- 不合并路径、不改变元素顺序。
- 不自动添加 `width="1em" height="1em"`。
- 不删除被渐变、滤镜或其他渲染结构实际依赖的 `<defs>` 内容。
- 不保留可安全主题化的固定 `fill` 或 `stroke` 颜色。
- 不在可以安全清洗时询问用户是否继续。

## 交互方式

- 用户直接粘贴 SVG 时，立即输出清洗后的代码。
- 遇到多色、渐变、滤镜、样式表或引用关系时，优先输出不破坏视觉效果的保守清洗结果，并附一条简洁说明。
- 只有缺少画布信息或任何自动选择都可能改变视觉效果时，才提出一个最小化澄清问题。
