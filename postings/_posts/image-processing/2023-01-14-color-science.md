---
layout: post
title: Color Science
description: >
  다양한 색 공간에 대한 이해 및 활용 방법.
sitemap: false
hide_last_modified: true
---

# Color Science

## 색 공간에 대한 이해의 필요성
우리가 일반적으로 이미지 처리를 하는데 있어서 가장 많이 사용하고 있는 color space로는 rgb, hsv 정도일 것이다.
하지만 우리가 실무에서 활용할 수 있는 다양한 색 공간

watermelon image

위 이미지로부터 red와 green을 두 가지 방법으로 분류해보려한다.
1. bgr 이미지로부터 r > 128 이상, g > 128 이상인 부분을 각각 masking 처리
2. lab 이미지로 변환하여 a > 128 이상, b > 128 이상인 부분을 각각 masking 처리

~~~python
import cv2


path = r''
bgr = cv2.imread(path)
b, g, r = cv2.split(bgr)

lab = cv2.cvtColor(bgr, cv2.COLOR_BGR2Lab)
l, a, b = cv2.split(bgr)
~~~

결과 image

이렇게 단순히 색 공간을 변환하는 것 만으로도 확연한 차이가 있을을 알 수 있다.
이것이 이미지 프로세싱을 하는데 있어서 색 공간에 대한 이해가 필요한 이유이다.


## Color Space, Color System
Wiki에 정의된 color space 및 color system 은 아래와 같다.

> 색 공간(color space)은 색 표시계(color system)를 3차원으로 표현한 공간 개념이다. 
> 색 표시계의 모든 색들은 이 색 공간에서 3차원 좌표로 나타낸다.
> 여기서 색 표시계(color system)란 CIERGB, CIEXYZ, CIELAB, CIELUV 등의 색 체계를 말한다. 
> 최근 디자인 학계나 산업계에서 색채 디자인 또는 시각 디자인을 연구하거나, 카메라, 스캐너, 모니터, 컬러 프린터 등의 컬러 영상 장비 개발 및 응용 단계에서 색 공간은 정확한 색을 재현하기 위하여 필수적으로 활용되고 있다.


## Color History

