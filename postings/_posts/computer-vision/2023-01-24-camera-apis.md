---
layout: post
title: "[Image Processing] Camera에서 이미지 읽기"
description: >
  다양한 종류의 카메라 API 통합 interface 구현
sitemap: false
hide_last_modified: true
comments: true
---


# [Image Processing] Camera에서 이미지 읽기

## Intro
다양한 카메라를 사용해보면서 일반적으로 사용할 수 있는 interface가 있었으면 좋겠다 싶어서 간단하게 만들어 봤다.
현재는 Basler 카메라, opencv에서 읽을 수 있는 일반적인 웹캠 및 카메라들만 지원하지만, 앞으로 차차 늘려가 볼 생각이다 :)

각 카메라에서 제공하는 api를 활용해서 제가 쓰기 편하게 만든 것일 뿐이라, 더 세세한 기능을 컨트롤하고 싶다면
[camera-apis repo]에서 필요한 부분만 쓰셔도 되고, 카메라 api를 직접 사용하시는 것을 추천드립니다 :)
python 기반으로 만들었으며, opencv를 활용한 점 참고부탁드립니다!
{:.note}


## Getting Started
전체 코드는 [camera-apis repo]에 올려놨으며, 꾸준히 업데이트해나갈 예정이다.

먼저 [camera-apis repo] 페이지 오른쪽에 있는 Releases에서 최신 버전을 다운 받은 후 아래 command로 설치한다.
(환경변수가 모두 설정되어 있다는 가정)

~~~console
pip install camapislib-1.0.0-py3-none-any.whl
~~~

pypylon은 basler 카메라를 사용할 때 필요한 패키지이니 참고!  
설치가 안될 경우 다운받은 파일명이 일치하는지 확인해보고 그래도 안된다면 다른 문제(proxy? network?)일 수도 있으니 확인해보길 바란다. 
{:.note}

### sample
설치가 되었다면 아래 sample code를 통해 카메라로부터 어떻게 이미지를 읽는지 확인해보자.

대부분의 카메라들이 open -> grab -> close -> release 순서로 동작을 하기에 사용하실 때에는 아래와 같은 순서로 사용하시면 된다.

~~~python
# Please change below relative path of imported files. It may vary depending on your project.
import sys
from .camapislib import \
    Camera, \
    CAMERA_SETTINGS, \
    Default, \
    Basler


def set_camera(camera: Camera):
    camera.camera_setting(CAMERA_SETTINGS.GAIN, 0.00000)
    camera.camera_setting(CAMERA_SETTINGS.WIDTH, 1920)
    camera.camera_setting(CAMERA_SETTINGS.HEIGHT, 1200)
    

def main():
    # Set Camera Parameters before open it.
    cam = Basler()
    set_camera(camera=cam)
    
    # Open Camera
    cam.open()
    
    # If it is opened, exit process
    if not cam.is_open():
        sys.exit()
    
    # Grab image
    image = cam.read()
    
    ###########################
    # Do something with image #
    ###########################
        
    # Close and Release Camera
    cam.release()


if __name__ == '__main__':
    main()
~~~

## Details
~~~python
cam = Basler()
~~~
가장 먼저 어떤 카메라를 사용 할지 선택 후

~~~python
set_camera(camera=cam)
~~~
변경하고 싶은 카메라 셋팅이 있으면, set_camera method 내에 필요한 변경 사항들을 추가한다.

~~~python
if not cam.is_open():
    sys.exit()
~~~
그리고 카메라가 정상적으로 open이 되었는지 체크해준다. 
카메라의 물리적인 연결이 불안정하거나 다른 process에서 카메라 object를 잡고 있는 경우도 있으므로 체킹해주는 것이 좋다.

~~~python
image = cam.read()
~~~
카메라가 정상적으로 open이 되었다면 grabbing을 하고 있는 카메라로부터 이미지를 가져와 필요한 작업을 하면 된다.

~~~python
cam.release()
~~~
마지막으로 반드시 카메라 object를 release해주는 것이 중요!


## Notes
- 일반적으로 지원하는 기능들을 아래와 같이 abstract로 명시해 뒀으니 다른 카메라를 추가해서 사용하고 싶으면,
해당 abstract class를 상속받아 추가하면 된다.

~~~python
from abc import *


class Camera(metaclass=ABCMeta):
    def __init__(self, cam) -> None:
        self.camera = cam

    @abstractmethod
    def camera_setting(self, target, value):
        pass

    @abstractmethod
    def get_serial_number(self):
        pass

    @abstractmethod
    def open(self):
        pass

    @abstractmethod
    def close(self):
        pass

    @abstractmethod
    def is_open(self):
        pass

    @abstractmethod
    def read(self):
        pass

    @abstractmethod
    def release(self):
        pass
~~~

- Git에 구현해 놓은 바슬라 카메라 관련 코드의 경우 camera를 open 해놓고 계속 grabbing 하는 방식인데, 
계속 grabbing을 하지 않고 필요할 때만 하는 것이 개인적으로는 좋다고 생각한다. 
[camera-apis repo]에서는 영상 저장과 image processing을 병렬 처리하기 위함이니 참고 부탁드립니다.


<!-- links -->
[camera-apis repo]: https://github.com/devpko/camera-apis


---
#### *Reference*
- https://github.com/basler/pypylon
---