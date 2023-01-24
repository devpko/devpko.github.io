---
layout: post
title: "[Color Science] Color의 이해 및 CIE Lab 활용 방법 (with python, opencv)"
description: >
  다양한 색 공간에 대한 이해 및 활용 방법
sitemap: false
hide_last_modified: true
comments: true
---


# [Color Science] Color의 이해 및 CIE Lab 활용 방법 (with python, opencv)

## 다양한 색 공간에 대한 이해의 필요성
영상인식 알고리즘을 개발하는 사람이라면 RGB, Gray, HSV, YCbCr 색상 모델에 대해서는 많이 들어보고 사용해봤을 것이다.
이미지를 위와 같은 색상 모델로 변환하여 전처리하는 작업은 우리가 원하는 RoI(Region of Interest)를 얻는데, 굉장히 중요한 첫번째 일이다.

그런데 CIE RGB, CIE XYZ, CIE Lab에 대해서는 들어봤는가? 
우리가 일반적으로 알고 있는 디지털 색상 모델 외에도 색을 표현하는데는 다양한 방식들이 있으며,
어떤 색 공간을 어떻게 활용하냐에 따라 전처리의 난이도와 결과물은 확연하게 달라진다.
그렇기 때문에 computer vision 분야에서 종사하는 사람에게 color science에 대한 이해는 필수 요소라 생각한다.

이번 포스팅에서는 color space, color system, 그리고 아마 여태까지는 익숙치 않았던 Lab 색 공간에 대해 이해하고, 이를 활용하는 방법에 대해 설명하고자 한다.


## 같은 빨간색이지만 사람마다 다르게 보이는 이유
분명히 같은 영상인데 A Display에서 보이는 색상과 B Display에서 보이는 색상이 다른 것 같은 경험, 아마 한번쯤은 겪어봤을 것이다.
누군가에게는 빨간색, 누군가에게는 짙은 빨간색, 누군가에게는 짙은 핑크색 등 rgb 코드가 모두 동일한 색상임에도 불구하고,
사람마다 인지하는 색상의 차이가 발생하는 이유는 무엇일까?

![difference_red](/assets/img/computer-vision/color-science/difference_red.png)

A Display에서의 (255, 0, 0)와 B Display에서의 (255, 0, 0)는 같은 '빨간색 디지털 코드'이지만, 측색기로 측정한 실제 값은 조금씩 다른 색일 수가 있다. 
이러한 차이가 생기는 이유는 바로 각 디스플레이마다 표현하고자 하는 색역(color gamut)이 다르기 때문이다.

위 chromatic diagram (색도도) 그림을 보면 A Display의 삼각형 영역과 B Display의 삼각형 영역이 다른 것을 볼 수 있는데,
각 삼각형의 빨간 부분의 꼭지점 위치가 Display 에서 표현하고자 하는 (255, 0, 0) 지점이다.
삼각형 영역은 색역이라 부르며, display가 표현할 수 있는 모든 색들의 범위를 의미하는데, 
display마다 서로 다른 pixel 크기와 형태 때문에 컬러의 불일치가 발생하게 되는 것이다.


## Color History (feat. CIE RGB, CIE XYZ, CIE Lab)

개인적으로 공부하면서 작성한 내용이라 부정확한 내용이 있을 수도 있으니 참고바랍니다 :)
{:.note}

Color space 와 Color system이 무엇인지 이해하기에 앞서 잠시 색의 역사와 함께 CIE RGB, CIE, XYZ, CIE Lab가 무엇인지부터 알고가보자.

![color_history](/assets/img/computer-vision/color-science/color_history.png)

