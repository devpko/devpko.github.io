---
layout: post
title: Color Science
description: >
  다양한 색 공간에 대한 이해 및 활용 방법.
sitemap: false
hide_last_modified: true
comments: true
---


# Color Science

## 다양한 색 공간에 대한 이해의 필요성
영상인식 알고리즘을 개발하는 사람이라면 RGB, Gray, HSV, YCbCr 색상 모델에 대해서는 많이 들어보고 사용해봤을 것이다.
이미지를 위와 같은 색상 모델로 변환하여 전처리하는 작업은 우리가 원하는 RoI(Region of Interest)를 얻는데, 굉장히 중요한 첫번째 일이다.

그런데 CIERGB, CIEXYZ, CIELAB에 대해서는 들어봤는가? 
우리가 일반적으로 알고 있는 디지털 색상 모델 외에도 색을 표현하는데는 다양한 방식들이 있으며,
어떤 색 공간을 어떻게 활용하냐에 따라 전처리의 난이도와 결과물은 확연하게 달라진다.
그렇기 때문에 computer vision 분야에서 종사하는 사람에게 color science에 대한 이해는 필수 요소라 생각한다.

이번 포스팅에서는 color space, color system, 그리고 아마 여태까지는 익숙치 않았던 Lab 색 공간에 대해 이해하고, 이를 활용하는 방법에 대해 설명하고자 한다.


## 같은 빨간색이지만 사람마다 다르게 보이는 이유
분명히 같은 영상인데 A Display에서 보이는 색상과 B Display에서 보이는 색상이 다른 것 같은 경험, 아마 한번쯤은 겪어봤을 것이다.
누군가에게는 빨간색, 누군가에게는 짙은 빨간색, 누군가에게는 짙은 핑크색 등 rgb 코드가 모두 동일한 색상임에도 불구하고,
사람마다 인지하는 색상의 차이가 발생하는 이유는 무엇일까?

A Display에서의 (255, 0, 0), B Display에서의 (255, 0, 0), 그리고 C Display에서의 (255, 0, 0)는 같은 '빨간색 디지털 코드'이지만,
측색기로 측정한 실제 값은 아래와 같이 조금씩 다른 색일 수가 있다.

![difference_red](/assets/img/image-processing/color-science/difference_red.png)

이러한 차이가 생기는 이유는 바로 각 디스플레이마다 표현하고자 하는 색역(color gamut)이 다르기 때문이다.
색역이란 display가 표현할 수 있는 모든 색들의 범위를 의미하는데, 
display마다 서로 다른 크기와 형태의 색역 때문에 컬러의 불일치가 발생하게 되는 것이다.


## Color History (feat. CIERGB, CIEXYZ, CIELab)
![color_history](/assets/img/image-processing/color-science/color_history.png)

연도는 부정확 할 수도 있습니다 :)
{:.note}

1850년, 빛의 삼원색이 정의된 이후, 빛, 조명, 물체, 그리고 보는 사람에 따라 발생하는 불일치를 해소하기 위해 표준으로 삼을 수 있는 기준이 필요하게 되었으며, 
국제조명위원회 CIE (Commission internationale de l'éclairage)가 그 기준을 발표하게 되었다.
1920년대, W. 데이빗 라이트와 존 길드가 인간의 시력에 관한 일련의 실험을 독립적으로 수행했으며, 
이를 바탕으로 CIERGB 색 공간이 만들어졌고, 이에 기반하여 다시 CIEXYZ 색 공간이 만들어졌다.

먼저, CIERGB 색상 공간은 특정 단색 (단일 파장) 원색 세트로 구별되는 많은 RGB 색상 공간 중 하나이다.
W.G.Wright과 J.Guild의 등색 실험을 통해 정의가 되었는데, 
우리 눈은 장파장(빨간색에 가까운 빛), 중파장(초록색에 가까운 빛), 단파장(파란색에 가까운 빛)을 각각 감지하는 세 종류의 원추세포가 존재하기 때문에 
"모든 색은 3원색의 조합으로 만들어 낼 수 있다." 라는 가설을 바탕으로 380nm ~ 780nm까지의 단색광을 만들어내는 실험을 말한다.

![CIERGB](/assets/img/image-processing/color-science/CIERGB.png)

위 그림은 등색 실험의 결과를 그래프화한 것이고, 이를 통해 아래와 같은 수식을 도출해 낼 수 있는데, 일단은 참고만 해두자.

$$
\begin{align}
  R &= \int_0^\infty \mathrm{I}(\lambda)\bar{r}(\lambda) \mathrm{d}x  \quad \quad \quad \quad \quad \quad  r = { R \over R + G + B } \\[2em]
  G &= \int_0^\infty \mathrm{I}(\lambda)\bar{g}(\lambda) \mathrm{d}x  \quad \quad => \quad \quad g = { G \over R + G + B } \\[2em]
  B &= \int_0^\infty \mathrm{I}(\lambda)\bar{b}(\lambda) \mathrm{d}x  \quad \quad \quad \quad \quad \quad b = { B \over R + G + B } 
\end{align}
$$

CIERGB 색상 공간에는 한 가지 문제점이 있는데, 위 그래프를 자세히 보면 하늘색으로 색칠된 음의 영역이 색 재현이 안된다는 것이다.
CIERGB의 이러한 문제점을 수학적으로 보완하여 나온 것이 CIEXYZ 색상 공간(CIE 1931 색 공간)이다.

![CIEXYZ](/assets/img/image-processing/color-science/CIEXYZ.png)


## Color Space, Color System
Wiki에 정의된 color space 및 color system.
> 색 공간(color space)은 색 표시계(color system)를 3차원으로 표현한 공간 개념이다. 
> 색 표시계의 모든 색들은 이 색 공간에서 3차원 좌표로 나타낸다.
> 여기서 색 표시계(color system)란 CIERGB, CIEXYZ, CIELAB, CIELUV 등의 색 체계를 말한다. 
> 최근 디자인 학계나 산업계에서 색채 디자인 또는 시각 디자인을 연구하거나, 카메라, 스캐너, 모니터, 컬러 프린터 등의 컬러 영상 장비 개발 및 응용 단계에서 색 공간은 정확한 색을 재현하기 위하여 필수적으로 활용되고 있다.


## 색 공간 활용

[opencv-document](https://docs.opencv.org/4.7.0/d8/d01/group__imgproc__color__conversions.html)에서 제공하는 색상 모델들
{:.note}

Lab color space에 대해 이해가 어느 정도 됐을거라 생각한다.
자 그럼 아래의 예시를 통해 어떤 색상 모델을 사용하냐에 따라 결과물이 어떻게 다르게 나오는지 한번 비교해자.
 
![watermelon](/assets/img/image-processing/color-science/watermelon.png)

위 watermelon 이미지로부터 빨간 영역만 추출하는 task를 두 가지 방법으로 해보려한다.
1. bgr 이미지로부터 r > 128 이상 이상인 부분을 masking 처리
2. lab 이미지로 변환하여 a > 128 이상인 부분을 masking 처리

~~~python
import cv2
import numpy as np


path = r'watermelon.png'

bgr = cv2.imread(path)
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

![watermelon_concat](/assets/img/image-processing/color-science/watermelon_concat.png)

이렇게 단순히 색 공간을 변환해서 처리하는 것 만으로도 확연한 차이가 있을을 알 수 있다.