---
title: "用代码折纸鹤(示例)"
date: 2026-09-03
description: "展示代码块的排版:一段 Python、一段 CSS,外加行内代码的样子。"
categories: ["技术"]
tags: ["示例", "python", "css"]
---

*示例文章。*

写博客和折纸有点像:给定一张纸,能折出的东西无穷多。下面展示代码排版。

## Python

```python
def fold(box, idea):
    """把想法折进盒子里。"""
    box.ideas.append(idea)
    return f"现在盒子里有 {len(box.ideas)} 个想法了"

box = {"ideas": []}
print(fold(box, "纸盒"))
```

## CSS

```css
:root {
  --bg: #F3E7CE; /* 纸盒内壁的米黄 */
  --ink: #46352A; /* 深棕,像牛皮纸上的字 */
}
```

## 行内代码

比如给元素加 `class="pill"`,或者配置里写 `mainSections = ["article"]`。行内代码底色偏暖,不刺眼。
