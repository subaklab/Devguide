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

좋은 optical flow 품질을 얻기 위해서는 PX4Flow에 있는 카메라를 원하는 비행 높이로 초점을 맞추는 것이 중요합니다. 카메라에 초점을 맞추기 위해서 object를 .
In order to ensure good optical flow quality, it is important to focus the camera on the PX4Flow to the desired height of flight. To focus the camera, put an object with text on (e. g. a book) and plug in the PX4Flow into usb and run QGroundControl. Under the settings menu, select the PX4Flow and you should see a camera image. Focus the lens by unscrewing the set screw and loosening and tightening the lens to find where it is in focus.

**Note: If you fly above 3m, the camera will be focused at infinity and won't need to be changed for higher flight.**

![](images/flow/flow_focus_book.png)

*Figure: Use a text book to focus the flow camera at the height you want to fly, typically 1-3 meters. Above 3 meters the camera should be focused at infinity and work for all higher altitudes.*


![](images/flow/flow_focusing.png)

*Figure: The px4flow interface in QGroundControl that can be used for focusing the camera*

### Sensor Parameters

All the parameters can be changed in QGroundControl
* SENS_EN_LL40LS
	Set to 1 to enable lidar-lite distance measurements
* SENS_EN_SF0X
	Set to 1 to enable lightware distance measurements (e.g. sf02 and sf10a)

## Local Position Estimator (LPE)
--------------------------------------------------------

LPE is an Extended Kalman Filter based estimator for position and velocity states. It uses inertial navigation and is similar to the INAV estimator below but it dynamically calculates the Kalman gain based on the state covariance. It also is capable of detecting faults, which is beneficial for sensors like sonar which can return invalid reads over soft surfaces.

### Flight Video Outdoor
{% youtube %}https://www.youtube.com/watch?v=Ttfq0-2K434{% endyoutube %}

Below is a plot of the autonomous mission from the outdoor flight video above using optical flow. GPS is not used to estimate the vehicle position but is plotted for a ground truth comparison. The offset between the GPS and flow data is due to the initialization of the estimator from user error on where it was placed. The initial placement is assumed to be at LPE_LAT and LPE_LON (described below).

![](images/lpe/lpe_flow_vs_gps.png)

*Figure 4: LPE based autonomous mission with optical flow and sonar*


### Parameters

The local position estimator will automatically fuse LIDAR and optical flow data when the sensors are plugged in.

* LPE_FLOW_OFF_Z - This is the offset of the optical flow camera from the center of mass of the vehicle. This measures positive down and defaults to zero. This can be left zero for most typical configurations where the z offset is negligible.
* LPE_FLW_XY - Flow standard deviation in meters.
* LPW_FLW_QMIN - Minimum flow quality to accept measurement.
* LPE_SNR_Z -Sonar standard deviation in meters.
* LPE_SNR_OFF_Z - Offset of sonar sensor from center of mass.
* LPE_LDR_Z - Lidar standard deviation in meters.
* LPE_LDR_Z_OFF -Offset of lidar from center of mass.
* LPE_GPS_ON - You won't be able to fly without GPS if LPE_GPS_ON is set to 1. You must disable it or it will wait for GPS altitude to initialize position. This is so that GPS altitude will take precedence over baro altitude if GPS is available.

**NOTE: LPE_GPS_ON must be set to 0 to enable flight without GPS **

### Autonomous Flight Parameters

*Tell the vehicle where it is in the world*

* LPE_LAT - The latitude associated with the (0,0) coordinate in the local frame.
* LPE_LON - The longitude associated with the (0,0) coordinate in the local frame.

*Make the vehicle keep a low altitude and slow speed*

* MPC_ALT_MODE - Set this to 1 to enable terrain follow
* LPE_T_Z - This is the terrain process noise. If your environment is hilly, set it to 0.1, if it is a flat parking lot etc. set it to 0.01.
* MPC_XY_VEL_MAX - Set this to 2 to limit leaning
* MPC_XY_P - Decrease this to around 0.5 to limit leaning
* MIS_TAKEOFF_ALT - Set this to 2 meters to allow low altitude takeoffs.

*Waypoints*

* Create waypoints with altitude 3 meters or below.
* Do not create flight plans with extremely long distance, expect about 1m drift / 100 m of flight.

**Note: Before your first auto flight, walk the vehicle manually through the flight with the flow sensor to make sure it will trace the path you expect.**
