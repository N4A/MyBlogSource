---
title: 使用 Typora 编辑文档
date: 2016-10-30 15:42:58
tags: [Latex, markdown]
---



Typora是一款简约但功能强大文档编辑工具，支持导出pdf，md，html等多种格式文本，支持markdown语法和LaTex数学表达式语法。这里着重介绍LaTex的语法。参考[MathJax basic tutorial and quick reference](http://meta.math.stackexchange.com/questions/5020/mathjax-basic-tutorial-and-quick-reference/5044%20MathJax%20basic%20tutorial%20and%20quick%20reference)

### 基本语法
1. 行内表达式写在**\$...\$**之间，行间表达式写在**\$\$...\$\$**之间。
2. 使用**\\**表示转义，例如结合前面的行内表达式可得**\$\alpha \in A\$**表示**$\alpha \in A$**
3. 行间表达式**\$\$\sum_{i=0}^n i^2 = \frac{(n^2+n)(2n+1)}{6}\$\$**表示 
   $$
   \sum_{i=0}^n i^2 = \frac{(n^2+n)(2n+1)}{6} \tag {1}
   $$






<!-- more -->

### 其它表达式

在表达式上右键可查看语法等信息

$$
\mathbf{V}_1 \times \mathbf{V}_2 =  \begin{vmatrix} 
\mathbf{i} & \mathbf{j} & \mathbf{k} \\\\
\frac{\partial X}{\partial u} &  \frac{\partial Y}{\partial u} & 0 \\\\
\frac{\partial X}{\partial v} &  \frac{\partial Y}{\partial v} & 0 \\\\
\end{vmatrix} \tag{2}
$$

$$
\begin{align}
\sqrt{37} & = \sqrt{\frac{73^2-1}{12^2}} \\\\
& = \sqrt{\frac{73^2}{12^2}\cdot\frac{73^2-1}{73^2}} \\\\ \\\\
& = \sqrt{\frac{73^2}{12^2}}\sqrt{\frac{73^2-1}{73^2}} \\\\
& = \frac{73}{12}\sqrt{1 - \frac{1}{73^2}} \\\\ \\\\
& \approx \frac{73}{12}\left(1 - \frac{1}{2\cdot73^2}\right)
\end{align} \tag{3}
$$






$$
\begin{array}{c|lcr}
n & \text{Left} & \text{Center} & \text{Right} \\\\
\hline
1 & 0.24 & 1 & 125 \\\\
2 & -1 & 189 & -8 \\\\
3 & -20 & 2000 & 1+10i
\end{array} \tag{5}
$$

$$
% outer vertical array of arrays
\begin{array}{c}
% inner horizontal array of arrays
\begin{array}{cc}
% inner array of minimum values
\begin{array}{c|cccc}
\text{min} & 0 & 1 & 2 & 3\\\\
\hline
0 & 0 & 0 & 0 & 0\\\\
1 & 0 & 1 & 1 & 1\\\\
2 & 0 & 1 & 2 & 2\\\\
3 & 0 & 1 & 2 & 3
\end{array}
&
% inner array of maximum values
\begin{array}{c|cccc}
\text{max}&0&1&2&3\\\\
\hline
0 & 0 & 1 & 2 & 3\\\\
1 & 1 & 1 & 2 & 3\\\\
2 & 2 & 2 & 2 & 3\\\\
3 & 3 & 3 & 3 & 3
\end{array}
\end{array}
\\\\
% inner array of delta values
\begin{array}{c|cccc}
\Delta&0&1&2&3\\\\
\hline
0 & 0 & 1 & 2 & 3\\\\
1 & 1 & 0 & 1 & 2\\\\
2 & 2 & 1 & 0 & 1\\\\
3 & 3 & 2 & 1 & 0
\end{array}
\end{array} \tag{6}
$$


$$
\require{enclose}\begin{array}{rl}
\verb|\enclose{horizontalstrike}{x+y}| & \enclose{horizontalstrike}{x+y}\\\\
\verb|\enclose{verticalstrike}{\frac xy}| & \enclose{verticalstrike}{\frac xy}\\\\
\verb|\enclose{updiagonalstrike}{x+y}| & \enclose{updiagonalstrike}{x+y}\\\\
\verb|\enclose{downdiagonalstrike}{x+y}| & \enclose{downdiagonalstrike}{x+y}\\\\
\verb|\enclose{horizontalstrike,updiagonalstrike}{x+y}| & \enclose{horizontalstrike,updiagonalstrike}{x+y}\\\\
\end{array}\tag{7}
$$



详细请参考[MathJax basic tutorial and quick reference](http://meta.math.stackexchange.com/questions/5020/mathjax-basic-tutorial-and-quick-reference/5044%20MathJax%20basic%20tutorial%20and%20quick%20reference),有兴趣也可以自己编辑一些有趣的表达式。

   ### 数学符号识别工具

   利用Detexify这款工具，在方框中手写数学符号，右边会给出对应的latex语法。

   ![Detexify](/img/detexify.PNG)

   网址[Detexify](http://detexify.kirelabs.org/classify.html)

   