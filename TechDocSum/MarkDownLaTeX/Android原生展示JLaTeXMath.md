[toc]
### JLaTeXMath

1.JLaTeXMath是一个Java库。它的主要功能是显示LaTeX编写的数学公式，是最好的显示LaTeX代码的Java库。
2.JLaTeXMath曾经被许多著名且重要的项目引用，例如 Scilab ,Freeplane,Geogebra,Mathpiper 等等
3.JLaTeXMath的默认编码格式是UTF－8
源码的地址在http://forge.scilab.org/index.php/p/jlatexmath/
github也有其源码 https://github.com/opencollab/jlatexmath
源码从github克隆下来的，之后用IntelliJ IDEA 打开项目。
结构大概就是这样的：


### JLaTeXMath项目结构
其中JLaTeXMath是最重要最核心的
在test目录下找到ExamplesTest.java，运行单元测试，结果就出现在target目录里面了。so easy！
原生的latex的数学公式
\(\begin{split}{S_n} &= {a_1} + {a_2} + {a_3} + \cdot \cdot \cdot +{a_n} \\&= \left( {1 - \dfrac{1}{2}} \right) + \left( {\dfrac{1}{2} - \dfrac{1}{3}} \right) + \cdot \cdot \cdot + \left( {\dfrac{1}{n} - \dfrac{1}{n + 1}} \right)\\ &= 1 - \dfrac{1}{n + 1}.\end{split}\) \\

结果


### 结果:android 中怎么应用？

JLaTeXMath是一个标准的Java库，而android使用的Java是没有Swing组件的，所以单纯的放进去肯定是运行不起来的。
目前这样处理的：
1.把Swing相关的东西改写成Android的GUI组件。
2.利用这个库将Latex生成Bitmap。(这一块感觉可以做一个缓存)
3.利用SpannableString将bitmap和文字一块显示出来
最终效果


android的效果
参考连接
https://github.com/cyuanyang/widgetKit/tree/master/javamathview
https://github.com/cyuanyang/jlatexmath-android


作者：shawn_yy
链接：https://www.jianshu.com/p/c38637d6123c
