# ROS와 시뮬레이션 인터페이스

autopilot을 시뮬레이션을 시작하면 14557 포트로 2번째 MAVLink 인터페이스가 시작됩니다. MAVROS를 이 포트로 연결하면 실제 비행시에 비행체가 내보내는 모든 데이터를 받을 수 있습니다.

## MAVROS 실행하기

ROS와 인터페이스를 원하면, 이미 실행된 2번째 MAVLink 인스턴스가 [mavros](ros-mavros-offboard.md)를 통해 ROS에 연결할 수 있습니다. 특정 IP(`fcu_url` is the IP / port of SITL)로 연결하고자 한다면 아래와 같은 형태로 URL을 사용합니다. :

<div class="host-code"></div>

```sh
roslaunch mavros px4.launch fcu_url:="udp://:14540@192.168.1.36:14557"
```

localhost에 연결하려면 다음 URL을 사용 :

<div class="host-code"></div>

```sh
roslaunch mavros px4.launch fcu_url:="udp://:14540@127.0.0.1:14557"
```


## ROS를 위한 Gazebo 설치

Gazebo ROS SITL 시뮬레이션은 Gazebo 6와 Gazebo 7과 동작하므로 다음과 같이 설치합니다. :

```sh
sudo apt-get install ros-$(ROS_DISTRO)-gazebo7-ros-pkgs    //Recommended
```
or
```sh
sudo apt-get install ros-$(ROS_DISTRO)-gazebo6-ros-pkgs
```

## ROS wrapper와 함께 Gazebo 실행하기

ROS topic에 직접 publish하는 센서를 통합하기 위해서 Gazebo 시뮬레이션을 수정하고 싶은 경우(Gazebo ROS laser plugin), Gazebo는 반드시 적절한 ROS wrapper와 함께 실행해야만 합니다.

ROS에서 시뮬레이션을 실행을 가능하게 하는 ROS launch 스크립트 :
* [posix_sitl.launch](https://github.com/PX4/Firmware/blob/master/launch/posix_sitl.launch): plain SITL launch

  * [mavros_posix_sitl.launch](https://github.com/PX4/Firmware/blob/master/launch/mavros_posix_sitl.launch): SITL과 MAVROS

ROS와 동작하는 SITL을 실행하기 위해서는 ROS 환경 업데이트를 하고 실행하는 것이 보통 :

(옵션): MAVROS나 다른 ROS 패키지를 소스로 컴파일하는 경우 catkin workspace만 사용해야 함

```sh
cd <Firmware_clone>
make posix_sitl_default gazebo
source ~/catkin_ws/devel/setup.bash    // (optional)
source Tools/setup_gazebo.bash $(pwd) $(pwd)/build_posix_sitl_default
export ROS_PACKAGE_PATH=$ROS_PACKAGE_PATH:$(pwd)
export ROS_PACKAGE_PATH=$ROS_PACKAGE_PATH:$(pwd)/Tools/sitl_gazebo
roslaunch px4 posix_sitl.launch
```

여러분이 가지고 있는 launch 파일에서 위에 언급한 launch 파일 중에 하나를 include해서 시뮬레이션에서 ROS application을 실행합니다.

### 실제로 일어나는 동작

(수동으로 실행하는 방법)

```sh
no_sim=1 make posix_sitl_default gazebo
```

이렇게 하면 시뮬레이터를 구동시키고 콘솔은 다음과 같이 :


```sh
[init] shell id: 46979166467136
[init] task name: px4

______  __   __    ___
| ___ \ \ \ / /   /   |
| |_/ /  \ V /   / /| |
|  __/   /   \  / /_| |
| |     / /^\ \ \___  |
\_|     \/   \/     |_/

Ready to fly.


INFO  LED::init
729 DevObj::init led
736 Added driver 0x2aba34001080 /dev/led0
INFO  LED::init
742 DevObj::init led
INFO  Not using /dev/ttyACM0 for radio control input. Assuming joystick input via MAVLink.
INFO  Waiting for initial data on UDP. Please start the flight simulator to proceed..
```

이제 새로운 터미널에서 Gazebo 메뉴를 통해 Iris model을 삽입할 수 있게 됩니다. 이렇게 하려면 적절한 `sitl_gazebo` 폴더에 포함하기 위해 환경변수를 설정합니다.

```sh
cd <Firmware_clone>
source Tools/setup_gazebo.bash $(pwd) $(pwd)/build_posix_sitl_default
```

이제 ROS와 동작을 원하는 경우 Gazebo를 구동시키고 Iris quadcopter model을 삽입합니다. 일단 Iris가 로드되면 자동으로 px4 app에 연결됩니다.

```sh
roslaunch gazebo_ros empty_world.launch world_name:=$(pwd)/Tools/sitl_gazebo/worlds/iris.world
```
