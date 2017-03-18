# Optical Flow Outdoors
----------------------------------------------------

이 페이지에서는 position estimation과 아웃도어 자동비행을 위해 PX4Flow를 설정하는 방법에 대해서 알아봅니다. LIDAR 장치를 반드시 이용할 필요는 없지만 LIDAR를 이용하면 성능을 개선합니다.

## LPE Estimator 선택
--------------------------------------------------------

아웃도어 자동비행을 기반으로 optical flow와 동작을 테스트하는 유일한 estimator는 LPE입니다.

`SYS_MC_EST_GROUP = 1` parameter를 이용해서 estimator를 선택하고 난 후 reboot합니다.


## Hardware
--------------------------------------------------------

![](images/hardware/px4flow_offset.png)

*그림 1: Mounting Coordinate Frame (아래 parameter 관련)*

![](images/hardware/px4flow.png)

*그림 2: PX4Flow optical flow 센서 (카메라와 sonar)*

PX4Flow는 지면을 향해야 합니다. pixhawk에 있는 I2C 포트를 사용해서 연결할 수 있습니다. 최고 성능을 위해서 PX4Flow가 올바른 위치에 부탁되어 있는지 확인하고 진동에 노출되지 않도록 합니다. (쿼드로터의 아래쪽을 선호)

**Note: 기본 방향은 PX4Flow sonar 쪽(flow에서 +Y)이 비행체에서 +X(정면) 방향을 가리켜야 합니다. 만약 그렇지 않은 경우 SENS_FLOW_ROT을 적절하게 설정해야 합니다. **

![](images/hardware/lidarlite.png)

*그림 3: Lidar Lite*

