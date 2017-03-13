# Flight Modes

**Flight Modes** 는 주어진 시간에 시스템의 상태를 정의합니다. flight mode 사이에 사용자 전환은 리모트 콘트롤의 스위치를 이용하는 방법과 [ground control station](qgroundcontrol-intro.md)을 이용하는 방법이 있습니다.

## Flight Mode 간단 정리

  * **_MANUAL_**
    * **Fixed wing aircraft/ rovers / boats:**
        * **MANUAL:** 조정자의 제어 입력이 그대로 출력 mixer로 전달
        * **STABILIZED:** 조정자의 입력 중 roll과 pitch는 *angle* command로 yaw는 manual 명령.
    * **Multirotors:**
        * **ACRO:** 조정자의 입력은 roll, pitch, yaw *rate* command로 비행체에 전달. multirotor를 완전히 뒤집기가 가능. throttle은 직접 출력 mixer로 전달.
        * **RATTITUDE** 조정자의 입력은 해당 mode의 임계값보다 큰 경우 roll, pitch, yaw *rate* command로 비행체에 전달. 만약 임계값보다 크지 않다면 roll과 pitch는 *angle* command로 yaw는 *rate* command로 전달. throttle은 직접 출력 mixer로 전달.
        * **ANGLE** 조정자의 입력은 roll과 pitch *angle* command로 전달되고 yaw는 *rate* command로 전달. throttle은 직접 output mixer로 전달.
  * **_ASSISTED_**
    * **ALTCTL**
      * **Fixed wing aircraft:** roll, pitch 그리고 yaw 입력이 모두 가운데 위치할때(지정한 deadband 범위을 넘지 않는 경우), 비행체는 현재 고도를 유지하며 똑바로 돌아온다. 바람이 부는 경우 drift가 발생할 수 있다.
      * **Multirotors:** roll, picth 그리고 yaw 입력은 MANUAL mode와 동일. throttle 입력은 사전에 지정한 최대 rate에서 위나 아래로 이동 지시. throttle은 큰 deadzone을 가짐.
    * **POSCTL**
      * **Fixed wing aircraft:** Neutral inputs give level, flight and it will crab against the wind if needed to maintain a straight line.
      * **Multirotors** Roll controls left-right speed, pitch controls front-back speed over ground. When roll and pitch are all centered (inside deadzone) the multirotor will hold position. Yaw controls yaw rate as in MANUAL mode. Throttle controls climb/descent rate as in ALTCTL mode.
  * **_AUTO_**
    * **AUTO_LOITER**
        * **Fixed wing aircraft:** The aircraft loiters around the current position at the current altitude (or possibly slightly above the current altitude, good for 'I'm losing it').
        * **Multirotors:**  The multirotor hovers / loiters at the current position and altitude.
    * **AUTO_RTL**
        * **Fixed wing aircraft:** The aircraft returns to the home position and loiters in a circle above the home position.
        * **Multirotors:** The multirotor returns in a straight line on the current altitude (if higher than the home position + loiter altitude) or on the loiter altitude (if higher than the current altitude), then lands automatically.
    * **AUTO_MISSION**
        * **All system types:** The aircraft obeys the programmed mission sent by the ground control station (GCS). If no mission received, aircraft will LOITER at current position instead.
  * **_OFFBOARD_**
    In this mode the position, velocity or attitude reference / target / setpoint is provided by a companion computer connected via serial cable and MAVLink. The offboard setpoint can be provided by APIs like [MAVROS](https://github.com/mavlink/mavros) or [Dronekit](http://dronekit.io).

## Flight Mode Evaluation Diagram
![](images/diagrams/commander-flow-diagram.png)
