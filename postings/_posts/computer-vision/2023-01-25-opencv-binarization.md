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
Binarization(이진화)는 영상처리에서 가장 기본적이면서도 제일 중요한 task이다.
이번 포스팅에서는 opencv에서 제공하는 binarization 방법들에 대해 설명하고,
어떤 상황에서 어떤 binarization 기법을 사용하는 것이 효율적일지에 대해 설명하고자 한다.


## Sample
먼저 opencv로 이진화를 하기 위해서는 단일 채널 이미지로 변경한 후 아래와 같이 이진화를 적용하면 된다.

~~~python
image = cv2.imread('some/path/image.jpg')
gray = cv2.cvtColor(image, cv2.COLOR_BGR2GRAY)
_, thresh = cv2.thresh(gray, 0, 255, cv2.THRESH_BINARY)
~~~


---
#### *Reference*
- dd
---
