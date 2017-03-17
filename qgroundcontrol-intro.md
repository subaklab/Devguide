# QGroundControl

QGroundControl은 PX4 기반 autopilot을 설정하고 날리는데 사용하는 app입니다. 크로스 플랫폼으로 개발되어 모든 주요 운영체제에서 동작합니다. :

  * 모바일: Android와 iOS (현재 타블릿에 초점)
  * 데스크탑: Windows, Linux, Mac OS

## Planning Missions

새로운 mission을 생성하기 위해서는 planning 탭으로 변경하고 왼쪽위에 있는 + 아이콘을 클릭하고 waypoint를 생성하기 위해서 map을 클릭합니다. context 메뉴는 waypoint를 조정하기 위해서 측면에서 실행됩니다. 비행체에 전송할려면 하이라이트된 transmission 아이콘을 클릭합니다.

![](images/gcs/planning-mission.png)

## Flying Missions

flying 탭으로 변경합니다. mission은 map에서 확인할 수 있습니다. 현재 flight mode를 클릭해서 MISSION으로 변경하고 비행체를 arm하기 위해서 DISARMED를 클릭합니다. 비행체가 이미 비행 중이라면 mission의 첫번째 위치로 날라가면서 mission을 수행하게 됩니다.

![](images/gcs/flying-mission.png)

## parameters 설정

setup 탭으로 변경합니다. 왼쪽에 있는 메뉴를 끝까지 내려서 parameter 아이콘을 클릭합니다. parameter를 더블클릭해서 수정이 가능하며 이렇게하면 context menu를 열게 됩니다.

![](images/gcs/setting-parameter.png)

## 설치

QGroundControl은 [website](http://qgroundcontrol.com/downloads)에서 다운받을 수 있습니다.

<aside class="tip">
개발자는 stable 버전보다는 최신 버전을 사용하는 것을 추천합니다.
</aside>

## 소스에서 빌드하기

펌웨어 개발자는 최신 flight 코드와 매칭되는 소스를 받아서 빌드해야 하는 경우가 많습니다.

Qt를 설치하고 소스코드를 빌드하는 방법은 [QGroundControl build instructions](https://github.com/mavlink/qgroundcontrol#obtaining-source-code)를 참고합니다.
