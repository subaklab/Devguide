# Optical Flow (Snapdragon Flight에서)

Snapdragon Flight 보드는 position stabiliztion 기반으로 optical flow에 사용할 수 있는 아래 방향을 찍는 그레이 스케일 카메라가 있습니다.

카메라 외에, optical flow는 아래 방향으로의 거리 센서가 필요합니다. 여기 TeraRanger One을 사용하는 방법에 대해서 알아봅니다.


## TeraRanger One 셋업
TeraRanger One (TROne)을 Snapdragon Flight에 연결하기 위해서 TROne I2C 어댑터를 반드시 사용해야 합니다. TROne은 반드시 벤더가 제공하는 I2C 펌웨어로 flash해야 합니다.

TROne을 Snapdragon Flight에 연결할 때는 커스텀 DF13 4-to-6 핀 케이블을 사용합니다. 연결은 다음과 같습니다. :

| 4 pin | <-> | 6 pin |
| -- | -- | -- |
| 1 |  | 1 |
| 2 |  | 6 |
| 3 |  | 4 |
| 4 |  | 5 |

TROne은 10 - 20V 전압이 필요합니다.

## Optical flow
optical flow는 application processor에서 계산을 하고 Mavlink로 PX4에 전달됩니다. clone하고 [snap_cam](https://github.com/PX4/snap_cam) repo를 컴파일합니다. readme 문서를 참고하세요.

optical flow application을 root로 실행 :
````
optical_flow -n 50 -f 30
```

optical flow application은 PX4로부터 IMU Mavlink 메시지를 필요로 합니다. `px4.config`에 아래와 같이 추가하면 PX4에 추가로 Mavlink instance를 추가할 수도 있습니다. :

```
mavlink start -u 14557 -r 1000000 -t 127.0.0.1 -o 14558
mavlink stream -u 14557 -s HIGHRES_IMU -r 250
```
