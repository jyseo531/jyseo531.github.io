---
title: DrivingGaussian++, arxiv preprint 2025
author: jiyoung
date: 2025-09-28 12:00:00 +0800
categories: [Study,Paper-review]
tags: [3D Editable Simulation, 3D Editing, Autonomous Driving]
---

<script type="text/javascript">
  MathJax = {
    tex: {
      inlineMath: [['$', '$'], ['\\(', '\\)']]
    }
  };
</script>
<script type="text/javascript" src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-mml-chtml.js"></script>

[paper link](https://arxiv.org/abs/2508.20965)
- DrivingGaussian(CVPR 2024) 후속 연구

## Abstract
- driving scene(주행 환경)에서 "**realistic reconstructing**"과 "**contrallable editing"**을 수행하는 프레임워크 제안
- *정적인 배경*을 3D gaussian으로 복원하고, *움직이는 물체*는 dynamic Gaussian graph로 정확한 위치와 가려짐을 복원함
- **LiDAR prior**와 함께 사용함으로서 정교하고 연속적인 장면 복원을 가능하게 함
- **LLM과 통합**함으로써 dynamic object의 움직임 궤적을 자동적으로 생성할 수 있음

## Introduction
- 자율주행 관련 기술의 발전 => "주행 데이터"의 중요성 
- **3D scene editing**은 이 자율주행분야의 **데이터의 다양성을 확보하고 향상**하는데 중요한 태스크임
  - test와 validation시 발생하는 어렵고 드문 케이스를 합성하는데 유용함
  - 현실 세계에서의 다양한 주행 환경을 시뮬레이션하는 것 가능함
- 이 3D scene editing은 폼이 아래와 같이 다양한 카테고리로 존재함
  - motion modification
  - weather simulation
  - addition / removal of objects
- 하지만 기존의 3D scene editing 메소드들은 특정 도메인에만 특화되고 저런 다양한 스콥의 편집 태스크를 통합적으로 다루는 것이 거의 없음
- 그리고 기존 메소드들은 2D Editing 기술을 활용해서 이 2D edit을 반복해서 수행하기 때문에 multi-view consistency가 깨지고 계산 비용이 많이 드는게 다수임
- sparse한 vehicle-mounted sensor data로부터 복잡한 3D 장면 복원하는것 어려움
  - complex geometry, diverse optical degradation, and spatiotemporal inconsistency때문
  
<br>

- 본 논문에서는 주위의 움직이는 주행 환경에서 계층적인 모델 구조를 통해 센서들로부터 시계열적인 데이터(sequential data)를 이용하는 것이 핵심 아이디어임
- "**Composite Gaussian Splatting**" 적용해서 장면을 (1)정적인 배경과 (2)움직이는 물체로 분해하고, 각 부분을 개별적으로 복원함
  - real world에서의 가려짐(occlusion)을 글로벌 렌더링으로 포착하고, 정적인 배경과 동적인 물체들을 잘 포착함
- **LiDAR 센서로 얻은 사전지식**을 가우시안 표현에 통합하여서 보다 나은 multi-view consistency  ghkrqh
- 추가로, editing에서 multitask 지원
  - texture modification
  - weather simulation
  - object manipulation
 
- computational & temporal 비용 줄이기 위해 "**post-editing"전략** 사용

## Method
- **training-free editing** within 3D autonomous driving scenes
- 그니까 reconstruction 파이프라인(composite gaussian splatting포함) 이랑 editing 파이프라인 두개를 각각 제안함
<div align="center">
<table>

<img src="/assets/img/posts_storage/DrivingGaussian/recon_method.png" width="800" alt="teaser">
</table>
</div>

### Composite Gaussian Splatting
- **LiDAR Prior**를 활용한 초기화: 기존 3D Gaussian Splatting(3DGS)은 SfM(Structure-from-Motion)으로 Gaussians를 초기화하지만, 이는 자율주행 환경의 대규모 언바운드(unbounded) 장면에서 부정확한 기하학적 구조를 초래할 수 있음
-  DrivingGaussian++는 LiDAR prior를 활용하여 Gaussians의 초기화를 개선하고, multi-camera consistency을 유지
#### **< 본격 CGS 이전 pre-setting>**
-  가장 먼저 LiDAR 스윕 데이터를 병합하여 완전한 "point cloud"을 얻음 ($\mathcal{L}$로 표기)
-  이후 Colmap을 이용해서 각 이미지에 대한 image feature $\mathcal{X}=x_p^q$를 얻음
-  그 후, 주의를 둘러싼 이미지 뷰에서의LiDAR point cloud $l$를 2D 이미지에 투영하여 "camera coordinate system"에서 2D-pixel상 정확한 포인트 위치와 기하학적 정보를 얻는 방식
<div align="center">
<table>
<img src="/assets/img/posts_storage/DrivingGaussian/composite_GS.png" width="800" alt="teaser">
</table>
</div>

#### Incremental "Static 3D Gaussians"
- 정적인 배경(static backgrounds)
- 차량 이동에 따른 **정적 배경의 대규모 변화를 처리**하기 위해, 정적 장면을 **LiDAR depth를 기반**으로 여러 bin으로 나눔
  - 각 bin은 시간에 따라 인접한 bin의 중첩 영역을 사용하여 점진적으로 Gaussian 필드에 병합
  - 원근 변화로 인한 스케일 혼란과 아티팩트를 방지
-   Gaussians의 파라미터(위치 $P(x,y,z)$, 공분산 행렬 $\Sigma$, 구면 고조파 계수 $C(r,g,b)$, 불투명도 $\alpha$)는 주변 뷰를 통해 아래 수식으로 업데이트
    $$\tilde{C} = \varsigma(G_{s}) \sum \omega(\hat{C}(G_{s}) | R, T)$$
    -  $\tilde{C}$는 최적화된 픽셀 색상
    -  $\varsigma$는 미분 가능한 Splatting
    -  $\omega$는 서로 다른 뷰에 대한 가중치
    -  $[R, T]$는 다중 카메라 뷰 정렬을 위한 뷰-매트릭스


#### Composite "Dynamic Gaussian Graph"
- 동적인 물체(dynamic objects)
- 다수의 동적 객체와 시간적 변화를 처리하기 위해 동적 가우시안 그래프를 도입
- 데이터셋의 **bounding box를 사용하여 동적 객체를 정적 배경에서 분리**하고, **Grounded SAM** 모델을 통해 픽셀 단위로 정확하게 추출
- 그래프 $H = <O, G_d, M, P, A, T>$는 각 객체 $o \in O$에 대한 동적 Gaussians $g_i \in G_d$, 변환 행렬 $m_o \in M$, 중심 좌표 $p_o(x_t, y_t, z_t) \in P$, 방향 $a_o = (\theta_t, \phi_t) \in A$를 포함
- 각 동적 객체별로 별도의 가우시안들이 계산됨
- 객체-world coordinate 변환 행렬 $m_o^{-1} = R_o^{-1} S_o^{-1}$를 통해 정적 배경의 좌표계로 변환
- 객체 간의 폐색은 카메라와의 거리에 기반하여 불투명도 $\alpha_{o,t}$를 조정하여 처리
  - $$\alpha_{o,t} = \sum (p_t - b_o)^2 \cdot \frac{\cot a_o}{\|(b_o|R_o, S_o) - \rho\|^2} \alpha_{p_0}$$
- 정적 배경의 가우시안인 $G_s$와 동적 가우시안 그래프 $H$를 결합시켜서 <복합 가우시안 필드 $G_{comp}$>를 형성한다

#### Global Rendering
- 미분 가능한 3D Gaussian Splatting renderer를 사용하여 전역 복합 3D Gaussian을 2D로 투영
- timestep마다 주변 부로 supervised learning
- loss function은 아래의 세 가지 구성 요소들로 이루어짐(TSSIM loss, Robust loss, LiDAR loss)
- $$L_{TSSIM}(\delta) = 1 - \frac{1}{Z} \sum_{z=1}^{Z} SSIM(\Psi(\hat{C}), \Psi(C))$$
  - rendered tile과 gt간의 유사도 측정
- $$L_{Robust}(\delta) = \kappa(\|\hat{I} - I\|^2)$$
  - 가우시안 이상치를 줄이는 역할
- $$L_{LiDAR}(\delta) = \frac{1}{s} \sum \|P(G_{comp}) - L_s\|^2$$
  - LiDAR prior에서 예상되는 가우시안 위치를 업데이트해서 더 나은 기하학적 구조를 얻는 역할(?)

---
### Controllable "Editing" for Dynamic Driving Scenes
<div align="center">
<table>
<img src="/assets/img/posts_storage/DrivingGaussian/edit_method.png" width="800" alt="teaser">
</table>
</div>

- 복원이랑 가우시안 렌더링까지 끝났고 에디팅 파이프라인(3가지 태스크 포함)얘기
- 
