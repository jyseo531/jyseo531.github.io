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
<br>

### 1.2. Triangle Rasterization
- 2D points로 **2D 삼각형** 그리는 방법
  - Given points $p_0 = (x_0, y_0)$ , $p_1 = (x_1, y_1)$, $p_2 = (x_2, y_2)$ in screen space
  - 각 꼭짓점이 color $c_0, c_1, c_2$를 저장하고 있다는 가정하에, 삼각형 위의 한 점을 $(\alpha, \beta, \gamma)$계수르 이용하여 표현 가능 ( barycentric 좌표계를 이용): <br>
  
  $$\begin{align} c = \alpha c_0 + \beta c_1 + \gamma c_2. \end{align}$$

  - 위 방법의 interpolation을 "**Gouraud interpolation**"(고러드 보간, 고러드 쉐딩이랑 차이점 공부하기)이라고도 부름


- line drawing이랑 삼각형 rasterization의 미묘한 차이점중에 하나는 **vertices and edges**에 있다.
1. **"Hole problem"**: 인접한 삼각형을 그릴 떄, hole이 생기면 안됨  
2. **"Order problem"** : 만약 인접한 삼각형들이 다른 색상을 갖고 있다면 그려지는 순서에 따라서 이미지 색상이 달라짐

이 두가지 문제를 완화하기 위해, 삼각형의 center가 삼각형 내부에 존재할때만 픽셀을 그리는 성질을 이용함 (당연한거 아님?)
- 같은 말로, barycentric coordinate상에서 삼각형의 중심점(center)이 (0,1)간격 사이에 존재해야 한다는 의미
- 만약, center가 삼각형의 edge에 정확하게 존재하면 어떻게 하는가?? 이 질문에 대한 내용은 후에 다룬다

- **Barycentric Coordinate**을 통해 삼각형같은 다면체의 **"꼭짓점"**이 주어지는 상황에서 어떤 <u>위치</u>의 픽셀에 그릴지와 그 픽셀의 <u>색상</u>또한 결정하는 key가 된다.
- 모든 픽셀에 대해서 고러드 보간을 이용하면 계산 복잡성이 크기 때문에, 아래의 간단한 방법으로 효율성 증가시킬 수 있음
  - 삼각형의 세 꼭짓점을 기준으로 한 **bounding rectangle**을 찾아서 이 직사각형 내부에 존재하는 후보 픽셀들에 대해서만 계산 Looping을 진행한다


#### 알고리즘
```python
x_min = floor(x_i)
x_max = ceiling(x_i)
y_min = floor(y_i)
y_max = ceiling(y_i)

for y in range(y_min, y_max + 1):
    for x in range(x_min, x_max + 1):
        α = f_12(x, y) / f_12(x_0, y_0)
        β = f_20(x, y) / f_20(x_1, y_1)
        γ = f_01(x, y) / f_01(x_2, y_2)

        if α > 0 and β > 0 and γ > 0:s
            c = α * c_0 + β * c_1 + γ * c_2
            drawpixel(x, y, color=c)
```
- 여기서 $f_{ij}$는 꼭짓점을 잇는 직선의 방정식(line drawing 섹션)이고 아래처럼 표현함
  $$\begin{align} f_{01}(x,y) &= (y_0 - y_1)x + (x_1 - x_0)y + x_0y_1 - x_1y_0 , \\
                  f_{12}(x,y) &= (y_1 - y_2)x + (x_2 - x_1)y + x_1y_2 - x_2y_1 , \\
                  f_{20}(x,y) &= (y_2 - y_0)x + (x_0 - x_2)y + x_2y_0 - x_0y_2 . \end{align}$$

  ![img.png](assets/img/posts_storage/ch8/IMG_2F6ED0EC9567-1.jpeg)


#### Dealing with Pixels on Triangle Edges
- 삼각형의 *중심(center)이 Edge에 정확하게 존재*하는 경우의 픽셀은 어떻게 결정할 것인지에 대한 내용
- 삼각형 중심이 삼각형 변에 위치하는 대표적인 예시가 **직각-삼각형**임

<div align="center">
<table>
<tr>
<td>
<img src="assets/img/posts_storage/ch8/IMG_20C5C034366A-1.jpeg" width="200" alt="Midpoing algorithm">
</td>
<td>
 "off-screen point"는 두 삼각형의 공유된 변의 정확히 한쪽에 위치하며, <br>
 우리가 그릴 edge는 바로 그 변임, <br>
 공유 변 위에 있지 않은 정점들은 해당 변 기준으로 서로 반대쪽에 위치한다. <br>
</td>
</tr>
</table>
</div>
<br>

- 따라서, 그림 상 a 또는 b 정점 중 하나는 off-screen point와 shared edge기준 **같은 부호**면을 가진다는 성질이용 
- $pq > 0$

#### 알고리즘
```python
y_max = ceiling(y_i)

f_α = f_12(x_0, y_0)
f_β = f_20(x_1, y_1)
f_γ = f_01(x_2, y_2)

for y in range(y_min, y_max + 1):
    for x in range(x_min, x_max + 1):
        α = f_12(x, y) / f_α
        β = f_20(x, y) / f_β
        γ = f_01(x, y) / f_γ

        if α >= 0 and β >= 0 and γ >= 0:
            if (α > 0 or f_α * f_12(-1, -1) > 0) and \  # 부호
               (β > 0 or f_β * f_20(-1, -1) > 0) and \  # 부호
               (γ > 0 or f_γ * f_01(-1, -1) > 0):       # 부호

                c = α * c_0 + β * c_1 + γ * c_2
                drawpixel(x, y, color=c)
```
<br>

### 1.3. CLipping
- 눈의 뒤(behind)에 존재하는 공간 혹은 **view volume의 바깥**에 존재하는 primitive를 clipping해주는 과정들이 있어야 정확한 Rasterizing을 수행할 수 있다
- 

<br>

## 2. Operations <u>Before and After</u> Rasterization


<br>

## 3. Simple Anti-aliasing
