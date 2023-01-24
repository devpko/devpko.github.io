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
현재는 Basler 카메라, opencv에서 읽을 수 있는 일반적인 웹캠 및 카메라들만 지원하지만, 차차 앞으로 늘려가 볼 생각이다 :)

각 카메라에서 제공하는 api를 활용해서 제가 쓰기 편하게 만든 것일 뿐이라, 더 세세한 기능을 컨트롤하고 싶다면
[camera-apis repo](https://github.com/devpko/camera-apis)에서 필요한 부분만 쓰셔도 되고, 카메라 api를 직접 사용하시는 것을 추천드립니다 :)
python 기반으로 만들었으며, opencv를 활용한 점 참고부탁드립니다!
{:.note}


## Python Sample
전체 코드는 [camera-apis repo](https://github.com/devpko/camera-apis)에 올려놨으며, 차차 업데이트해나갈 예정이다.
git에 있는 파일들을 개인 프로젝트에 받아서 사용하시면 되며, 필요한 패키지는 requirements.txt에 있는 opencv-pyhton과 pypylon이다.

pypylon은 basler 카메라를 사용할 때 필요한 패키지이니 참고
{:.note}

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

먼저 일반적으로 지원하는 기능들을 위와 같이 abstract로 명시해 뒀다.
대부분의 카메라들이 open -> grab -> close -> release 순서로 동작을 하기에 사용하실 때에는 아래와 같은 순서로 사용하시면 된다.

~~~python
# Please change below relative path of imported files. It may vary depending on your project.
import sys
from .camera import Camera, CAMERA_SETTINGS
from .basler import Basler


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

가장 먼저 