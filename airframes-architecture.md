# Airframe 개요

PX4 시스템은 모듈화 구조로 되어 있어 모든 로봇 타입을 단일 코드베이스로 대응합니다.

{% mermaid %}
graph LR;
  Autopilot-->Controller;
  SafetyPilot-->Controller;
  Controller-->Mixer;
  Mixer-->Actuator
{% endmermaid %}

## 기본 장치

airframe 섹션에서 전체 하드웨어 설정은 다음과 같은 장치를 사용한다고 가정 :

  * 안전한 비행을 위한 Taranis Plus 리모트 컨트롤 (혹은 PPM / S.BUS 출력을 가지는 비슷한 장치)
  * 그라운드 컨트롤 스테이션
    * 삼성 Note 4 혹은 안드로이드 타블릿
    * iPad (Wifi telemetry 아답터 요망)
    * MacBook 혹은 Ubuntu Linux 노트북
  * 내부 컴퓨터 (소프트웨어 개발자용)
    * MacBook Pro 혹은 Air로 OS X 10.10 이상 지원
    * 최신 노트북으로 Ubuntu Linux 지원(14.04 이상)
  * 보안경
  * 멀티로터용 : 위험한 테스트를 위한 tether

PX4는 다양한 장치들과 함께 사용할 수 있지만 새로 입문하는 개발자는 표준 셋업으로 작업하면 도움을 받을 수 있을 것입니다. 비교적 저렴하게 사용할 수 있는 키트를 구성할 수 있습니다.
