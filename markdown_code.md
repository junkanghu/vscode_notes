# Markdown Notes
## markdown
[TOC]
1. 标题
   1. 一级标题：#text or 写完text(Markdown Notes)后在下面加三个===
   2. 二级标题：##text or 写完text(markdown)后在下面加三个---
   3. 三级标题：###text
   以此类推，下级标题的字会比上级标题要小（见第一、二行内容）
2. 段落
   1. 段落与段落之间要空一行（独立的两段之间要空行）
3. 分割线
   1. 三条及以上的‘-’表示分割线。效果如下。
   ---
4. 字体效果
   1. 加粗：用**，将内容包裹起来。如**text**。可以选中文本用快捷键（cmd+B）将选中部分加粗
   2. 斜体：用*，将内容包裹起来。如*text*。可以选中文本用快捷键（cmd+I）将选中部分加粗
   3. 删除线：用～～，将内容包裹起来。如~~text~~
   4. 高亮：用==，将内容包裹起来。如==text==
5. 列表
   1. 有序列表：就如本文中1、2、3···的组织方式。只要打出1.， 并按一下空格键即可自动生成有序列表。
   2. 无序列表：将有序列表中的数字换成‘*’即可。（见下）
   * hujun
     * kang   
   * linchen
     *yu 
6. 任务列表：使用‘- [x] ’和‘- [] ’形成一下格式 
   - [x] 已经完成的事1
   - [x] 已经完成的事2
   - [ ] 已经完成的事3 
