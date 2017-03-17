# UAVCAN 소개

![](images/uavcan-logo-transparent.png)

[UAVCAN](http://uavcan.org)는 autopilot과 항공 전자기기를 연결하는 onboard 네트워크입니다. 다음과 같은 하드웨어를 지원 :

  * Motor controllers
    * [Pixhawk ESC](https://pixhawk.org/modules/pixhawk_esc)
    * [SV2740 ESC](https://github.com/thiemar/vectorcontrol)
  * Airspeed sensors
    * [Thiemar airspeed sensor](https://github.com/thiemar/airspeed)
  * GPS와 GLONASS용 GNSS receivers
    * [Zubax GNSS](http://zubax.com/product/zubax-gnss)

취미용으로 사용하는 장치와 구별되는 특징은 bus를 이용해서 differential signaling과 펌웨어 업그레이드를 지원합니다. 모든 motor controller는 상태를 피드백하고 FOC(field-oriented-control)을 구현합니다.

## Node 펌웨어 업그레이드

상황에 따라서 PX4 미들웨어는 자동으로 UAVCAN 노드에 있는 펌웨어를 업그레이드할 수 있습니다. 절차 및 요구사항은 [UAVCAN Firmware](uavcan-node-firmware.md)를 참고하세요.

## Enumerating과 Motor Controllers 설정

ID와 각 모터 제어기의 회전 방향은 셋업 절차를 마친 후에 할당할 수 있습니다.([UAVCAN Node Enumeration](uavcan-node-enumeration.md)) 사용자가 QGroundControl에서 이 절차대로 수행할 수 있습니다.

## 유용한 링크

* [홈페이지](http://uavcan.org)
* [스펙](http://uavcan.org/Specification)
* [구현 및 튜토리얼](http://uavcan.org/Implementations)
