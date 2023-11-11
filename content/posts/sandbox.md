---
title: "Sandbox"
draft: true
---

## Markdown All in One (VSCode)
- `Alt + S` で打ち消し線

## HTML
{{<rawhtml>}}
<p align="center" style="color: blue;"><strong>This is raw HTML</strong></p>
{{</rawhtml>}}


## Diagram
```goat
      .               .                .               .--- 1          .-- 1     / 1
     / \              |                |           .---+            .-+         +
    /   \         .---+---.         .--+--.        |   '--- 2      |   '-- 2   / \ 2
   +     +        |       |        |       |    ---+            ---+          +
  / \   / \     .-+-.   .-+-.     .+.     .+.      |   .--- 3      |   .-- 3   \ / 3
 /   \ /   \    |   |   |   |    |   |   |   |     '---+            '-+         +
 1   2 3   4    1   2   3   4    1   2   3   4         '--- 4          '-- 4     \ 4

```

- [Diagrams](https://gohugo.io/content-management/diagrams/)
- [HUGOサイトでMermaid記法](https://tech.nosuz.jp/post/2022-02/mermaid/)


## $\LaTeX$
$$
\begin{align*}
a  &= b  & c  &= d  & e  &= f \\\\\\
a' &= b' & c' &= d' & e' &= f'
\end{align*}
$$

$$
\hat u_0(x_k):=\left\\{
\begin{align\*}
  &g(\bar x_k), & x_k \in \partial \Omega_{\epsilon} \\\\\\
  &\hat u_0 (x_{k+1}), & \text{otherwise}
\end{align\*} 
\right.
$$

$$
\\begin{array}{rl} \\biggl\\\{ \\frac{1}{g(x)} \\biggr\\\}^{\\prime} &=& \\lim_{h \\to 0}\\frac{\\frac{1}{g(x+h)}-\\frac{1}{g(x)}}{h} \\cr\\cr &=& \\lim_{h \\to 0}\\frac{g(x)-g(x+h)}{g(x) g(x+h) h} \\cr\\cr  &=& -\\frac{1}{ \\bigl\\\{ g(x) \\bigr\\\}^2} \\lim_{h \\to 0}\\frac{g(x+h)-g(x)}{h} \\cr\\cr &=& -\\frac{g^{\\prime}(x)}{ \\bigl\\\{g(x) \\bigr\\\}^2} \end{array}
$$

$\sum\limits_\mathrm{min}^\mathrm{max}\quad\sum_\mathrm{min}^\mathrm{max}\quad$
$\displaystyle\sum_\mathrm{min}^\mathrm{max}$

