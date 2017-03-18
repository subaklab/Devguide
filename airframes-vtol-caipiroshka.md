# TBS Caipiroshka

Caipiroshka VTOL는 TBS Caipirinha를 조금 수정하였습니다.

{% youtube %}https://www.youtube.com/watch?v=acG0aTuf3f8&vq=hd720{% endyoutube %}

## 부품 리스트

  * TBS Caipirinha Wing ([Eflight store](http://www.eflight.ch/shop/USER_ARTIKEL_HANDLING_AUFRUF.php?von_suchresultat=true&Ziel_ID=19638&Kategorie_ID=110923))
  * Left and right 3D-printed motor mount ([design files](parts/motor_mounts.zip))
  * CW 8045 propeller ([Eflight store](http://www.eflight.ch/shop/USER_ARTIKEL_HANDLING_AUFRUF.php?von_suchresultat=true&Ziel_ID=19532&Kategorie_ID=288))
  * CCW 8045 propeller ([Eflight store](http://www.eflight.ch/shop/USER_ARTIKEL_HANDLING_AUFRUF.php?von_suchresultat=true&Ziel_ID=19533&Kategorie_ID=288))
  * 2x 1800 kV 120-180W motors
    * [Quanum MT2208 1800 kV](http://www.hobbyking.com/hobbyking/store/__67014__Quanum_MT_Series_2208_1800KV_Brushless_Multirotor_Motor_Built_by_DYS.html)
    * [ePower 2208](http://www.eflight.ch/pi/ePower-X-22081.html)
  * 2x 20-30S ESC
    * [Eflight store](http://www.eflight.ch/shop/USER_ARTIKEL_HANDLING_AUFRUF.php?von_suchresultat=true&Ziel_ID=19713&Kategorie_ID=36077)
  * BEC (3A, 5-5.3V) (5V 파워 공급을 수행하지 않는 ESC를 사용하는 경우에만 필요)
  * 3S 2200 mA LiPo battery
    * Team Orion 3S 11.1V 50 C ([Brack store](https://www.brack.ch/team-orion-2200mah-11-1v-50c-308340))
  * [Pixracer autopilot board + power module](hardware-pixracer.md)
  * [Digital airspeed sensor](http://www.hobbyking.com/hobbyking/store/__62752__HKPilot_32_Digital_Air_Speed_Sensor_And_Pitot_Tube_Set.html)

## 조립
아래 사진은 전체 Caipiroshka를 조립하면 어떤 모양인지를 보여줍니다.

![Caipiroshka](images/airframes/vtol/caipiroshka/caipiroshka.jpg)

다음으로 비행체를 조립하는 방법에 대한 일반적인 팁을 알려드립니다.

### Autopilot
airframe의 무게중심(CG)와 가까운 중간에 autopilot을 장착합니다.

### Motor 장착


### Motor controllers

### GPS

### Airspeed sensor

### Sensor connection to the I2C bus

### Elevons

### General assembly rules

## Airframe configuration

[QGroundControl](qgroundcontrol-intro.md)에서 configuration 섹션으로 가서 airframe 탭을 선택합니다. 목록에서 VTOL Duorotor Tailsitter 아이콘을 찾아서  ```Duorotor Tailsitter```을 선택합니다.

![](images/gcs/qgc_caipiroshka.jpg)

## Servo Connections

| Output | Rate | Actuator |
| --- | --- | --- |
| MAIN1 | 400 Hz | Left motor controller |
| MAIN2 | 400 Hz | Right motor controller |
| MAIN3 | 400 Hz | Empty |
| MAIN4 | 400 Hz | Empty |
| MAIN5 | 50 Hz | Left aileron servo |
| MAIN6 | 50 Hz | Right aileron servo |
