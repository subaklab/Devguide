# State Estimator 스위칭하기
이 페이지에서는 어떤 state estimator가 유효한지 그리고 어떻게 estimator 간에 스위치하는 방법에 대해서 알아봅시다.

## 유효한 estimator

**1. Q attitude estimator**

attitude Q estimator는 attitude에 대한 아주 간단한 quaternion 기반의 상보필터(complementary filter)입니다.

**2. INAV position estimator**

INAV position estimator는 3D position과 velocity state에 대한 상보필터입니다.


**3. LPE position estimator**

LPE position estimator는 3D position과 velocity state에 대한 확장 칼만필터(extended kalman filter)입니다.

**4. EKF2 attitude, position and wind states estimator**

EKF2는 attitude, 3D position / velocity, wind state에 대한 확장 칼만필터입니다.

**4. EKF attitude, position and wind states estimator (depricated)**
EKF2와 유사한 확장 칼만필터입니다. 하지만 조만간 EKF2로 완전히 대체될 예정입니다.
이 필터는 고정날개형에서만 사용됩니다.

## 다른 estimator를 활성화시키는 방법

멀티로터와 VTOL은 **SYS_MC_EST_GROUP** 파라미터를 사용해서 다음 설정중에서 선택합니다.
<aside class="tip">
현재 사용을 권장하지 않는 EKF estimator는 비행기 타입에서만 사용합니다. 조만간 EKF2로 대체될 예정입니다.
</aside>


| SYS_MC_EST_GROUP | Q Estimator| INAV | LPE | EKF2 |
| --- | --- | --- | --- | --- |
| 0 | enabled | enabled | | |
| 1 | enabled |  | enabled | |
| 2 |  |  | | enabled |
