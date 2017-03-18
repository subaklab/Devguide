# Gazebo 시뮬레이션

[Gazebo](http://gazebosim.org)는 자동화 로봇을 위한 3D 시뮬레이션 환경입니다. ROS없이 사용할 수 있고 SITL + ROS 조합으로도 사용할 수 있습니다.

{% youtube %}https://www.youtube.com/watch?v=qfFF9-0k4KA&vq=hd720{% endyoutube %}


{% mermaid %}
graph LR;
  Gazebo-->Plugin;
  Plugin-->MAVLink;
  MAVLink-->SITL;
{% endmermaid %}

## 설치

Gazebo와 시뮬레이션 plugin을 설치합니다.

> ** 주석 ** Gazebo 버전 7을 추첩합니다.(최소 버전은 Gzebo 6) Linux에서 ROS의 버전이 Jade이전 버전을 사용한다면 번들로 제공되는 Gazebo 버전은 너무 옛날 버전이라 삭제합니다. (sudo apt-get remove ros-indigo-gazeo명령 사용)

### Mac OS

Mac OS에서 Gazebo 7을 사용할려면 xquartz와 OpenCV가 필요합니다.

```sh
brew cask install xquartz
brew install homebrew/science/opencv
brew install gazebo7
```

### Linux

PX4 SITL은 Gazebo 시뮬레이터를 사용하지만 ROS가 꼭 있어야 하는 것은 아닙니다. 시뮬레이션은 일반 flight code와 동일한 방식으로 [interfaced to ROS](simulation-ros-interface.md)로 연결됩니다.

#### ROS Users

ROS와 함께 PX4를 사용한다면, [Gazebo version guide for version 7](http://gazebosim.org/tutorials?tut=ros_wrapper_versions#Gazebo7.xseries) 을 참고합니다.

#### 일반적인 설치

Gazebo 7을 [Linux에 설치 방법](http://gazebosim.org/tutorials?tut=install_ubuntu&ver=7.0&cat=install)을 참고합니다.

모두 설치 : `gazebo7`와 `libgazebo7-dev`.

## 시뮬레이션 실행하기

PX4 펌웨어의 소스 디렉토리 내부에서 PX4 SITL을 airframe들(Quad, planes, VTOL 그리고 optical flow포함) 중에 하나와 실행하기:

> **주석** Gazebo 실행을 유지하면서 PX4를 재실행하는 방법은 아래 참고

### Quadrotor

```sh
cd ~/src/Firmware
make posix_sitl_default gazebo
```

### Quadrotor (Optical Flow 기능 사용)

```sh
make posix gazebo_iris_opt_flow
```

### 3DR Solo

```sh
make posix gazebo_solo
```

![](/assets/gazebo_solo.png)

### Standard Plane

```sh
make posix gazebo_plane
```

![](/assets/gazebo_plane.png)

### Standard VTOL

```sh
make posix_sitl_default gazebo_standard_vtol
```

![](/assets/gazebo_standard_vtol.png)

### Tailsitter VTOL

```sh
make posix_sitl_default gazebo_tailsitter
```

![](/assets/gazebo_tailsitter.png)

## World 바꾸기

현재 기본 world는 iris.world로  [worlds](https://github.com/PX4/sitl_gazebo/tree/367ab1bf55772c9e51f029f34c74d318833eac5b/worlds)에 위치하고 있습니다. iris.world에서 기본적으로 지면과 같이 heightmap을 사용합니다. 여기서의 지면은 거리 센서를 사용하는 경우 복잡해질 수 있습니다. heightmap에서 의도하지 않은 동작을 한다면 iris.model의 모델에서 uneven_ground을 asphalt_plane로 변경하는 것을 추천합니다.

## 이륙시키기

> ** 주석 ** 실행 중에 에러가 발생하면 [Installing Files and Code](http://dev.px4.io/starting-installing-mac.html) 가이드를 참고합니다.

PX4 shell이 나타납니다 :

```sh
[init] shell id: 140735313310464
[init] task name: px4

______  __   __    ___
| ___ \ \ \ / /   /   |
| |_/ /  \ V /   / /| |
|  __/   /   \  / /_| |
| |     / /^\ \ \___  |
\_|     \/   \/     |_/

px4 starting.


pxh>
```

> ** Note ** Right-clicking the quadrotor model allows to enable follow mode from the context menu, which is handy to keep it in view.

![](images/sim/gazebo.png)

초기화를 마치고나면 시스템은 home position을 출력합니다. (`telem> home: 55.7533950, 37.6254270, -0.00`) 다음과 같이 입력하면 이륙시킵니다. :

```sh
pxh> commander takeoff
```

> ** 주석 ** 조이스틱은 QGC를 통해서 사용할 수 있습니다. 수동입력을 사용하고 싶은 경우 manual flight mode(POSCTL, position control)로 설정합니다. QGC preferences 메뉴에서 thumb joystick을 활성화시킵니다.

## Gazebo와 PX4를 따로 구동시키기

개발을 진행 중에는 Gazebo와 PX4를 따로 구동시키는 것이 더 편리할 수 있습니다.

px4가 올바른 model을 로드할 수 있도록 하기 위해 parameter로 `sitl_run.sh`를 실행하는 기존 cmake target에 추가로 `px4_<mode>` 라는 이름의 launcher target을 생성합니다. 이것은 원래 sitl px4 app의 얇은 wrapper역할을 합니다. 이런 얇은 wrapper는 현재 작업 디렉토리와 같이 단순히 app 인자와 model 파일 경로를 포함시킵니다.

### 사용 방법

  * gazebo server(아니면 다른 시뮬레이터)와 client viewer를 터미널로 실행 :
```
make posix_sitl_default gazebo_none_ide
```
  * IDE에서 디버깅을 원하는 `px4_<mode>` 타겟을 선택 (예로 `px4_iris`)
  * IDE에서 직접 디버그 세션 구동
이 방법은 시뮬레이터가 항상 백그라운드로 실행되기 때문에 px4 프로세스만 재실행하므로 부담이 줄어 디버그 사이클 타임을 크게 줄일 수 있습니다.

## 확장과 커스터마이징

시뮬레이션 인터페이스를 확장하고 커스터마이징하기 위해서 `Tools/sitl_gazebo` 폴더에 파일을 수정합니다. Github에 [sitl_gazebo repository](https://github.com/px4/sitl_gazebo)에서 코드를 확인할 수 있습니다.

> ** 주석 ** 빌드 시스템은 올바른 submodule인지를 모든 의존관계에 대해서 강제로 체크하는데 여기에는 시뮬레이터도 포함됩니다. 디렉토리에 있는 파일을 덮어쓰기로 변경하지는 않습니다.

## ROS와 인터페이스

시뮬레이션으로 실제 비행체의 온보드와 같은 방식으로 [interfaced to ROS](simulation-ros-interface.md)이 가능합니다.
