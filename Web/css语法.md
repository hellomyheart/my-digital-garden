# css语法

## CSS 实例

CSS 规则由两个主要的部分构成：选择器，以及一条或多条声明:

![](https://zeroone-bucket.oss-cn-beijing.aliyuncs.com/blog/20210406224152.gif)

选择器通常是您需要改变样式的 HTML 元素。

每条声明由一个属性和一个值组成。

属性（property）是您希望设置的样式属性（style attribute）。每个属性有一个值。属性和值被冒号分开。

------

## CSS 实例

CSS 声明总是以分号 (` ;` ) 结束，声明组以大括号 (`{ }`) 括起来:

```css
p {color:red;text-align:center;}
```

为了让 CSS 可读性更强，你可以每行只描述一个属性:

实例

```css
p
{color:red;
text-align:center;}
```

### CSS 颜色值的写法

在描述颜色的时候，除了使用英文单词 red，我们还可以使用十六进制的颜色值 #ff0000： 

```css
p { color: #ff0000; }
```

为了节约字节，我们可以使用 CSS 的缩写形式： 

```css
p { color: #f00; }
```

我们还可以通过两种方法使用 RGB 值：

```css
p { color: rgb(255,0,0); } 
p { color: rgb(100%,0%,0%); }css
```

**提示：**当使用 RGB 百分比时，即使当值为 0 时也要写百分比符号。但是在其他的情况下就不需要这么做了。比如说，当尺寸为 0 像素时，0 之后不需要使用 px 单位。

在本站的编程实战部分中，介绍了[CSS使用十六进制代码设置颜色](https://www.w3cschool.cn/codecamp/use-hex-code-for-specific-colors.html)，以及[CSS使用rgb属性设置颜色](https://www.w3cschool.cn/codecamp/use-rgb-values-to-color-elements.html)，你可以参考该部分的内容进行更多了解。

------

## CSS 注释

注释是用来解释你的代码，并且可以随意编辑它，浏览器会忽略它。

CSS注释以 "`/*`" 开始, 以 "`*/`" 结束, 实例如下:

```css
/*这是个注释*/        

p        

{       

text-align:center;       

/*这是另一个注释*/     

color:black;     

font-family:arial;       

}
```

