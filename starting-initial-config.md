# 초기 설정

PX4에서 개발을 시작하기에 앞서, 하드웨어가 제대로 설정되어 있고 테스트되었는지 확인하기 위해서 시스템을 기본 설정으로 초기화해야 합니다. 아래 비디오는 [Pixhawk 하드웨어](hardware-pixhawk.md)와 [QGroundControl](qgroundcontrol-intro.md)에서 셋업 절차를 설명합니다. 지원하는 에어프레임 목록은 [여기](airframes-architecture.md)를 참고합니다.

> **정보** [QGroundControl Daily 빌드 버전](http://qgroundcontrol.com/downloads) 다운받고 아래 비디오를 보고 비행체를 셉업하세요. 미션 플래닝, 비행 및 파라미터 셋팅관련해서는 다음 비디오 [QGroundControl Tutorial](http://dev.px4.io/qgroundcontrol-intro.html)를 참고하세요.

셋업 옵션의 목록은 아래 비디오를 참고합니다.

{% youtube %}https://www.youtube.com/watch?v=91VGmdSlbo4&rel=0&vq=hd720{% endyoutube %}

## 라디오 컨트롤 옵션

PX4 flight stack은 반드시 라디오 컨트롤 시스템을 필요로 하지는 않습니다. 또한 flight mode를 선택하기 위해서도 개별 스위치 사용을 강제하지는 않습니다.

### 라디오 컨트롤 없이 비행

모든 라디오 컨트롤 셋업 체크는 `COM_RC_IN_MODE` 파라미터를 `1`로 설정하면 비활성화됩니다. 이렇게 하면 manual flight를 사용할 수 없게 됩니다.

### 싱글 채널 모드 스위치(Single Channel Mode Switch)

이 모드에서는 여러 스위치를 사용하는 대신에 하나의 채널을 모드 스위치로 사용합니다. 이에 대한 설명은 [기존 wiki](https://pixhawk.org/peripherals/radio-control/opentx/single_channel_mode_switch)를 참고합니다.
