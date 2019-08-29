---
title: css实现内容区域背景半透明的圆角渐变边框
author: GFELF
date: 2019-08-29 22:19:05
tags: [css, 圆角渐变, border, 透明]
---

### 最近项目中遇到一个这样的需求，如下图红框部分，需要一个带圆角的渐变色边框，而且中间的文字部分是一个半透明背景，需要露出body的渐变背景色
---
<!-- more -->
{% asset_img 圆角渐变.jpg 圆角渐变边框 %}
* 首先想到的方案是使用border-image做渐变，但是border-image存在时，border-radius是不会生效的，因为：
    > A box's backgrounds, but not its border-image, are clipped to the appropriate curve (as determined by ‘background-clip’). Other effects that clip to the border or padding edge (such as ‘overflow’ other than ‘visible’) also must clip to the curve. The content of replaced elements is always trimmed to the content edge curve. Also, the area outside the curve of the border edge does not accept mouse events on behalf of the element.
* 然后考虑使用一个外层div或者伪元素设置渐变背景来模拟边框，然后内层div作为背景并留出border的宽度，但是这样比较繁琐，而且无法实现content区域半透明露出body的效果；
* 经过思考，最终使用了如下变通的方案，因为背景可以设置多个并各自设置不同的渲染方式，所以通过设置多个background-image和对应的background-attachment、background-clip，来模拟三层效果，分别对应：
  * 底层 - 模拟border
  * 中间层 - 模拟body背景
  * 上层 - 内容区域的半透明效果，露出中间层
    代码如下：

    ```css
    .dialog-right>.chat-bubble {
        border-radius: 1.6rem 1.6rem .2rem 1.6rem;
        border: .1rem solid transparent;
        background-image: linear-gradient(rgba(141, 185, 255, .2), rgba(141, 185, 255, .2)),linear-gradient(120deg, #635799 10%,#04065c 90%),
        linear-gradient(-30deg, #00ccff, #0049ff);
        background-attachment: scroll, fixed, scroll;
        background-clip: padding-box, padding-box, border-box;
        background-origin: border-box; 
    }
    ```
---
这个方案中的两个关键属性是background-attachment和background-clip。
1. background-attachment可以设置背景图是否固定或者随着页面的其余部分滚动，通过把body和目标区域的此属性都设置为fixed，保证两个区域的背景相对视口渲染，来达到中间层覆盖底层，模拟body背景的效果；
2. 而background-clip可以设置背景的绘制区域，这样就免去了多余的div或者伪元素的复杂度。
以下完整demo可以比较清楚地看到一、二、三层背景的效果以及backgroun-attachment的作用：
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
    <style>
        * {
            box-sizing: border-box;
        }
        body {
            background-image: linear-gradient(120deg, red, black);
            background-attachment: fixed;
            display: flex;
        }
        body>div {
            flex: 1;
        }
        .one-layer {
            border: 1px solid transparent;
            /* 无效 */
            border-radius: 10px;
            width: 100px;
            height: 100px;
            margin: 100px auto;
            border-image: linear-gradient(-30deg, black, white) 1;
            background-image: linear-gradient(120deg, red, black);
            /* 去掉下面属性即为默认的背景渲染方式 */
            background-attachment: fixed;
        }
        .two-layer {
            border: 1px solid transparent;
            border-radius: 10px;
            width: 100px;
            height: 100px;
            margin: 100px auto;
            background-image: linear-gradient(rgba(255, 255, 255, 0), rgba(255, 255, 255, 0)), linear-gradient(-30deg, black, white);
            background-attachment: scroll, scroll;
            background-clip: padding-box, border-box;
        }
        .three-layer {
            border: 1px solid transparent;
            border-radius: 10px;
            width: 100px;
            height: 100px;
            margin: 100px auto;
            background-image: linear-gradient(rgba(255, 255, 255, 0), rgba(255, 255, 255, 0)), linear-gradient(120deg, red, black), linear-gradient(-30deg, black, white);
            background-attachment: scroll, fixed, scroll;
            background-clip: padding-box, padding-box, border-box;
        }
    </style>
</head>
<body>
    <!-- 此时radius无效 -->
    <div>
        <div class="one-layer"></div>
        <div class="one-layer"></div>
        <div class="one-layer"></div>
    </div>
    <!-- 此时由于内容是透明色，会露出用于模拟渐变边框的背景 -->
    <div>
        <div class="two-layer"></div>
        <div class="two-layer"></div>
        <div class="two-layer"></div>
    </div>
    <!-- 加入中间层覆盖用于模拟边框的背景 -->
    <div>
        <div class="three-layer"></div>
        <div class="three-layer"></div>
        <div class="three-layer"></div>
    </div>
</body>
</html>
```