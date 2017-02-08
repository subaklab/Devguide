# VTOL 테스팅

VTOL 기능으르 테스트하는 방법으로 핵심은 transition :

  * 대기 상태 (On the bench)
  * 비행 중 (In flight)

## transition 중 일반 주의내용

현재 제공하는 VTOL에 transiton 명령을 내리는 3가지 방식 :

  * RC 스위치 (2 pos, aux1)
  * MAVLink 명령 (MAV_CMD_DO_VTOL_TRANSITION)
  * mission 중에 transition (내부적으로 MAV_CMD_DO_VTOL_TRANSITION)

위 방법중에 하나로 transition 명령이 내리면, VTOL은 transition phase로 들어갑니다. 만약 VTOL이 transiton을 진행 중인 상태에서 이전 상태로 돌아가는 새로운 transition 명령을 받으면, 바로 이전 상태로 돌아갑니다. 필요한 경우 transition을 취소하는 것은 안전과 관련된 기능입니다. transition이 완료되면, VTOL은 새로운 상태로 들어갔으므로 반대 방향의 transition 명령은 정상적으로 수행하게 됩니다.

<aside class="note">
AUX1 채널을 RC 스위치에 할당하고 airspeed가 정상적으로 동작하는지 확인합니다.
</aside>

## On the bench

<aside class="caution">
모든 프로펠러를 제거합니다! transition 기능을 제대로 테스트하기 위해서 비행체는 arm 상태로 되어 있어야 합니다.
</aside>

By default, starting in multirotor mode:

  * arm the vehicle
  * check that motors are running in multirotor configuration (rudders/elevons should not move on roll/pitch/yaw inputs)
  * toggle transition switch
  * (if applicable) wait on step 1 of the transition phase to complete
  * blow into pito tube to simulate airspeed
  * (if applicable) step 2 of the transition phase will be executed
  * check that motors are running in fixed-wing configuration (roll/pitch/yaw inputs should control rudders/elevons)
  * toggle transition switch
  * observe back transition
  * check that motors are running in multirotor configuration (rudders/elevons should not move on roll/pitch/yaw inputs)

## In flight

<aside class="tip">
Before testing transitions in flight, make sure the VTOL flies stable in multirotor mode. In general, if something doesn't go as planned, transition to multirotor mode and let it recover (it does a good job when it's properly tuned).
</aside>

In-flight transition requires at least the following parameters to match your airframe and piloting skills:

| Param | Notes |
| :--- | :--- |
| VT_FW_PERM_STAB | Turns permanent stabilization on/off for fixed-wing. |
| VT_ARSP_BLEND | At which airspeed the fixed-wing controls becom active. |
| VT_ARSP_TRANS | At which airspeed the transition to fixed-wing is complete. |

There are more parameters depending on the type of VTOL, see the [parameter reference](https://pixhawk.org/firmware/parameters#vtol_attitude_control).

### Manual transition test

The basic procedure to test manual transitions is as follows:

  * arm and takeoff in multirotor mode
  * climb to a safe height to allow for some drop after transition
  * turn into the wind
  * toggle transition switch
  * observe transition **(MC-FW)**
  * fly in fixed-wing
  * come in at a safe height to allow for some drop after transition
  * toggle transition switch
  * observe transition **(FW-MC)**
  * land and disarm

**MC-FW**

During the transition from MC to FW the following can happen:

  1. it looses control while gaining speed (this can happen due to many factors)
  2. the transition takes too long and it flies too far away before the transition finishes

For 1): Switch back to multirotor (will happen instantly). Try to identify the problem (check setpoints).

For 2): If blending airspeed is set and it has a higher airspeed already it is controllable as fixed-wing. Therefore it is possible to fly around and give it more time to finish the transition. Otherwise switch back to multirotor and try to identify the problem (check airspeed).

**FW-MC**

The transition from FW to MC is mostly unproblematic. In-case it seems to loose control the best approach is to let it recover.

### Automatic transition test (mission, commanded)

Commanded transitions only work in auto (mission) or offboard flight-mode. Make sure you are confident to operate the auto/offboard and transition switch in flight.

Switching to manual will reactivate the transition switch. For example: if you switch out of auto/offboard when in automatic fixed-wing flight and the transition switch is currently in multirotor position it will transition to multirotor right away.

#### Proceduce

The following procedure can be used to test a mission with transition:

  * upload mission
  * takeoff in multirotor mode and climb to mission height
  * enable mission with switch
  * observe transition to fixed-wing flight
  * enjoy flight
  * observe transition back to multirotor mode
  * disable mission
  * land manually

During flight, the manual transition switch stays in multirotor position. If something doesn't go as planned, switch to manual and it will recover in multirotor mode.

#### Example mission

The mission should contain at least (also see screenshots below):

  * (1) position waypoint near takeoff location
  * (2) position waypoint in the direction of the planned fixed-wing flight route
  * (3) transition waypoint (to plane mode)
  * (4) position waypoint further away (at least as far away as the transition needs)
  * (6) position waypoint to fly back (a bit before takeoff location so back transition takes some distance)
  * (7) transition waypoint (to hover mode)
  * (8) position waypoint near takeoff location

![Mission, showing transition WP to plane](images/vtol/qgc_mission_example_a.png)

![Mission, showing transition WP to hover](images/vtol/qgc_mission_example_b.png)
