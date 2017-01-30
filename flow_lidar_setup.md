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

*Figure 1: Mounting Coordinate Frame (relevant to parameters below)*

![](images/hardware/px4flow.png)

*Figure 2: PX4Flow optical flow sensor (camera and sonar)*

The PX4Flow has to point towards the ground and can be connected using the I2C port on the pixhawk. For best performance make sure the PX4Flow is attached at a good position and is not exposed to vibration. (preferably on the down side of the quad-rotor).

**Note: The default orientation is that the PX4Flow sonar side (+Y on flow) be pointed toward +X on the vehicle (forward). If it is not, you will need to set SENS_FLOW_ROT accordingly.**

![](images/hardware/lidarlite.png)

*Figure 3: Lidar Lite*

Several LIDAR options exist including the Lidar-Lite (not currently manufacutured) and the sf10a: [sf10a](http://www.lightware.co.za/shop/en/drone-altimeters/33-sf10a.html). For the connection of the LIDAR-Lite please refer to [this](https://pixhawk.org/peripherals/rangefinder?s[]=lidar) page. The sf10a can be connected using a serial cable.

![](images/hardware/flow_lidar_attached.jpg)

*Figure: PX4Flow/ Lidar-Lite mounting DJI F450*

![](images/flow/flow_mounting_iris.png)

*Figure: This Iris+ has a PX4Flow attached without a LIDAR, this works with the LPE estimator.*

![](images/flow/flow_mounting_iris_2.png)

*Figure: A weather-proof case was constructed for this flow unit. Foam is also used to surround the sonar to reduce prop noise read by the sonar and help protect the camera lens from crashes.*


### Focusing Camera

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

### Flight Video Indoor
{% youtube %}https://www.youtube.com/watch?v=CccoyyX-xtE{% endyoutube %}

### Flight Video Outdoor
{% youtube %}https://www.youtube.com/watch?v=Ttfq0-2K434{% endyoutube %}

For outdoor autonmous missions with LPE estimator, see tutorial on (Optical Flow Outdoors)[./optical-flow-outdoors.md]

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

## Inertial Navigation Estimator (INAV)
--------------------------------------------------------

INAV has a fixed gain matrix for correction and can be viewed as a steady state Kalman filter. It has the lowest computational cost of all position estimators.


### Flight Video Indoor
{% youtube %}https://www.youtube.com/watch?v=MtmWYCEEmS8{% endyoutube %}

### Flight Video Outdoor
{% youtube %}https://www.youtube.com/watch?v=4MEEeTQiWrQ{% endyoutube %}


### Parameters
* INAV_LIDAR_EST
	Set to 1 to enable altitude estimation based on distance measurements
* INAV_FLOW_DIST_X and INAV_FLOW_DIST_Y
	These two values (in meters) are used for yaw compensation.
	The offset has to be measured according to Figure 1 above.
	In the above example the offset of the PX4Flow (red dot) would have a negative X offset and a negative Y offset.
* INAV_LIDAR_OFF
	Set a calibration offset for the lidar-lite in meters. The value will be added to the measured distance.


### Advanced Parameters

For advanced usage/development the following parameters can be changed as well. Do NOT change them if you do not know what you are doing!

* INAV_FLOW_W
	Sets the weight for the flow estimation/update
* INAV_LIDAR_ERR
	Sets the threshold for altitude estimation/update in meters. If the correction term is bigger than this value, it will not be used for the update.
