# Optical Flow와 LIDAR
----------------------------------------------------

이 페이지에서 position estimation을 위해 PX4Flow와 LIDAR 거리 측정 장치를 셋업하는 방법에 대해서 알아봅니다. LPE estimator를 사용한다면 LIDAR 장치를 사용할 필요가 없습니다. 왜냐하면 PX4FLOW는 sonar가 있기 때문입니다. 그래도 LIDAR를 사용하면 성능을 개선할 수 있습니다.

## Estimator 선택하기
--------------------------------------------------------

2가지 estimator인 LPE와 INAV는 navigation 기반의 optical flow를 지원합니다. LPE는 새로 시작하는 사용자에게 추천하는데 이유는 테스팅을 가장 많이 했고 가장 안정적입니다. INAV는 CPU를 약간 적게 사용합니다.

`SYS_MC_EST_GROUP` 파라미터를 이용해서 estimator를 선택하고 나서 리부팅합니다.


## 하드웨어
--------------------------------------------------------

![](images/hardware/px4flow_offset.png)

*Figure 1: Mounting Coordinate Frame (아래 parameter에 관련)*

![](images/hardware/px4flow.png)

*Figure 2: PX4Flow optical flow sensor (카메라와 sonar)*

PX4Flow는 지면을 향하도록 해야하고 pixhawk의 I2C 포트를 이용해서 연결합니다. PX4Flow가 최상의 성능을 내기 위해서는 적당한 위치에 부착해야하고 진동에 노출되지 않도록 해야합니다.(쿼드로터의 아랫면을 선호)

**주석: 기본 방향은 PX4Flow sonar 면이(flow에서 +Y) 비행체의 +X를(정면) 향하는 것입니다. 만약 그렇지 않은 경우 SENS_FLOW_ROT을 알맞게 설정해줘야 합니다.**

![](images/hardware/lidarlite.png)

*Figure 3: Lidar Lite*

Lidar-Lite와 sf10a를 포함해서 다양한 LIDAR 선택이 가능합니다. : [sf10a](http://www.lightware.co.za/shop/en/drone-altimeters/33-sf10a.html). [LIDAR-Lite 연결](https://pixhawk.org/peripherals/rangefinder?s[]=lidar).
sf10a는 시리얼케이블로 연결할 수 있습니다.


![](images/hardware/flow_lidar_attached.jpg)

*Figure: PX4Flow/ Lidar-Lite를 DJI F450에 장착*

![](images/flow/flow_mounting_iris.png)

*Figure: 이 Iris+는 LIDAR 없이 PX4Flow가 장착. LPE estimator로 동작*

![](images/flow/flow_mounting_iris_2.png)

*Figure: flow unit을 위해 바람을 막아주는 케이스 사용. 폼으로 sonar 주면을 감싸줘서 프로펠러 노이즈를 감소시키고 충돌시 카메라를 보호*


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

### Flight Video Indoor
{% youtube %}https://www.youtube.com/watch?v=CccoyyX-xtE{% endyoutube %}

### Flight Video Outdoor
{% youtube %}https://www.youtube.com/watch?v=Ttfq0-2K434{% endyoutube %}

LPE estimator로 outdoor automouse mission을 수행관련해서 (Optical Flow Outdoors)[./optical-flow-outdoors.md] 튜토리얼을 참고합니다.

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

**주석: GPS이 비행하려면 LPE_GPS_ON은 반드시 0으로 설정되어야 함 **

## Inertial Navigation Estimator (INAV)
--------------------------------------------------------

INAV는 correction을 위해 고정 gain matrix를 가지고 있어서 고정 상태 Kalman filter로 볼 수 있습니다. 전체 position estimator 중에서 계산량이 가장 적습니다.


### Flight Video Indoor
{% youtube %}https://www.youtube.com/watch?v=MtmWYCEEmS8{% endyoutube %}

### Flight Video Outdoor
{% youtube %}https://www.youtube.com/watch?v=4MEEeTQiWrQ{% endyoutube %}


### Parameters
* INAV_LIDAR_EST
	1로 설정하면 측정 거리 기반으로 altitude estimation을 활성화
* INAV_FLOW_DIST_X and INAV_FLOW_DIST_Y
	이 2개 값(in meters)은 yaw compensation을 위해 사용.
	offset은 위 Figure 1과 같이 측정해야만 함.
	위 예제에서 PX4Flow의 offset은(붉은 점) 음수의 X offset과 음수의 Y offset을 가짐.
* INAV_LIDAR_OFF
	lidar-lite(in meters)에 대해서 칼리브레이션 offset을 설정. 그 값을 측정한 거리에 더해준다.


### 고급 Parameters

고급 사용/개발을 위해서 다음 parameter도 변경할 수 있어야 합니다. 관련된 내용을 잘 모른다면 변경하지 않습니다!

* INAV_FLOW_W
	flow의 estimation/update에 대한 weight를 설정
* INAV_LIDAR_ERR
	altitude estimation/update(in meter)에 대한 threshold를 설정. 만약 correction term이 이 값보다 크다면, 업데이트에 사용되지 않음.
