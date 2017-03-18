# Hardware in the Loop Simulation (HITL)

HITL은 autopilot이 시뮬레이터에 연결되는 일종의 시뮬레이션 모드이고 모든 flight code는 autopilot에서 실행됩니다. 이 방법은 실제 flight code를 실제 프로세서에서 테스팅할 수 있다는 장점이 있습니다.

## HITL을 위한 시스템 설정

PX4는 멀티콥터(jMAVSim)와 고정익용(X-Plane 데모)으로 HITL을 지원합니다. flightgear 지원도 가능하지만 일정적으로 X-Plane을 추천합니다. 이를 활성화시키기 위해서 airframe 메뉴를 통해 설정합니다.

![](images/gcs/qgc_hil_config.png)

## jMAVSim 사용하기 (Quadrotor)

- QGroundControl를 실행하지 않는 상태인지 확인(아니면 장치를 시리얼 포트를 통해 연결)
- jMAVSim을 HITL 모드에서 실행 (필욯아면 시리얼 포트를 대체) :
  ```
  ./Tools/jmavsim_run.sh -q -d /dev/ttyACM0 -b 921600
  ```
- 콘솔은 mavlink 텍스트 메시지를 autopilot에서 보여줌
- 다음으로 QGroundControl을 실행하고 UDP 기본 설정으로 연결


## X-Plane 사용하기
#### X-Plane엣어 원격 접속하기

X-Plane에서 해야할 2가지 핵심 설정 : Setting -> Data Input and Output 에서 다음과 같이 체크박스를 설정 :

![](images/gcs/xplane_data_config.png)

Data 탭에 있는 Settings -> Net Connections에서 IP주소로 localhost와 port 49005를 설정. 다음 화면을 참고하세요 :

![](images/gcs/xplane_net_config.png)

#### QGroundControl에서 HITL 활성화

Widgets -> HIL Config에서 드롭다운해서 X-Plane 10을 선택하고 연결을 누릅니다. 일단 시스템이 연결되면 배터리 상태, GPS 상태, 비행체 위치가 모두 유효하게 됩니다. :

![](images/gcs/qgc_sim_run.png)

## 조이스틱 입력으로 전환

조이스틱이 라디오 리모트 콘트롤보다 낫다면, `COM_RC_IN_MODE` parameter를 `1`로 설정합니다. Commander parameter group에서 찾을 수 있습니다.

## HITL에서 자동 mission으로 비행

flight planning 창으로 변환하고 비행기 앞에 waypoint를 하나 찍습니다. waypoint를 전송을 위해 sync 아이콘을 클릭합니다.

툴바에 있는 flight mode 메뉴에서 MISSION을 선택하고 비행체를 arm하기 위해서 DISARMED을 클릭합니다. 이륙하고 이륙한 waypoint 주변에 떠있습니다.

![](images/gcs/qgc_sim_mission.png)
