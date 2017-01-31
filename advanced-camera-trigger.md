# 카메라 트리거 (Camera Trigger)
카메라 트리거 드라이버는 펄스를 AUX 포트를 사용해서 전송합니다.  드론을 이용해서 조사를 할때 타임스탬프를 가지는 사진, 여러 카메라를 사용하는 시스템의 동기화 혹은 시각-관성(visual-inertial) 네비게이션을 포함한 다양한 어플리케이션에 사용할 수 있습니다.

펄스를 보내는 것 외에도, 메시지의 순서 번호와 타임 스탬프를 포함한 MAVLink 메시지를 publish할 수 있습니다.

지원하는 3가지 모드 :
* `TRIG_MODE` 1은 기본 intervalometer입니다. 이것을 이용하면 시스템 콘솔에서 `camera_trigger enable`나 `camera_trigger disable`을 호출해서 활성/비활성화 가능합니다.
* `TRIG_MODE` 2는 intervalometer를 on 상태를 유지합니다.
* `TRIG_MODE` 3은 거리 기반한 트리거입니다. 설정한 수평 거리를 초과할 때마다 사진을 찍습니다. 사진을 찍는 시간 간격은 triggering interval 설정으로 제한됩니다.

`TRIG_MODE` 0에서 트리거링 기능은 off상태가 됩니다.

카메라 트리거 모듈과 관련된 파리미터의 전체 목록은 [parameter reference](https://pixhawk.org/firmware/parameters#camera_trigger)를 참조하세요.

> **정보 ** 처음으로 카메라 트리거 app을 활성화시킨 경우라면, `TRIG_MODE` 파라미터를 1, 2, 3 중에 하나 값으로 변경 후에 리부팅해야 한다는 것을 기억하세요.

## Camera-IMU sync 예제
이 예제에서 전형적인 VINS(Visual-Inertial Navigation System)을 구성하기 위해서 IMU 측정값을 영상 데이터와 동기화에 관한 기본 내용을 살펴봅니다. 정확히 말하자면 여기서 말하는 아이디어로는 사진찍기와 정확히 동일한 시점의 IMU 측정값을 가져오지는 못합니다. 오히려 이미지에 포함된 타임 스탬프가 VI 알고리즘에 더 정확한 데이터를 제공합니다.

autopilot과 companion은 서로 다른 clock(autopilot은 boot-time을 companion은 UNIX epoch)을 기반으로 합니다. 특정 clock을 선택하기보다는 직접 clock사이의 time offset을 보면 됩니다. 이 offset은 미들웨어간 교환이 가능한 컴포넌트(PX4에서는 `mavlink_receiver`고 companion에서는 Mavros)에서 mavlink 메시지(e.g `HIGHRES_IMU`)에 있는 timestamp에서 더하거나 빼는 것입니다. 실제 동기화 알고리즘은 NTP(Network Time Protocol)의 수정 버전이며 구한 time offset을 평평하게 하려고 평균을 사용합니다. Mavros가 높은 통신 대역폭을 가지는 보드에서 실행되는 경우 동기화는 자동으로 완료됩니다.

동기화된 이미지 프레임과 관성 측정값을 얻기 위해서, 2개 카메라의 트리거 입력을 autopilot의 GPIO 핀에 연결해야 합니다. mid-exposure부터 관성 측정값의 타임스탬프와 이미지 순서 번호가 기록되고 companion computer(`CAMERA_TRIGGER` 메시지)로 전송됩니다. 이 패킷들과 카메라에서 가져온 이미지 프레임들을 버퍼에 모아둡니다. 순서 번호, 타임스탬프 찍은 이미지들을 기반으로 매치시키고 나면 publish합니다.

다음 다이아그램은 이미지에 제대로 타임스탬프를 찍기 위해서 순서대로 일어나는 일련의 일들을 보여줍니다.

{% mermaid %}
sequenceDiagram
  Note right of px4 : Time sync with mavros is done automatically
  px4 ->> mavros : Camera Trigger ready
  Note right of camera driver : Camera driver boots and is ready
  camera driver ->> mavros : mavros_msgs::CommandTriggerControl
  mavros ->> px4 : MAVLink::MAV_CMD_DO_TRIGGER_CONTROL
  loop Every TRIG_INTERVAL milliseconds
  px4 ->> mavros : MAVLink::CAMERA_TRIGGER
  mavros ->> camera driver : mavros_msgs::CamIMUStamp
  camera driver ->> camera driver : Match sequence number
  camera driver ->> camera driver : Stamp image and publish
end
{% endmermaid %}

### Step 1
먼저 TRIG_MODE를 1로 설정해서 드라이버가 start 명령을 기다리게 하고 FCU를 리부팅해서 다른 파라미터가 반영되도록 합니다.

### Step 2
이 예제의 목적은 30 FPS로 실행되는 Point Grey Firefly MV camera와 함께 동작하기 위해서 트리거를 설정하는 것입니다.

* TRIG_INTERVAL: 33.33 ms
* TRIG_POLARITY: 0, active low
* TRIG_ACT_TIME: 0.5 ms, 기본값으로. 메뉴얼에서는 최소 1ms라고만 되어있음.
* TRIG_MODE: 1, 카메라 드라이버가 트리거를 시작하기 전에 이미지를 받을 준비가 되기를 원합니다. 적절하게 처리하기 위해서순서 번호가 필요합니다.
* TRIG_PINS: 12, 기본값으로.

### Step 3
그라운드(-)와 신호 핀을 적절하게 연결함으로써 카메라를 AUX 포트에 연결하세요.

### Step 4
위에 순서도를 따라서 드라이버를 수정해야 합니다. [IDS Imaging UEye](https://github.com/ProjectArtemis/ueye_cam) 카메라와 [IEEE1394 compliant](https://github.com/andre-nguyen/camera1394) 카메라에 대한 구현을 참고하세요.
