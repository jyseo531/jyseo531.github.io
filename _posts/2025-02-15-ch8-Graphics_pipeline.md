---
title: Graphics-Ch8. Graphics Pipeline
author: jiyoung
date: 2025-02-15 11:04:00 +0800
categories: [Graphics, Study]
tags: [Graphics]
---
<script type="text/javascript">
  MathJax = {
    tex: {
      inlineMath: [['$', '$'], ['\\(', '\\)']]
    }
  };
</script>
<script type="text/javascript" src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-mml-chtml.js"></script>

## Intro
- **Graphics Pipeline** : object에서 시작해서 pixel이 업데이트되기까지의 일련의 operations sequence
- object-order rendering = rendering by "rasterization"
- 그래픽스 파이프라인 과정들 중에서 "vertex processing", "Rasterization", "Fragment Processing", "Blending" 이렇게 핵심 네 스텝에 대해서 이번 단원에 다루도록 하겠다
  - Vertex processing : vertices(꼭짓점)들이 연산되는 과정
  - Rasterziation stage : 그 꼭짓점들을 이용한 primitives가 보내져서 연산되고, 각각의 Primitive를 fragment로 쪼갬
  - Fragment processing : 쪼개진 fragment들이 연산되는 과정
  - Blending : 다양한 Fragment들이 대응되는 픽셀에 따라서 혼합됨

<br>

## 1. Rasterization
- 그래픽스 파이프라인 중 가장 핵심이 되는 과정
- rasterizer는 다음의 두 가지 일을 수행함 
  1. 픽셀을 순회함, 각 픽셀은 primitive로 덮여있음
  2. 그 primitive들을 보간(Interpolate)함

### 1.1. Line Drawing
- 오든 그래픽스 패키지들은 line drawing관련된 명령을 포함하고 있음
- 시작점 $(x_0, y_0)$과 끝점 $(x_1, y_1)$을 연결하는 직선 그리기
- 크게 implicit line equation을 이용하는 방법과 parametric한 line equation을 이용하는 방법으로 2가지 방법이 있음

#### 1.1.1. Implicit Line Equations
- **Midpoint alogrithm** 이용: 직선의 방정식 $f(x,y)$가 이 midpoint보다 <u>위에 있는지 아래에 있는지</u>에 따라서 픽셀 선택하는 알고리즘
- 시작점 끝점을 잇는 직선의 방정식 $f(x,y)$일 떄, 기울기 $m = \frac{y_1 - y_o}{x_1 - x_0}$
- 직선 위에 있는 점들에 대해서는 $f(x,y)=0$을 만족한다는 성질 이용
- $(x, y)$에서 다음으로 이동할 픽셀의 후보군(아래 그림에서 주황색 픽셀들)을 다음의 두 가지로 설정함 : $(x+1, y)$ 또는 $(x+1, y+1)$
  - 평행하게 오른쪽 한 칸으로 움직이거나, 대각선 방향으로 이동하는 경우의 수 
- **Midpoint** = $(x+1, y+0.5)$
  
<div align="center">

<table>
<tr>
<td>
<img src="assets/img/posts_storage/ch8/IMG_9ADA66D1DFE3-1.jpeg" width="200" alt="Midpoing algorithm">
</td>
<td>
  $f$가 midpoint보다 above한지, below한지 분류하는 방법 <br>
      : $d = f(x+1, y+0.5)$라고 할 때,  <br>
      1. if $d < 0$이라면 <br>
         대각선 방향으로 즉, $(x+1, y+1)$ 으로 픽셀 결정 <br>
      2. else $d > 0$이라면, <br>
          평행한 방향으로 즉, $(x+1)$ 으로 픽셀 결정
</td>
</tr>
</table>

</div>

### 1.2. Triangle Rasterization
- 2D points로 **2D 삼각형** 그리는 방법
  - Given points $p_0 = (x_0, y_0)$ , $p_1 = (x_1, y_1)$, $p_2 = (x_2, y_2)$ in screen space
  - 각 꼭짓점이 color $c_0, c_1, c_2$를 저장하고 있다는 가정하에, 삼각형 위의 한 점을 $(\alpha, \beta, \gamma)$계수르 이용하여 표현 가능 ( barycentric 좌표계를 이용): <br>
  
  $$\begin{align} c = \alpha c_0 + \beta c_1 + \gamma c_2. \end{align}$$

- 
