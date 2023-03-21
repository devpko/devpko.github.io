---
layout: post
title: "[Image Processing] OpenCV Binarization, 이진화 활용 방법"
description: >
  OpenCV에서 제공하는 이진화 활용 방법
sitemap: false
hide_last_modified: true
comments: true
---


# [Image Processing] OpenCV Binarization, 이진화 활용 방법

## Intro
Binarization이란 이미지를 이진화(Binarization)하여 밝기 값이 특정 임계값 이상인 픽셀을 흰색(255)으로, 이하인 픽셀을 검은색(0)으로 변환하는 과정이다.
Computer Vision 분야에서 Binarization은 가장 기본적이면서도 제일 중요한 task 중 하나이다. 
이진화는 이미지에서 필요한 정보를 추출하고, 이미지 처리 및 분석에 유용한 다양한 알고리즘을 적용하는 데 매우 중요하다. 
이번 포스팅에서는 opencv에서 제공하는 binarization 방법들에 대해 설명하고,
어떤 상황에서 어떤 binarization 기법을 사용하는 것이 효율적일지에 대해 설명하고자 한다.


## Binarization Methods
1. Global Thresholding  
Global Thresholding은 이미지 전체에 대한 하나의 임계값을 사용하여 이미지를 이진화하는 것이다. 
OpenCV에서는 cv2.threshold() 함수를 사용하여 Global Thresholding을 수행하며, 
입력으로 이미지, 임계값, 흰색 픽셀 값, 이진화 유형 등의 parameter들을 사용한다.
~~~python
image = cv2.imread('some/path/image.jpg')
gray = cv2.cvtColor(image, cv2.COLOR_BGR2GRAY)
_, thresh = cv2.thresh(gray, 0, 255, cv2.THRESH_BINARY)
~~~


3. Adaptive Thresholding  
Adaptive Thresholding은 이미지의 작은 영역에 대해 적응적으로 임계값을 계산하고, 각 영역을 이진화하는 것이다. 
이 기술은 이미지의 광도가 일정하지 않을 때 효과적이다. 
OpenCV에서는 cv2.adaptiveThreshold() 함수를 사용하여 Adaptive Thresholding을 수행하며,
입력으로 이미지, 흰색 픽셀 값, 이진화 유형, 블록 크기 등의 parameter들을 사용한다.
~~~python
image = cv2.imread('some/path/image.jpg')
gray = cv2.cvtColor(image, cv2.COLOR_BGR2GRAY)

block_size = 9
C = 5
thresh = cv2.adaptiveThreshold(img, 255, cv2.ADAPTIVE_THRESH_MEAN_C, cv2.THRESH_BINARY, block_size, C)
thresh = cv2.adaptiveThreshold(img, 255, cv2.ADAPTIVE_THRESH_GAUSSIAN_C, cv2.THRESH_BINARY, block_size, C)
~~~

5. Otsu's Binarization  
Otsu's Binarization은 이미지의 임계값을 자동으로 결정하는 알고리즘이다. 
Otsu's Binarization은 이미지의 히스토그램에서 bimodal distribution을 찾아내고, 이를 기반으로 최적의 임계값을 결정한다. 
아래 그림에서 볼 수 있듯이 제일 하단의 histogram처럼 bimodal distribution 일수록 이진화가 선명해 지는 것을 볼 수 있다. 
이 알고리즘은 입력 이미지와 임계값을 사용하여 cv2.threshold() 함수를 호출하여 수행된다.
![otsu_algorithm](/assets/img/computer-vision/opencv-binarization/otsu_algorithm.png)
~~~python
image = cv2.imread('some/path/image.jpg')
gray = cv2.cvtColor(image, cv2.COLOR_BGR2GRAY)
_, thresh = cv2.thresh(gray, 0, 255, cv2.THRESH_BINARY | cv2.THRESH_OTSU)
~~~

6. Triangle Binarization  
Triangle Binarization은 Otsu's Binarization과 유사한 방법으로 이미지의 임계값을 자동으로 결정한다. 
이 알고리즘은 이미지의 히스토그램에서 최대 거리를 찾아내고, 이를 기반으로 최적의 임계값을 결정한다. 
OpenCV에서는 cv2.threshold() 함수의 인수로 cv2.THRESH_TRIANGLE 값을 사용하여 Triangle Binarization을 수행한다.
![triangle_algorithm](/assets/img/computer-vision/opencv-binarization/triangle_algorithm.png)
~~~python
image = cv2.imread('some/path/image.jpg')
gray = cv2.cvtColor(image, cv2.COLOR_BGR2GRAY)
_, thresh = cv2.thresh(gray, 0, 255, cv2.THRESH_BINARY | cv2.THRESH_TRIANGLE)
~~~

## Otsu vs Triangle
**Otsu's Binarization**은 이미지의 히스토그램이 bimodal distribution인 경우에 특히 효과적이다. 
즉, 이미지의 밝기 값이 두 그룹으로 나뉘어져 있는 경우이다. 
예를 들어, 전경과 배경의 색감이 명확히 구분되는 이미지, 물체와 배경이 분리된 이미지 등에서 Otsu's Binarization을 사용하면 좋은 결과를 얻을 수 있다.

**Triangle Binarization**은 Otsu's Binarization과 유사하지만, 이미지의 히스토그램이 bimodal distribution이 아닌 경우에도 효과적이다.
이 알고리즘은 이미지의 히스토그램에서 최대 거리를 찾아 최적의 임계값을 결정하기 때문에, 
이미지의 밝기 분포가 불규칙하게 변하는 경우에도 좋은 결과를 얻을 수 있다. 
예를 들어, 이미지의 밝기 값이 서서히 변하는 경우, 이미지의 밝기 분포가 단일 피크인 경우 등에서 Triangle Binarization을 사용하면 좋은 결과를 얻을 수 있다.

이러한 Binarization 알고리즘들은 이미지 처리 및 분석에서 매우 중요한 역할을 한다. 
각 알고리즘은 다른 장단점을 가지고 있으므로, 이미지의 특성에 맞게 적절한 알고리즘을 선택하여 사용하는 것이 중요하다.

특정 이미지를 이진화 했을 때 잘 구분이 안되는 이미지는 먼저 histogram으로 시각화하여 밝기 분포를 파악하고, 
이를 기반으로 적절한 Binarization 알고리즘을 선택하는 것이 좋다.


---
#### *Reference*
- https://docs.opencv.org/4.x/d7/d4d/tutorial_py_thresholding.html
- https://journals.sagepub.com/doi/pdf/10.1177/25.7.70454
---
