# Raspberry Pi - ROS 설치

이 가이드는 Pixhawk를 위한 컴패니온 컴퓨터로 동작하는 Rapsberry Pi 2에 ROS-indigo를 설치하는 방법을 설명합니다.

## 전제 조건
* Raspberry Pi에 맞는 모니터, 키보드, SSH 연결 설정
* 이 가이드에서 여러분의 RPi에 Raspbian "JESSIE"가 설치되어 있다고 가정. 그렇지 않은 경우 [설치하기](https://www.raspberrypi.org/downloads/raspbian/)나 Wheezy를 Jessie로 [업그레이드](http://raspberrypi.stackexchange.com/questions/27858/upgrade-to-raspbian-jessie) 합니다.

## 설치
[가이드](http://wiki.ros.org/ROSberryPi/Installing%20ROS%20Indigo%20on%20Raspberry%20Pi)를 따라 ROS Indigo를 설치합니다. 주석: "ROS-Comm"를 설치합니다. Desktop버전은 너무 무겁습니다.

### 패키지 설치 중 에러
패키지를 다운받을려면(`sudo apt-get install ros-indigo-ros-tutorials`) 다음과 같은 에러가 발생할 수 있습니다 : "unable to locate package ros-indigo-ros-tutorials"

이런 경우 다음과 같이 조치합니다. :
catkin workspace (예로 ~/ros_catkin_ws)로 가서 패키지 이름을 변경합니다.

```sh
$ cd ~/ros_catkin_ws

$ rosinstall_generator ros_tutorials --rosdistro indigo --deps --wet-only --exclude roslisp --tar > indigo-custom_ros.rosinstall
```

다음으로 workspace를 wstool로 업데이트합니다.

```sh
$ wstool merge -t src indigo-custom_ros.rosinstall

$ wstool update -t src
```

다음으로(workspace 폴더내에서) source와 make명령을 수행합니다.

```sh
$ source /opt/ros/indigo/setup.bash

$ source devel/setup.bash

$ catkin_make
```
