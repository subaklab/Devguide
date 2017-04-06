# ecl EKF 사용하기
이 튜토리얼에서는 ECL EKF 알고리즘 사용과 관련된 일반적인 질문과 그에 대한 답변을 드립니다.

## ecl EFK가 뭔가요?
ECL (Estimation and Control Library)은 확장 칼만 필터(Extended Kalman Filter)를 사용하여 센서 측정을 처리하고 다음 상태를 추측(estimate)하는 기능을 제공합니다:

* 쿼터니언(Quaternion)은 북, 동, 아래(North, East, Down)와 같은 local earth 프레임을 X, Y, Z와 같은 body frame으로의 rotation을 정의합니다.
* IMU에서 속도 - North, East, Down (m/s)
* IMU에서 위치 - Northm, East, Down (m)
* IMU delta angle bias estimates - X,Y,Z (rad)
* IMU delta velocity bias estimates - X,Y,Z(m/s)
* Earth Magnetic field components - North,East,Down (gauss)
* Vehicle body frame magnetic field bias - X,Y,Z (gauss)
* 바람 속도 - North,East (m/s)

EKF는 IMU에 관련해서 각 측정마다 다른 지연시간을 처리하기 위해서 지연된 'fusion time horizon'에서 실행합니다. 각 센서로부터 데이터는 FIFO 버퍼에 쌓이며 EKF가 버퍼에서 제때 사용하기 위해서 꺼내갑니다. 각 센서에 대한 지연 보상은 EKF2_*_DELAY 파라미터로 제어합니다.

상보필터는 버퍼에 쌓인 IMU 데이터를 사용하기 위해서 'fusion time horizon'에서 현재 시간으로 상태를 전파시키는데 사용합니다. 이 필터에 대해서 시간 상수는 EKF2_TAU_VEL과 EKF2_TAU_POS 파라미터로 제어합니다.

노트: 'fusion time horizon' 지연과 버퍼의 크기는 가장 큰 EKF2_*_DELAY 파라미터로 결정합니다. 만약 센서를 사용하지 않는 경우, 지연 시간을 0으로 설정하는 것을 추천합니다. 'fusion time horizon' 지연을 줄이면 상보필터에서 현재 시간으로 상태를 전파시키는데 있어 에러를 줄입니다.

위치와 속도 상태는 IMU와 바디 프레임 사이에 offset으로 조정합니다. 즉 이 상태들이 제어 루프에 출력으로 되기 전이어야 합니다. 바디 프레임과 관련된 IMU의 위치는 EKF2_IMU_POS_X,Y,Z 파라미터로 설정됩니다.

EKF는 단지 상태 예측만을 위해 IMU 데이터를 사용합니다.