1850년, 빛의 삼원색이 정의된 이후, 빛, 조명, 물체, 그리고 보는 사람에 따라 발생하는 불일치를 해소하기 위해 표준으로 삼을 수 있는 기준이 필요하게 되었으며, 
국제조명위원회 CIE (Commission internationale de l'éclairage)가 그 기준을 발표하게 되었다.


### CIE RGB
1920년대, W.G.Wright와 J.Guild가 인간의 시력에 관한 일련의 등색 실험을 독립적으로 수행했고, 이를 바탕으로 CIE RGB 색 공간이 만들어졌다.
(여기서 말하는 CIE RGB는 디지털코드 RGB와는 다른 것이니 참고 바란다.)
CIE RGB 색상 공간은 특정 단색 (단일 파장) 원색 세트로 구별되는 많은 RGB 색상 공간 중 하나이다.
등색 실험이란, 우리 눈은 장파장(빨간색에 가까운 빛), 중파장(초록색에 가까운 빛), 단파장(파란색에 가까운 빛)을 각각 감지하는 세 종류의 원추세포가 존재하기에 
"모든 색은 3원색의 조합으로 만들어 낼 수 있다." 라는 가설을 바탕으로 380nm ~ 780nm까지의 단색광을 만들어내는 실험을 말한다.

![CIE RGB](/assets/img/computer-vision/color-science/CIERGB.png)

위 그림은 등색 실험의 결과를 그래프화한 것이고, 이를 통해 아래와 같은 수식을 도출해 낼 수 있는데, 일단은 참고만 해두자.

$$
\begin{align}
  R &= \int_0^\infty \mathrm{I}(\lambda)\bar{r}(\lambda) \mathrm{d}\lambda  \quad \quad \quad \quad \quad \quad  r = { R \over R + G + B } \\[2em]
  G &= \int_0^\infty \mathrm{I}(\lambda)\bar{g}(\lambda) \mathrm{d}\lambda  \quad \quad => \quad \quad g = { G \over R + G + B } \\[2em]
  B &= \int_0^\infty \mathrm{I}(\lambda)\bar{b}(\lambda) \mathrm{d}\lambda  \quad \quad \quad \quad \quad \quad b = { B \over R + G + B }
\end{align}
$$

I(λ) : 파장에 따른 광원값, 표준은 P_D65이며 [opencv](https://docs.opencv.org/4.6.0/de/d25/imgproc_color_conversions.html)에서도 해당 표준을 사용
{:.figcaption}

CIE RGB 색상 공간에는 한 가지 문제점이 있는데, 위 이미지를 자세히 보면 하늘색으로 색칠된 음의 영역은
사람의 눈으로 볼 수도 없고 색 재현이 안된다는 것이다.
CIE RGB의 이러한 문제점을 수학적으로 보완하여 나온 것이 바로 1931년에 정립된 CIE XYZ 색상 공간(CIE 1931 색 공간)이다.

### CIE XYZ
![CIE XYZ](/assets/img/computer-vision/color-science/CIEXYZ.png)

CIE XYZ 색 공간은 CIE RGB 색 공간으로부터 선형변환을 통해 얻을 수 있으며, 모든 값이 양수를 갖는다.
모든 색이 chromatic diagram에서 (1, 0), (0, 1), (0, 0)의 삼각형 내에 존재해야 인간이 볼 수 있는 색으로 표현이 가능했다.
그래서 CIE RGB chromatic diagram의 음의 영역을 양의 영역으로 변환하기 위해 Cr, Cg, Cb 를 잇는 삼각형의 
각 꼭지점들을 xy = (1, 0), (0, 1), (0, 0)에 대응해 변환한 것이 CIE XYZ chromatic diagram이다.
이 그래프를 수학적으로 표현한 것이 아래 수식이다.

$$
\begin{align}
  \begin{bmatrix} X\\Y\\Z \end{bmatrix} &= { 1 \over 0.17697 } \begin{bmatrix} 0.49&0.31&0.20\\0.17697&0.81240&0.01063\\0.00&0.01&0.99 \end{bmatrix} \begin{bmatrix} R\\G\\B \end{bmatrix}
\end{align}
$$

위 수식과 CIE RGB에서 구한 수식을 활용하면 아래와 같은 function을 도출해 낼 수 있다.

$$
\begin{align}
  \bar{x}(\lambda) &= 2.7689 \bar{r}(\lambda) + 1.7517 \bar{g}(\lambda) + 1.1302 \bar{b}(\lambda) & X = \int_0^\infty \mathrm{I}(\lambda)\bar{x}(\lambda) \mathrm{d}\lambda \qquad\qquad & \qquad x = { X \over X + Y + Z } \\[2em]
  \bar{y}(\lambda) &= 1.0000 \bar{r}(\lambda) + 4.5907 \bar{g}(\lambda) + 0.0601 \bar{b}(\lambda) & Y = \int_0^\infty \mathrm{I}(\lambda)\bar{y}(\lambda) \mathrm{d}\lambda \qquad => & \qquad y = { Y \over X + Y + Z } \\[2em]
  \bar{z}(\lambda) &= 0.0000 \bar{r}(\lambda) + 0.0565 \bar{g}(\lambda) + 5.5943 \bar{b}(\lambda) & Z = \int_0^\infty \mathrm{I}(\lambda)\bar{z}(\lambda) \mathrm{d}\lambda \qquad\qquad & \qquad z = { Z \over X + Y + Z } = 1 - x - y
\end{align}
$$

하지만 이러한 보완에도 불구하고 색상과 채도를 색차 같은 공학 계산에 활용이 어렵다는 단점이 있다.
CIE XYZ에서는 색상을 나타내는 선이 직선이 아닌 굽은 선이며, 색상각도 균등하지가 않다. 
그리고 채도를 나타내는 원들은 둥근 동심원들이어야 되는데 그렇지 않으며, 사이 간격도 일정치 않다.

이러한 이유 때문에 공학 계산에 활용이 어려운 CIE XYZ를 보완한 것이 1976년에 발표한 CIE Lab이다.

### CIE Lab
![CIELab](/assets/img/computer-vision/color-science/CIELAB.png)

L : 휘도, black-white 요소, 0(black) ~ 100(white) 값을 가짐  
a : green-red 요소, -128(green) ~ +128(red) 의 값을 가짐  
b : blue-yellow 요소, -128(blue) ~ +128(yellow) 의 값을 가짐
{:.note}

"CIE 1976 L\*a\*b\*" 는 균일한(uniform) 색공간 좌표로서 눈으로 보는 것과 매우 근소한 차이를 보여주기 때문에 현재 세계적으로 표준화되어 있는 색공간이다.
CIE Lab는 CIE XYZ에서 도출해낸 수식을 아래와 같이 변형한 형태이다.

$$
\begin{align}
  L^* &= 116 f(Y/Y_n) - 16 \\[2em]
  a^* &= 500 [f(X/X_n) - f(Y/Y_n)]  \qquad & f(t) = \begin{cases} t^{1/3} & \text{if } y = 1 \\
                                                                         \frac{1}{3} \left( \frac{29}{6} \right)^2 t + \frac{4}{29} & \text{otherwise.} \end{cases} \\[2em]
  b^* &= 200 [f(Y/Y_n) - f(Z/Z_n)] \\[2em]
      & \Tiny X_n, Y_n \text{ 및 } Z_n \text{은 CIE XYZ를 표준 흰색에 대해 정규화한 값}
\end{align}
$$

이렇게 정립된 CIE Lab는 Uniform한 공간이기에 이 공간 안에서 색차를 계산했을 때 사람이 느끼는 색차와 어느 정도 유사하다고 할 수 있다.
위 그림에서 볼 수 있듯이 θ(각도) 값은 색상이 얼마나 차이나는지를 나타내고, 
L(휘도) 축을 제외한 a점과 b점 사이의 euclidian distance 를 구하면 채도가 얼마나 차이나는지 알 수 있으며,
3차원 공간 상의 euclidian distance를 구한다면 delta E, 색차를 구할 수 있다. 

Lab 색 공간의 가장 큰 장점은 RGB나 CMYK와 달리 매체에 독립적이며, 인간의 시각에 대한 연구를 바탕으로 정의되었다는 것이다. 
특히 L(휘도) 값은 인간이 느끼는 밝기에 대응하도록 설계되어 실제 인간이 느끼는 것과 유사하다는 것이 큰 장점이다.

이 Lab와 유사한 색상계로는 대표적으로 YUV(YCbCr)가 있다. 
YUV(YCbCr)는 밝기(Y)와 청록(U or Cb), 적황(V or Cr)의 조합으로 이루어지는 색체계이며 영상 처리에서 주로 쓰이곤 한다.
{:.note}

여기까지가 Color의 역사에 대한 간략한 설명이었으며, 역사와 함께 색 체계가 어떻게 발전했는가에 대해 이해하면
CIE RGB, CIE XYZ, CIE Lab를 이해하는데 도움이 될 것이라 생각한다.


## Color Space, Color System
이제 [Wiki](https://ko.wikipedia.org/wiki/%EC%83%89_%EA%B3%B5%EA%B0%84)에 정의된 color space 및 color system을 보면 이해가 될 것이다.
> 색 공간(color space)은 색 표시계(color system)를 3차원으로 표현한 공간 개념이다. 
> 색 표시계의 모든 색들은 이 색 공간에서 3차원 좌표로 나타낸다.
> 여기서 색 표시계(color system)란 CIERGB, CIEXYZ, CIELab, CIELUV 등의 색 체계를 말한다. 
> 최근 디자인 학계나 산업계에서 색채 디자인 또는 시각 디자인을 연구하거나, 카메라, 스캐너, 모니터, 컬러 프린터 등의 컬러 영상 장비 개발 및 응용 단계에서 색 공간은 정확한 색을 재현하기 위하여 필수적으로 활용되고 있다.

자 CIE Lab, Color Space, 그리고 Color System에 대한 이해가 되었다면, 
이제 실제로 이것을 어떻게 활용할 수 있는지 샘플 python 코드와 함께 보자.  


## 색 공간 활용

[opencv-document](https://docs.opencv.org/4.7.0/d8/d01/group__imgproc__color__conversions.html)에서 제공하는 색상 모델들
{:.note}

Lab color space에 대해 이해가 어느 정도 됐을거라 생각한다.
그럼 아래의 예시를 통해 어떤 색상 모델을 사용하냐에 따라 결과물이 어떻게 다르게 나오는지 한번 비교해자.
 
![watermelon](/assets/img/computer-vision/color-science/watermelon.png)

위 watermelon 이미지로부터 빨간 영역만 추출하는 task를 두 가지 방법으로 해보려한다.
1. bgr 이미지로부터 r > 128 이상 이상인 부분을 masking 처리
2. lab 이미지로 변환하여 a > 128 이상인 부분을 masking 처리

실제로는 L : black-white 요소 0 ~ 100, a : green-red 요소 - ~ +, blue-yellow 요소 - ~ + 의 값을 가지지만,
[opencv](https://docs.opencv.org/4.6.0/de/d25/imgproc_color_conversions.html)에서는 L ← L∗255/100, a ← a+128, b ← b+128 하여 각 값들을 0 ~ 255로 바꿔 사용한다.
{:.note}

~~~python
import cv2
import numpy as np


path = r'watermelon.png'

bgr = cv2.imread(path)  # opencv에서 이미지를 읽을 때는 BGR 형태로 읽힘
lower = np.array([0, 0, 128])
upper = np.array([255, 255, 255])
masking = cv2.inRange(bgr, lower, upper)
bgr_masked = cv2.bitwise_and(bgr, bgr, mask=masking)

bgr = cv2.imread(path)
lab = cv2.cvtColor(bgr, cv2.COLOR_BGR2Lab)
lower = np.array([0, 128, 0])
upper = np.array([255, 255, 255])
masking = cv2.inRange(lab, lower, upper)
lab_masked = cv2.bitwise_and(bgr, bgr, mask=masking)

concat = cv2.hconcat([bgr_masked, lab_masked])
cv2.imshow('concat', concat)
cv2.waitKey()
~~~

![watermelon_concat](/assets/img/computer-vision/color-science/watermelon_concat.png)

왼쪽에 있는 이미지는 RGB에서 r > 128 이상인 부분을 masking 처리한 결과이고,
오른쪽에 있는 이미지는 Lab로 변환하여 a > 128 이상인 부분을 masking 처리한 결과이다.
RGB 공간에서 masking 처리를 하려고 하면 빨간색을 제외한 영역에도 어느 정도의 R 값이 존재하기 때문에,
결과 이미지와 같이 원하는 RoI 외 다른 영역도 함께 masking 처리가 되지만, 
Lab 색 공간으로 변환하여 처리를 한다면 색 분리가 깔끔하게 되는 것을 볼 수 있다.

어떤가, 이렇게 단순히 색 공간을 변환해서 처리하는 것 만으로도 확연한 차이가 있을을 알 수 있다.
이것이 바로 Computer Vision 분야에 종사하는 사람이라면 Color Science에 대한 이해가 필요하다고 생각하는 이유이다.

### My Opinion
색 공간에 대해 어느 정도 이해만 하고 있어도 어떤 task에서 어떤 색상 공간을 활용해야 할지 판단할 수 있을 것이라 생각한다.
가령, 색상에 상관없이 밝기에 따른 task를 처리하는 것이라며 단순히 Gray로만 변환해줘도 해결 될 수도 있을 것이고,
색을 분리해야하는 task라면 Lab로 변환하여 처리하는게 효율적일 것이다.

이 글을 읽는 여러분도 Color Science에 대해 간략하게 이해하고 실무에서 다양하게 활용 할 수 있었으면 하는 바람이다 :)