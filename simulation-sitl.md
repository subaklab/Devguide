# Software in the Loop (SITL) 시뮬레이션

SITL 시뮬레이션은 호스트 머신의 시스템에서 동작하고 autopilot을 시뮬레이션합니다. 네트워크를 통해 시뮬레이터에 연결합니다. 셋업은 다음과 같이 :

{% mermaid %}
graph LR;
  Simulator-->MAVLink;
  MAVLink-->SITL;
{% endmermaid %}

## SITL 실행하기

[simulation prerequisites](starting-installing.md)는 시스템에 설치된 것을 확인한 후에 그냥 실행합니다. : 간편하게 make로 POSIX 빌드와 시뮬레이션을 실행합니다.

```sh
make posix_sitl_default jmavsim
```

PX4 shell이 나타납니다. :

```sh
[init] shell id: 140735313310464
[init] task name: px4

______  __   __    ___
| ___ \ \ \ / /   /   |
| |_/ /  \ V /   / /| |
|  __/   /   \  / /_| |
| |     / /^\ \ \___  |
\_|     \/   \/     |_/

Ready to fly.


pxh>
```

## 중요 파일들

  * [posix-configs/SITL/init](https://github.com/PX4/Firmware/tree/master/posix-configs/SITL/init)폴더에 `rcS_SIM_AIRFRAME` 이름의 시작 스크립트가 있는데 기본은 `rcS_jmavsim_iris`입니다.
  * root 파일 시스템 (`/`)은 빌드 디렉토리 내부에 위치합니다. : `build_posix_sitl_default/src/firmware/posix/rootfs/`

## 하늘로 날리기

[jMAVSim](http://github.com/PX4/jMAVSim.git) 시뮬레이터의 3D 뷰 윈도우 :

![](images/sim/jmavsim.png)

초기화를 마치고나면 시스템은 home position을 출력합니다. (`telem> home: 55.7533950, 37.6254270, -0.00`) 다음과 같이 입력하면 이륙시킵니다. :

```sh
pxh> commander takeoff
```

> **정보** 조이스틱은 QGC를 통해서 사용할 수 있습니다. 수동입력을 사용하고 싶은 경우 manual flight mode(POSCTL, position control)로 설정합니다. QGC preferences 메뉴에서 thumb joystick을 활성화시킵니다.

## Wifi 드론 시뮬레이션하기

내부 네트워크에서 Wifi를 통해 연결한 드론을 시뮬레이션하기 위한 특수 타겟 :

```sh
make broadcast jmavsim
```

시뮬레이터는 실제 드론이 하는 것처럼 자신의 주소를 내부 네트워크에 broadcast합니다.

## 확장 및 커스터마이징

시뮬레이션 인터페이스를 확장하고 커스터마이징하기 위해서는 `Tools/jMAVSim` 폴더에 있는 파일을 수정합니다. 코드는 Github에 있는 e[jMAVSim repository](https://github.com/px4/jMAVSim)를 통해서 접근할 수 있습니다.

> ** 정보 ** 빌드 시스템은 올바른 submodule인지를 모든 의존관계에 대해서 강제로 체크하는데 여기에는 시뮬레이터도 포함됩니다. 디렉토리에 있는 파일을 덮어쓰기로 변경하지는 않습니다. 하지만 이런 변경이 commit되면 새로운 commit hash로 submodule은 Firmware repo에 등록되어야 합니다. 이렇게 하기 위해서는 `git add Tools/jMAVSim` 과 변경을 commit합니다. 이렇게 하면 시뮬레이터의 GIT hash를 업데이트합니다.

## ROS와 인터페이스

시뮬레이션으로 실제 비행체의 온보드와 같은 방식으로 [interfaced to ROS](simulation-ros-interface.md)이 가능합니다.