Lidar-Lite와 sf10a를 포함해서 여러 가지 LIDAR 옵션이 있습니다. [sf10a](http://www.lightware.co.za/shop/en/drone-altimeters/33-sf10a.html) LIDAR-Lite 연결에 관해서는 [this](https://pixhawk.org/peripherals/rangefinder?s[]=lidar)를 참고하세요. sf10a는 시리얼 케이블을 이용해서 연결할 수 있습니다.
![](images/hardware/flow_lidar_attached.jpg)


*그림: PX4Flow/ Lidar-Lite를 DJI F450에 장착*
![](images/flow/flow_mounting_iris.png)

*그림: Iris+는 LIDAR 없이 PX4Flow가 부착되어 있으며 LPE estimator가 동작합니다.*

![](images/flow/flow_mounting_iris_2.png)

*그림: 여기 flow unit에 대해서 전천후로 사용할 수 있게 구성하였습니다. 프로펠러 노이즈를 sonar가 입력으로 받을 수 있어서 이를 줄이기 위해서 sonar 주변을 폼으로 감쌀 수 있고 충돌시 카메라 렌즈를 보호하는데 도움이 됩니다.*


### Focusing Camera

좋은 optical flow 품질을 얻으려면, PX4Flow에 있는 카메라 초점을 원하는 비행 높이로 맞춰주는게 중요합니다. 카메라 초점을 맞추기 위해서 글자가 있는 대상을 두고 PX4Flow를 usb로 연결하고 QGroundControl를 실행합니다. setting 메뉴에서 PX4Flow을 선택하고 카메라 이미지를 살펴봅니다. 초점은 초점 나사를 조였다 풀었다하여 초점을 맞춰줍니다.

**주석: 만약 3m 이상 높이로 날면 카메라는 초점을 무한대가 되므로 더 높은 곳에 대해서 변경해주지 않아도 됩니다.**

![](images/flow/flow_focus_book.png)

*Figure: 원하는 비행 높이로 flow 카메라 초점을 맞추기 위해 책을 사용합니다. 일반적으로 1-3 미터입니다. 3미터가 넘으면 카메라는 초점이 무한대가 되므로 더 높은 고도에 대해서 동작합니다.*


![](images/flow/flow_focusing.png)

*Figure: QGroundControl에 있는 px4flow 인터페이스로 카메라 초점을 맞추기*

### Sensor Parameters

모든 parameter를 QGroundControl에서 변경가능합니다.
* SENS_EN_LL40LS
	1로 설정하면 lidar-lite 거리 측정을 사용 가능
* SENS_EN_SF0X
	1로 설정하면 lightware 거리 측정을 사용 가능 (예로 sf02와 sf10a)

## Local Position Estimator (LPE)
--------------------------------------------------------

LPE는 position과 velocity 상태에 대한 EKF 기반 estimator입니다. inertial navigation을 사용하며 INAV estimator와 유사하지만 상태 변화에 따라서 동적으로 Kalman gain을 계산합니다. 오류를 검출할 수 있어서 부드러운 표면에서 유효하지 않은 값을 읽는 sonar와 같은 센서에 적합합니다.

### Flight Video Outdoor
{% youtube %}https://www.youtube.com/watch?v=Ttfq0-2K434{% endyoutube %}

아래 autonomous mission의 플롯은 optical flow를 이용한 위의 야외 비행에서 얻어왔습니다. GPS는 비행체 position을 estimate하는데 사용하지는 않지만 지면의 실제 이동을 비교하는데 사용합니다. GPS와 flow 데이터 사이의 offset은 처음 놓여진 위치에서 사용자 에러로부터 estimator 초기화하기 때문입니다. 초기 위치는 LPE_LAT와 LPE_LON로 가정합니다.(아래 설명)

![](images/lpe/lpe_flow_vs_gps.png)

*Figure 4: optical flow와 sonar로 LPE 기반의 autonomous mission*


### Parameters

센서들이 연결되어 있으면 local position estimator는 자동으로 LIDAR와 optical flow 데이터를 퓨징합니다.

* LPE_FLOW_OFF_Z - 기체의 무게중신으로부터 optical flow 카메라의 offset. 기본값은 0이로 positive down을 측정. 대부분 설정에서 0으로 두는 이유는 z offset은 무시할 수준이기 때문.
* LPE_FLW_XY - Flow 표준 편차 (in meters)
* LPW_FLW_QMIN - 측정 수용을 위한 최소한의 품질
* LPE_SNR_Z - sonar 표준편차 (in meters)
* LPE_SNR_OFF_Z - 무게 중심으로부터의 sonar 센서의 offset
* LPE_LDR_Z - Lidar 표준 편차 (in meters)
* LPE_LDR_Z_OFF - 무게 중심으로부터의 lidar의 offset
* LPE_GPS_ON - LPE_GPS_ON이 1로 설정되어 있는 경우 GPS없이는 비행이 불가능함. 설정을 비활성화시키던가 아니면 GPS altitude가 초기화되도록 기다려야함. GPS가 유효한 경우에는 baro altitude보다 GPS altitude가 우선순위가 높다는 뜻.

**주석: GPS없이 비행하려면 LPE_GPS_ON은 반드시 0으로 설정되어야 함 **

### Autonomous Flight Parameters

*비행체에게 어디에 위치하고 있는지 알려주기*

* LPE_LAT - local frame에서 (0,0) 좌표에 관한 latitude
* LPE_LON - local frame에서 (0,0) 좌표에 관한 longitude

*비행체가 낮은 고도와 느린 속도를 유지하게 만들기*

* MPC_ALT_MODE - 이 값을 1로 설정하면 지형에 따르도록
* LPE_T_Z - 이는 지형 처리 noise. 언덕이 많은 환경이라면 이를 0.1로 설정하고 평평한 주차장이라면 0.01로 설정
* MPC_XY_VEL_MAX - 이 값을 2로 설정해서 기우는 것을 제한
* MPC_XY_P - 대략 0.5로 줄여서 기우는 것을 제한
* MIS_TAKEOFF_ALT - 이 값을 2 미터로 설정하여 낮은 고도로 이륙하도록 설정.

*Waypoints*

* 고도를 3 미터나 그 이하로 waypoint를 생성
* 아주 원거리 비행 plan은 생성하지 않는다. 대략 1m drift / 100 m 비행 정도로.

**주석: 처음 자동 비행을 하기 전에 수동 비행을 통해 flow 센서가 원하는 경로를 제대로 추적하는지 확인해야 합니다.**