7. 行缩进快捷键：’cmd+[‘为向左进行行缩进，‘cmd+]’为向右进行行缩进
8. 引用（也可理解为插入复制粘贴来的东西）
   1. 引用文本：使用>（如下）
   > hujun
   > kang
   > linchenyu
   2. 引用代码：`（上顿号，英文状态下按esc下面那个键）。（如下）
   ``` python
   print("hello world")
   ```
   3. 在引用代码时加入行数的显示（如下）
   ``` javascript {.line-numbers}
   function add(x, y){
    return x + y
   }
   ```
9.  超链接和图片
    1. 超链接格式：[链接名称] (链接地址，其为本地或网上地址)  （中间无空格，这里是为了避免直接显示，如下）。这样即可在文本中直接点击文字进行超链接，注意若是网上的内容必须要加http或https，否则会认为是本地内容
    [百度](https://www.baidu.com)
    [代码笔记](code.md)
    2. 图片格式：! [图片名称] (图片地址)（中间无空格，这里为了避免显示，如下）(最好将图片保存到本地)
    ![fern](./images/image000.png)
10. 表格
    1. 全部有字
    
    2. 
11. 注释（在调整代码重新形成preview内容时，不需要看的部分可以先注释掉，以加快形成preview内容）
    1. 单行注释：<!-- text -->
    2. 多行注释：<!-- text
    text
    text
    -->
12. 公式
    1. 行内公式和公式块
        1. 行内公式：用shift+4包裹（如下）
        $x^2+y^2=1$
        1. 公式块：用double(shift+4)包裹
        <!-- 
        1."\\"在下面的公式中是换行的意思 
        2.使用$$包裹多行公式可以使他们都居中
        -->
        $$
        x^2+y^2=z^2\\
        \begin{cases}
        a+b=1\\
        a^2+b^2=1\\
        \end{cases}
        $$
    2. 上标和下标
        1. 上标
        $x^2+y^{12}=1$
        1. 下标
        $x_1+x_2=1$
    3. 分式
        1. 较小的行内分式展示
        $\frac{1}{2}$
        $\frac{x+1}{x-1}$
        1. 展示型分式
        <!-- \displaystyle 的作用是将行内展示变为块状展示-->
        $\displaystyle\frac{x+1}{x-1}$
    4. 根式
        1. 开平方：
        $\sqrt{2}$
        1. 开n次方
        $\sqrt[n]{2}$
    5. 空格（代码中的数学公式中的空格和换行都会在编译时被忽略，必须用特殊命令）
        1. 紧贴
        <!-- \! -->
        $a\!b$
        1. 没有空格
        $ab$
        1. 小空格
        <!-- \, -->
        $a\,b$
        1. 中等空格
        <!-- \; -->
        $a\;b$
        1. 大空格（好像都一样）
        <!-- \+space -->
        $a\ b$
        1. quad空格
        <!-- \quad+space -->
        $a\quad b$
        1. 两个quad空格
        <!-- \qquad+space -->
        $a\qquad b$
    6. 累加、累乘、积分
        1. 累加
        $\sum_{k=1}^n\frac{1}{k} \qquad \displaystyle\sum_{k=1}^n\frac{1}{k}$
        1. 累乘
        $\prod_{k=1}^n\frac{1}{k} \qquad \displaystyle\prod_{k=1}^n\frac{1}{k}$
        1. 积分
        <!-- 
        1.公式里的字符的字体默认为某种字体，\rm可以移除这种格式 
        2.\int为单重积分；\iint为双重积分；\iiint为三重积分
        -->
        $\displaystyle\int_{0}^1xdx$
        $\displaystyle\int_{0}^1x{\rm d}x$
        $\int_{0}^1xdx$
        $\iint_{D_{xy}}$
        $\iiint_{\Omega_{xyz}}$
        1. 
    7. 括号
        <!--
        1.小括号（）
        2.中括号[]
        3.大括号\{ \}
        4.尖括号\langle \rangle
        -->
        $\displaystyle\left(\sum_{1}^n\frac{1}{k}\right)^2$
        $\displaystyle\left[\sum_{1}^n\frac{1}{k}\right]^2$
        $\displaystyle\left\{\sum_{1}^n\frac{1}{k}\right\}^2$
        $\displaystyle\left\langle\sum_{1}^n\frac{1}{k}\right\rangle^2$
        1. 括号表示的区间
        $\displaystyle\left(2,3\right)$
        $\displaystyle\left(2,3\right]$
        $\left(2,3\right)$
    8. 多行算式对齐
       1. 居中(在{aligned}中'&='能够将等号对齐，否则就是每行自己居中)
       $$
       \begin{aligned}
       y &=(x+5)^2-(x+1)^2 \\
       &=(x^2+10x+25)-(x^2+2x+1) \\
       &=8x+24 \\
       \end{aligned}
       $$
       $$
       \begin{aligned}
       y =(x+5)^2-(x+1)^2 \\
       =(x^2+10x+25)-(x^2+2x+1) \\
       =8x+24 \\
       \end{aligned}
       $$
       
       2. 左对齐(单个shift+4)
       $
       \begin{aligned}
       y &=(x+5)^2-(x+1)^2 \\
       &=(x^2+10x+25)-(x^2+2x+1) \\
       &=8x+24 \\
       \end{aligned}
       $
    9.  方程组(**’\cdots‘代表横的三个点，‘\vdots’代表竖的三个点，‘\ddots’代表斜的三个点**)
        $$
        \begin{cases}
        k_{11}x_1+k_{12}x_2+\cdots+k_{1n}x_n=b_1 \\
        k_{21}x_1+k_{22}x_2+\cdots+k_{2n}x_n=b_2 \\
        \cdots \\
        k_{n1}x_1+k_{n2}x_2+\cdots+k_{nn}x_n=b_n \\
        \end{cases}
        $$
    10. 矩阵和行列式
        1. 矩阵（**pmatrix代表小括号，bmatrix代表中括号；在matrix中‘&‘代表空一格**）
        $$
        \begin{pmatrix}
        1 & 1 & \cdots & 1 \\
        1 & 1 & \cdots & 1 \\
        \vdots & \vdots & \ddots & \vdots \\
        1 & 1 & \cdots & 1 \\
        \end{pmatrix}
        \quad
        \begin{bmatrix}
        1 & 1 & \cdots & 1 \\
        1 & 1 & \cdots & 1 \\
        \vdots & \vdots & \ddots & \vdots \\
        1 & 1 & \cdots & 1 \\
        \end{bmatrix}
        $$
        1. 行列式（vmatrix代表竖号）
            $$
            \begin{vmatrix}
            1 & 1 & \cdots & 1 \\
            1 & 1 & \cdots & 1 \\
            \vdots & \vdots & \ddots & \vdots \\
            1 & 1 & \cdots & 1 \\
            \end{vmatrix}
            $$
    11. 特殊字符
    $$
    \alpha\\
    \beta\\
    \delta\\
    \epsilon\\
    \eta\\
    \gamma \quad \Gamma\\
    \lambda \quad \Lambda\\
    \mu \quad \Mu\\
    \pi \quad \Pi\\
    \omega \quad \Omega\\
    \theta \quad \Theta\\
    \psi \quad \Psi\\
    \rho \quad \Rho\\
    \sigma \quad \Sigma\\
    \tau \quad \Tau\\
    \phi \quad \Phi\\
    \xi\\
    \zeta\\
    \Delta\\
    $$
    1.  公式编号与引用(使用equation时可以自动排号)
    $$
    x+2\tag{1.2}
    $$
    $$
    \begin{equation}
    x^n+y^n=z^n
    \end{equation}
    $$
    $$
    \begin{equation}
    x^n+y^n=z^n
    \end{equation}
    $$
    1.  其他一些
        1. 点乘 $\cdot$, 叉乘 $\times$, 异或 $\otimes$, 直和 $\oplus$, 加减 $\pm$, 复合 $\circ$.
        2. 小于等于 $\leq$, 大于等于 $\geq$, 不等 $\neq$, 恒等 $\equiv$, 约等 $\approx$, 等价 $\cong$, 相似 $\sim$, 相似等于 $\simeq$, 点等 $\doteq$.
        3. 逻辑与 $\land$, 逻辑或 $\lor$, 逻辑非 $\lnot$, 蕴涵 $\to$, 等价 $\leftrightarrow$.
        4. 因为 $\because$, 所以 $\therefore$, 存在 $\exist$, 任意 $\forall$.
        5. 左小箭头 $\leftarrow$, 右小箭头 $\rightarrow$, 左大箭头 $\Leftarrow$, 右大箭头 $\Rightarrow$, 右长箭头 $\xrightarrow[fgh]{abcde}$.
        6. 属于 $\in$, 包含于 $\subset$, 真包含于 $\subseteq$, 交 $\cap$, 并 $\cup$, 空集 $\empty$
        7. 短向量 $\vec{x}$, 长向量 $\overrightarrow{AB}$, 上横线 $\overline{p}$.
        8. 无限 $\infty$, 极限 $\lim$, 微分 ${\rm d}$, 偏导 $\partial$, 点求导 $\dot{y}$, 点二阶导 $\ddot{y}$, 变化量 $\Delta$, 梯度 $\nabla$.
        9. 横省略 $\cdots$, 竖省略 $\vdots$, 斜省略 $\ddots$.
        10. 常见函数 $\sin$, $\cos$, $\tan$, $\arcsin$, $\arccos$, $\arctan$, $\ln$, $\log$, $\exp$.
    2.  
13. 输出为pdf
    1. 在preview里右键点击，选择Open in Browser，然后在浏览器中选择打印（cmd+p）
14. 自动生成目录：在本文件最上方加上一行内容（[TOC]），即可自动生成目录，可以点击目录直达。但是目录只记录标题，不记录普通的列表。
15. 画图
    1. 流程图
        ```mermaid
        graph LR
            a --> b;
            b --> c
        ```    
    2. 思维导图
16. 在段落之间加空行：
    <br>
17. 在中文段落前空一个字：&emsp;
    &emsp;啦啦啦能够滤除偏振光的滤镜叫做偏振镜。普通的偏振镜叫做线偏振镜（PL镜）。把偏振镜装到镜头的前端.
18. 在中文段落前空两个字: &emsp;&emsp;
   &emsp;&emsp;能够滤除偏振光的滤镜叫做偏振镜。普通的偏振镜叫做线偏振镜（PL镜）。把偏振镜装到镜头的前端。 
   <br>
   I am asd asd asd asd asd asd asd asd asd asd asd asd asd asd asd asd asd asd asd asd asd asd asd asd asd 
19.  