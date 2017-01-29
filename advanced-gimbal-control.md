# 짐벌 컨트롤 셋업

PX4는 다른 입력과 출력 방식의 마운트/짐벌 컨트롤 드라이버를 포함하고 있다. 입력을 출력으로 선택할 수 있다.

먼저 드라이버가 실행되는지 확인한다. 이를 위해 `vmount start`를 사용한다. 다음으로 파라미터를 설정한다.

## 파라미터

[src/drivers/vmount/vmount_params.c](https://github.com/PX4/Firmware/blob/master/src/drivers/vmount/vmount_params.c)에 파라미터들이 있습니다. 가장 중요한 것은 input (`MNT_MODE_IN`) 과 output (`MNT_MODE_OUT`) 모드입니다. 기본으로 input은 비활성화되어 있습니다. input 방식은 유효한 output------
The most important ones are the input (`MNT_MODE_IN`) and the output (`MNT_MODE_OUT`) mode. By default, the input is disabled. Any input method can be selected to drive any of the available outputs.

mavlink input mode가 선택된 경우, manual RC input은 추가 (`MNT_MAN_CONTROL`)에서 활성화될 수 있습니다. mavlink 메시지를 아직 받지 않은 경우거나 mavlink가 명시적으로 RC mode를 요청한 경우에 활성화됩니다.



### AUX 출력을 위한 짐벌 mixer 설정

짐벌은 control group #2 (참고 : [Mixing and Actuators](concept-mixing.md))를 사용한다. mixer 설정은 다음과 같다 :

```
# roll
M: 1
O:      10000  10000      0 -10000  10000
S: 2 0  10000  10000      0 -10000  10000

# pitch
M: 1
O:      10000  10000      0 -10000  10000
S: 2 1  10000  10000      0 -10000  10000

# yaw
M: 1
O:      10000  10000      0 -10000  10000
S: 2 2  10000  10000      0 -10000  10000
```

main 혹은 추가 mixer에 필요에 따라 추가합니다.

## 테스팅

드라이버는 간단한 테스트 명령을 제공합니다. - 먼저 `vmount stop`으로 동작을 멈춥니다. 다음은 SITL에서 테스팅하는 것을 설명합니다. 실제 장치에서도 명령은 동작합니다.

시뮬레이션을 시작하기(이를 위해 파라미터를 변경할 필요는 없습니다.):
```
make posix gazebo_typhoon_h480
```
arm 상태가 되었는지 확인합니다. (`commander takeoff` 사용) 다음으로 아래 예제를 사용해서 짐벌제어하기 :
```
vmount test yaw 30
```
가상의 짐벌은 스스로 균형을 잡으며, mavlink 명령을 보내면 'stabilize' flag가 false로 설정됩니다.

![Gazebo 짐벌 시뮬레이션](images/gazebo-gimbal-simulation.png)
