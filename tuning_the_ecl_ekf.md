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
