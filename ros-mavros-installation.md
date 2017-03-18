
# MAVROS

 [mavros](http://wiki.ros.org/mavros#mavros.2BAC8-Plugins.sys_status) ros패키지는 ROS를 실행하고 있는 컴퓨터, MAVLink를 사용하는 autopilot, MAVLink를 사용하는 GCS 사이에서 MAVLink로 확장가능한 통신을 할 수 있습니다. MAVRos는 MAVLink가 가능한 autopilot과 통신할 수 있지만 이 문서에서는 PX4 flight stack과 ROS를 사용할 수 있는 컴패니온 컴퓨터 사이에 통신을 가능하게 하는 내용을 다룹니다.

## 설치

MAVROS는 소스나 바이너리 형태로 설치가 가능합니다. 개발자가 ROS로 개발하는 경우라면 소스로 설치하는 것을 추천합니다.

### 바이너리 설치 (Debian / Ubuntu)

v0.5이후로 x86과 amd64(x86\_64)용으로 미리 컴파일된 debian 패키지.
Ubuntu armhf용 ARMv7 repo에 v0.9+ 부터는 제공.
설치는 `apt-get`를 사용 :
```sh
$ sudo apt-get install ros-indigo-mavros ros-indigo-mavros-extras
```

### 소스 설치
**Dependencies**
`~/catkin_ws`에 catkin workspace가 있다는 가정합니다.
만약 없다면 다음과 같이 생성 :

```sh
$ mkdir -p ~/catkin_ws/src
$ cd ~/catkin_ws
$ catkin init
```

`wstool, rosinstall,and catkin_tools`와 같은 ROS python tool을 사용합니다. ROS설치할 때 설치될 수도 있지만 다음과 같이 직접 설치 :
```sh
$ sudo apt-get install python-wstool python-rosinstall-generator python-catkin-tools
```

해당 패키지는 catkin_make을 사용해서 빌드할 수 있지만 더 나은 방법은 catkin_tools를 이용하는 것입니다. 이렇게 하는 것이 좀더 다양하고 친근하게 빌드할 수 있는 도구입니다.

wstool을 사용해서 처음 빌드하는 경우라면 소스 위치를 초기화해 주어야 합니다. :
```sh
$ wstool init ~/catkin_ws/src
```

이제 빌드할 준비가 갖춰졌습니다.
```sh
    # 1. get source (upstream - released)
$ rosinstall_generator --upstream mavros | tee /tmp/mavros.rosinstall
    # alternative: latest source
$ rosinstall_generator --upstream-development mavros | tee /tmp/mavros.rosinstall

    # 2. get latest released mavlink package
    # you may run from this line to update ros-*-mavlink package
$ rosinstall_generator mavlink | tee -a /tmp/mavros.rosinstall

    # 3. Setup workspace & install deps
$ wstool merge -t src /tmp/mavros.rosinstall
$ wstool update -t src
$ rosdep install --from-paths src --ignore-src --rosdistro indigo -y

    # finally - build
$ catkin build
```
<aside class="note">
raspberry pi에 mavros를 설치하는 경우라면 "rosdep install ..."을 실행하는 중에 현재 설치된 OS와 관련된 에러 메시지가 발생할 수 있습니다. "--os=OS_NAME:OS_VERSION "를 rosdep 명령에 추가하고 OS_NAME과 OS_VERSION을 사용하는 OS저보에 맞게 수정합니다.
</aside>
