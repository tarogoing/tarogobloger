[toc]
在书写数值计算类文章，特别是机器学习相关算法时，难免需要插入复杂的数学公式。一种是用图片在网页上展示，另外一种是使用 [MathJax](https://www.mathjax.org/) 来展示复杂的数学公式。它直接使用 Javascript 使用矢量字库或 SVG 文件来显示数学公式。优点是效果好，比如在 Retina 屏幕上也不会变得模糊。并且可以直接把公式写在 Markdown 文章里。本文介绍使用 MathJax 在 Markdown 文件里直接插入数学公式。并且附带一个简单的书写数学公式的 LaTex 教程。

## <a id="t0"></a><a id="t0"></a>工具

### <a id="t1"></a><a id="t1"></a>配置 Markdown Preview 来支持 MathJax

使用 Sublime + Markdown Preview 插件来写博客时。需要开启 Markdown Preview 对 MathJax 的支持，这样在预览界面才能正确地显示数学公式。方法是打开在 Markdown Preview 的用户配置文件 (Package Settings -> Markdown Preview -> Setting - User) 里添加如下内容：

```
"enable_mathjax": true
```

### <a id="t2"></a><a id="t2"></a>配置 Pelican 主题模板来支持 MathJax

我使用的主题是 `foundation-default-colours`，它默认是支持 MathJax 的。我们可以在模板 `base.html` 找到如下内容：

```

<script type="text/x-mathjax-config">
MathJax.Hub.Config({
    jax: ["input/TeX", "output/HTML-CSS"],
    tex2jax: {
        inlineMath: [ ['$', '$'] ],
        displayMath: [ ['$$', '$$']],
        processEscapes: true,
        skipTags: ['script', 'noscript', 'style', 'textarea', 'pre', 'code']
    },
    messageStyle: "none",
    "HTML-CSS": { preferredFont: "TeX", availableFonts: ["STIX","TeX"] }
});
</script>
<script type="text/javascript" src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML"></script>
```

如果模板不支持，也可以直接添加上面的脚本来支持 MathJax。

## <a id="t3"></a><a id="t3"></a>LaTex 简明教程

### <a id="t4"></a><a id="t4"></a>例子

先来看个例子：

```
$$
J(\theta) = \frac 1 2 \sum_{i=1}^m (h_\theta(x^{(i)})-y^{(i)})^2
$$
```

上面用 LaTex 格式书写的数学公式经过 MathJax 展示后效果如下：

<a id="MathJax-Element-1-Frame"></a><a id="MathJax-Span-1"></a><a id="MathJax-Span-2"></a><a id="MathJax-Span-3"></a>J<a id="MathJax-Span-4"></a>(<a id="MathJax-Span-5"></a>θ<a id="MathJax-Span-6"></a>)<a id="MathJax-Span-7"></a>=<a id="MathJax-Span-8"></a><a id="MathJax-Span-9"></a>1<a id="MathJax-Span-10"></a>2<a id="MathJax-Span-11"></a><a id="MathJax-Span-12"></a>∑<a id="MathJax-Span-13"></a><a id="MathJax-Span-14"></a><a id="MathJax-Span-15"></a>i<a id="MathJax-Span-16"></a>=<a id="MathJax-Span-17"></a>1<a id="MathJax-Span-18"></a>m<a id="MathJax-Span-19"></a>(<a id="MathJax-Span-20"></a><a id="MathJax-Span-21"></a>h<a id="MathJax-Span-22"></a>θ<a id="MathJax-Span-23"></a>(<a id="MathJax-Span-24"></a><a id="MathJax-Span-25"></a>x<a id="MathJax-Span-26"></a><a id="MathJax-Span-27"></a><a id="MathJax-Span-28"></a>(<a id="MathJax-Span-29"></a>i<a id="MathJax-Span-30"></a>)<a id="MathJax-Span-31"></a>)<a id="MathJax-Span-32"></a>−<a id="MathJax-Span-33"></a><a id="MathJax-Span-34"></a>y<a id="MathJax-Span-35"></a><a id="MathJax-Span-36"></a><a id="MathJax-Span-37"></a>(<a id="MathJax-Span-38"></a>i<a id="MathJax-Span-39"></a>)<a id="MathJax-Span-40"></a><a id="MathJax-Span-41"></a>)<a id="MathJax-Span-42"></a>2J(θ)=12∑i=1m(hθ(x(i))−y(i))2

这个公式是线性回归算法里的成本函数。

### <a id="t5"></a><a id="t5"></a>规则

关于在 Markdown 书写 LaTex 数学公式有几个规则常用规则需要记住：

**行内公式**  
行内公式使用 `$` 号作为公式的左右边界，如 <a id="MathJax-Element-2-Frame"></a><a id="MathJax-Span-43"></a><a id="MathJax-Span-44"></a><a id="MathJax-Span-45"></a>h<a id="MathJax-Span-46"></a>(<a id="MathJax-Span-47"></a>x<a id="MathJax-Span-48"></a>)<a id="MathJax-Span-49"></a>=<a id="MathJax-Span-50"></a><a id="MathJax-Span-51"></a>θ<a id="MathJax-Span-52"></a>0<a id="MathJax-Span-53"></a>+<a id="MathJax-Span-54"></a><a id="MathJax-Span-55"></a>θ<a id="MathJax-Span-56"></a>1<a id="MathJax-Span-57"></a>xh(x)=θ0+θ1x 公式的 LaTex 内容如下

```
$h(x) = \theta_0 + \theta_1 x$
```

**行间公式**  
公式需要独立显示一行时，使用 `$$` 来作为公式的左右边界，如

<a id="MathJax-Element-3-Frame"></a><a id="MathJax-Span-58"></a><a id="MathJax-Span-59"></a><a id="MathJax-Span-60"></a><a id="MathJax-Span-61"></a>θ<a id="MathJax-Span-62"></a>i<a id="MathJax-Span-63"></a>=<a id="MathJax-Span-64"></a><a id="MathJax-Span-65"></a>θ<a id="MathJax-Span-66"></a>i<a id="MathJax-Span-67"></a>−<a id="MathJax-Span-68"></a>α<a id="MathJax-Span-69"></a><a id="MathJax-Span-70"></a>∂<a id="MathJax-Span-71"></a><a id="MathJax-Span-72"></a>∂<a id="MathJax-Span-73"></a><a id="MathJax-Span-74"></a>θ<a id="MathJax-Span-75"></a>i<a id="MathJax-Span-76"></a>J<a id="MathJax-Span-77"></a>(<a id="MathJax-Span-78"></a>θ<a id="MathJax-Span-79"></a>)θi=θi−α∂∂θiJ(θ)

的 LaTex 代码为：

```
$$
\theta_i = \theta_i - \alpha\frac\partial{\partial\theta_i}J(\theta)
$$
```

**常用 LaTex 代码**  
需要记住的几个常用的符号，这样书写起来会快一点

| 编码  | 说明  | 示例  |
| --- | --- | --- |
| \\frac | 分子分母之间的横线 | <a id="MathJax-Element-4-Frame"></a><a id="MathJax-Span-80"></a><a id="MathJax-Span-81"></a><a id="MathJax-Span-82"></a><a id="MathJax-Span-83"></a>1<a id="MathJax-Span-84"></a>x1x |
| _   | 用下划线来表示下标 | <a id="MathJax-Element-5-Frame"></a><a id="MathJax-Span-85"></a><a id="MathJax-Span-86"></a><a id="MathJax-Span-87"></a><a id="MathJax-Span-88"></a>x<a id="MathJax-Span-89"></a>ixi |
| ^   | 次方运算符来表示上标 | <a id="MathJax-Element-6-Frame"></a><a id="MathJax-Span-90"></a><a id="MathJax-Span-91"></a><a id="MathJax-Span-92"></a><a id="MathJax-Span-93"></a>x<a id="MathJax-Span-94"></a>ixi |
| \\sum | 累加器，上下标用上面介绍的编码来书写 | <a id="MathJax-Element-7-Frame"></a><a id="MathJax-Span-95"></a><a id="MathJax-Span-96"></a><a id="MathJax-Span-97"></a>∑∑ |
| \\alpha | 希腊字母 alpha | <a id="MathJax-Element-8-Frame"></a><a id="MathJax-Span-98"></a><a id="MathJax-Span-99"></a><a id="MathJax-Span-100"></a>y<a id="MathJax-Span-101"></a>:=<a id="MathJax-Span-102"></a>α<a id="MathJax-Span-103"></a>xy:=αx |

记住这几个就差不多了，倒回去看一下线性回归算法的成本函数的公式及其 LaTex 代码，对着练习个10分钟基本就可以掌握常用公式的写法了。要特别注意公式里空格和 `{}` 的运用规则。基本原则是，空格可加可不加，但如果会引起歧义，最好加上空格。`{}` 是用来组成群组的。比如写一个分式时，分母是一个复杂公式时，可以用 `{}` 包含起来，这样整个复杂公式都会变成分母了。

### <a id="t6"></a><a id="t6"></a>几个非常有用的资源

*   Github 上有个[在线 Markdown MathJax 编辑器](https://kerzol.github.io/markdown-mathjax/editor.html)，可以在这里练习，平时写公式时也可以在这里先写好再拷贝到文章里
*   这是 [LaTex 完整教程](http://www.forkosh.com/mathtextutorial.html)，包含完整的 LaTex 数学公式的内容，包括更高级的格式控制等
*   这是一份PDF 格式的 [MathJax 支持的数学符号表](http://mirrors.ctan.org/info/symbols/math/maths-symbols.pdf)，当需要书写复杂数学公式时，一些非常特殊的符号的转义字符可以从这里查到

好啦，这样差不多就可以写出优美的数学公式啦。