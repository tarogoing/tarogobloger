[toc]
# Markdown 添加 Latex 数学公式

[<img width="46" height="46" src="../_resources/10-e691107df16746d4a9f3fe9496fd1_1cc5467c7b5a449ca.jpg"/>](https://www.jianshu.com/u/a3e2567648b3)

[destiny0904](https://www.jianshu.com/u/a3e2567648b3)

2017.12.09 21:05:21字数 1,520阅读 2,090

# Markdown 中添加公式

行内公式

```ruby
$行内公式$

```

行间公式

```ruby
$$
行间公式
$$

```

# Latex 数学公式语法

## 角标（上下标）

**上标**

```css
^{}

```

**下标**

```css
_{}

```

上下标命令用来放在需要插入上下标的地方，花括弧内为上下标的内容，当角标为单个字符时候，可以不使用花括号；如果角标为多字符或者多层次的时候，必须要使用花括号。

比如：

```undefined
x^2, x_1^2, x^{(n)}_{22}, ^{16}O^{2-}_{32}, x^{y^{z^a}}, x^{y_z}

```

$$  
x^2, x_1^2, x^{(n)}_{22}, {16}O{2-}_{32}, x{y{z^a}}, x^{y_z}  
$$

**使用文字作为角标**

**文字模式**

```css
\mbox{}

```

**改变文字大小的命令**

```undefined
\tiny

```

比如：

```csharp
\partial f_{\mbox{\tiny 极大值}}

```

$$  
\\partial f_{\\mbox{\\tiny 极大值}}  
$$

**强制改变角标大小或层次命令**

```undefined
\scriptstyle

```

比如：

```undefined
y_N, y_{_N}, y_{_{\scriptstyle N}}

```

$$  
y\_N, y\_{_N}, y_{_{\\scriptstyle N}}  
$$

第一种输出为正常输出，但输出效果不明显；第二种是将一级角标改为二级角标，字体也自动变为二级角标字体；第三种将一级角标改为二级角标，但是强制字体改为一级角标字体。

## 分式

**分式命令**

```undefined
\frac{分子}{分母}

```

比如：

```undefined
\frac{x+y}{y+z}

```
行内分式：$\frac{x+y}{y+z}$
行间分式：$\frac{x+y}{y+z}$

表明**行内分式字体比行间字体小**，因为行内分式使用的是角标字体  
可以人工改变行内分式的字体大小，

```undefined
\displaystyle\frac{x+y}{y+z}

```

$\displaystyle\frac{x+y}{y+z}$

**连分式**

```undefined
x_0+\frac{1}{x_1+\frac{1}{x_2+\frac{1}{x_3+\frac{1}{x_4}}}}

```

$$  
x\_0+\\frac{1}{x\_1+\\frac{1}{x\_2+\\frac{1}{x\_3+\\frac{1}{x_4}}}}  
$$

可以通过强制改变字体大小使得分子分母字体大小一致：

```bash
\newcommand{\FS}[2]{\displaystyle\frac{#1}{#2}}x0+\FS{1}{X_1+\FS{1}{X_2+\FS{1}{X_3+\FS{1}{X_4}}}}

```

$$  
\\newcommand{\\FS}\[2\]{\\displaystyle\\frac{#1}{#2}} x\_0+\\FS{1}{X\_1+\\FS{1}{X\_2+\\FS{1}{X\_3+\\FS{1}{X_4}}}}  
$$

第一行命令定义了一个新的分式命令，规定每个调用该命令的分式都按 `\displaystyle` 的格式显示分式

分数线长度值是预设为分子分母的最大长度，如果想要使分数线再长一点，可以在分子或分母两端添加一些间隔

```undefined
\frac{1}{2},\frac{\;1\;}{\;2\;}

```

$$  
\\frac{1}{2},\\frac{;1;}{;2;}  
$$

其中第一个显示是正常的显示，第二个显示是分子分母前后都放入一个间隔命令 `\;` 中

## 根式

**二次根式命令**

```cpp
\sqrt{表达式}

```

如果表达式是个单个字符，则不需要花括号，但需要在字符和 `sqrt` 之间加入一个空格

**$n$ 次根式命令**

```cpp
\sqrt[n]{表达式}

```

被开方表达式字符高度不一致时，根号上面的横线可能不是在同一条直线上

为了使横线在同一条直线上，可以在被开方表达式插入一个只有高度没有宽度的数学支柱 `\mathstut`

```cpp
\sqrt{a}+\sqrt{b}+\sqrt{c},\qquad \sqrt{\mathstrut a}+\sqrt{\mathstrut b}+\sqrt{\mathstrut c}

```

$$  
\\sqrt{a}+\\sqrt{b}+\\sqrt{c},\\qquad \\sqrt{\\mathstrut a}+\\sqrt{\\mathstrut b}+\\sqrt{\\mathstrut c}  
$$

当被开方表达式高时，开方次数的位置显得略低，解决方法为：将开方此时改为上标，并拉近与根式的水平距离，即显示将命令中的 `[n]` 改为 `[^n\!]`,其中 `^` 表示是上标，`\!` 表示缩小间隔

```cpp
\sqrt{1+\sqrt[^p\!]{1+\sqrt[^q\!]{1+a}}}

```

$$  
\\sqrt{1+\\sqrt\[p\]{1+\\sqrt\[q\]{1+a}}}  
$$

$$  
\\sqrt{1+\\sqrt\[p!\]{1+\\sqrt\[q!\]{1+a}}}  
$$  
注意比较两个根式开方次数的显示位置

## 求和与积分

**求和命令**

```undefined
\sum_{k=1}^n表达式（求和项紧随其后,下同）

```

**积分命令**

```undefined
\int_a^b表达式

```

比如：

无穷级数 $\\sum_{k=1}\\infty\\frac{xn}{n!}$ 可以化为积分 $\\int\_0^\\infty e^x$ 也即是：$\\sum\_{k=1}\\infty\\frac{xn}{n!} = \\int_0^\\infty e^x$

**改变上下限位置的命令**：

```undefined
\limits(强制上下限在上下侧) 
\nolimits(强制上下限在左右侧)

```

比如：

```undefined
\sum\limits_{k=1}^n
\sum\nolimits_{k=1}^n

```

$\\sum\\limits_{k=1}^n$ 和 $\\sum\\nolimits_{k=1}^n$

```cpp
\int\limits_0^\infty e^x
\int\nolimits_0^\infty e^x

```

$\\int\\limits\_0^\\infty e^x$ 和 $\\int\\nolimits\_0^\\infty e^x$

## 上、下划线

**上划线命令**

```undefined
\overline{公式}

```

**下划线命令**

```undefined
\underline{公式}

```

比如：

```undefined
\overline{\overline{a^2}+\underline{ab}+\bar{a}^3}

```

$$\\overline{\\overline{a2}+\\underline{ab}+\\bar{a}3}$$

## 上、下括弧

**上花括弧命令**

```undefined
\overbrace{公式}{说明}

```

**下花括弧命令**

```undefined
\underbrace{公式}_{说明}

```

比如：

```undefined
\underbrace{a+\overbrace{b+\dots+b}^{m\mbox{\tiny 个}}}_{20\mbox{\scriptsize 个}}

```

$$\\underbrace{a+\\overbrace{b+\\dots+b}^{m\\mbox{\\tiny 个}}}_{20\\mbox{\\scriptsize 个}}$$

## 数学重音符号

这里以 `a` 为例，如果是字母 `i` 或 `j` 带有重音，应该替换为`\imath`、`\jmath`

```undefined
\hat{a}
\check{a}
\breve{a}
\tilde{a}
\bar{a}
\vec{a}
\acute{a}
\grave{a}
\mathring{a}
\dot{a}
\ddot{a}

```

$$  
\\hat{a}  
\\check{a}  
\\breve{a}  
\\tilde{a}  
\\bar{a}  
\\vec{a}  
\\acute{a}  
\\grave{a}  
\\mathring{a}  
\\dot{a}  
\\ddot{a}  
$$

## 堆积符号

*   `\stacrel{上位符号}{基位符号}` 基位符号大，上位符号小
*   `{上位公式\atop 下位公式}` 上下符号一样大
*   `{上位公式\choose 下位公式}` 上下符号一样大；上下符号被包括在圆弧内

比如：

```ruby
\vec{x}\stackrel{\mathrm{def}}{=}{x_1,\dots,x_n}\\ {n+1 \choose k}={n \choose k}+{n \choose k-1}\\ \sum_{k_0,k_1,\ldots>0 \atop k_0+k_1+\cdots=n}A_{k_0}A_{k_1}\cdots 

```

$$  
\\vec{x}\\stackrel{\\mathrm{def}}{=}{x\_1,\\dots,x\_n}\ {n+1 \\choose k}={n \\choose k}+{n \\choose k-1}\ \\sum_{k\_0,k\_1,\\ldots>0 \\atop k\_0+k\_1+\\cdots=n}A_{k\_0}A\_{k_1}\\cdots  
$$

## 定界符

```undefined
（）\big(\big) \Big(\Big) \bigg(\bigg) \Bigg(\Bigg)
\big(\Big) \bigg(\Bigg)

```

$$  
（）\\big(\\big) \\Big(\\Big) \\bigg(\\bigg) \\Bigg(\\Bigg)  
\\big(\\Big) \\bigg(\\Bigg)  
$$

**自适应放大命令**

```swift
\left
\right

```

放在左右定界符前，自动随着公式内容大小调整符号大小

比如：

```swift
(x)\left(x^{y^z}\right)

```

$$  
(x)\\left(x{yz}\\right)  
$$

## 占位宽度

*   两个 quad 空格 `\qquad`， 两个 m 的宽度：$a \\qquad b$
*   一个 quad 空格 `\quad`， 一个m的宽度：$a \\quad b$
*   大空格 `\`， 1/3m 宽度：$a\ b$
*   中等空格 `\;` 2/7m 宽度：$a;b$
*   小空格 `\,` 1/6m 宽度：$a,b$
*   没有空格：$ab$
*   紧贴 `\!` ，缩进1/6m宽度：$a!b$

`\quad` 代表当前字体下接近字符‘M’的宽度（approximately the width of an "M" in the current font）

## 集合相关的运算命令

**集合的大括号**

```undefined
 \{ ...\}

```

$${ ...}$$

**属于**

```undefined
\in

```

$$\\in$$

**不属于**

```undefined
\not\in 

```

$$  
\\not\\in  
$$

**包含于**

```undefined
A\subset B

```

$$  
A\\subset B  
$$

**真包含于**

```undefined
A \subsetneqq B

```

$$  
A \\subsetneqq B  
$$

**包含**

```undefined
A \supset B

```

$$  
A \\supset B  
$$

**真包含**

```undefined
A \supsetneqq B

```

$$  
A \\supsetneqq B  
$$

**A不包含于B**

```undefined
A \not \subset B

```

$$  
A \\not \\subset B  
$$

**A交B**

```bash
A \cap B

```

$$  
A \\cap B  
$$

**A并B**

```undefined
A \cup B

```

$$  
A \\cup B  
$$

**A的闭包**

```undefined
\overline{A}

```

$$  
\\overline{A}  
$$

**A减去B**

```undefined
A\setminus B

```

$$  
A\\setminus B  
$$

**实数集合**

```undefined
\mathbb{R}

```

$$  
\\mathbb{R}  
$$

**空集**

```undefined
\emptyset

```

$$  
\\emptyset  
$$

# 参考

> 引自：[Markdown 添加 Latex 数学公式](https://link.jianshu.com?t=https://www.cnblogs.com/peaceWang/p/Markdown-tian-jia-Latex-shu-xue-gong-shi.html#toc_1)

