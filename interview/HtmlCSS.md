## HTML


## CSS
**标准盒模型和 IE 盒模型**

标准盒模型的 width 和 hight 指的是内容区域的宽高。

IE 盒模型的 width 和 hight 指的是内容区域的宽高 + padding + border。

**BFC **

BFC（Block Formatting Context）直译为“块级格式化上下文”。

BFC 是 W3C CSS 2.1 规范中的一个概念，它决定了元素如何对其内容进行定位，以及与其他元素的关系和相互作用。

当涉及到可视化布局的时候，Block Formatting Context提供了一个环境，HTML元素在这个环境中按照一定规则进行布局。

如何形成 BFC ？

+ float 的值不能为 none
+ overflow 的值不能为 visible
+ display 的值为 table-cell, table-caption, inline-block 中的任何一个
+ position 的值不为 relative 和 static 

**BFC的约束规则**

+ 内部的 Box 会在垂直方向上一个接一个的放置
+ 垂直方向的距离有 margin 决定(属于同一个 BFC 的两个相邻 Box 的 margin 会发生重叠，与方向无关)
+ 每个元素的 margin box 的左边， 与包含块 border box 的左边相接触(对于从左往右的格式化，否则相反)。
+ 使存在浮动也是如此
+ BFC 的区域不会与 float 的元素区域重叠
+ 计算 BFC 的高度时，浮动子元素也参与计算
+ BFC 就是页面上的一个隔离的独立容器，容器里面的子元素不会影响到外面元素，反之亦然

**BFC 的作用**

1. 不和浮动元素重叠
2. 清除元素内部浮动
3. 防止垂直 margin 重叠