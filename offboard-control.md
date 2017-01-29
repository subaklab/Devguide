# Offboard Control

> ** 주의 ** Offboard control은 위험합니다. offboard 비행 전에 개발자가 충분한 준비, 테스팅 그리고 안전 예방조치를 취해야 합니다.

off-board control의 기본 개념은 autopilot 외부에서 실행되는 소프트웨어를 사용해서 px4 flight stack을 제어한다는 것입니다. 이는 Mavlink 프로토콜을 통해 수행되며, 특별히 [SET_POSITION_TARGET_LOCAL_NED](http://mavlink.org/messages/common#SET_POSITION_TARGET_LOCAL_NED)와 [SET_ATTITUDE_TARGET](http://mavlink.org/messages/common#SET_ATTITUDE_TARGET) 메시지가 그것입니다.

## Offboard Control 펌웨어 셋업

offboard 개발을 시작하기에 앞서 펌웨어 쪽에서 2가지 셋업이 필요합니다.

### 1. RC 스위치를 offboard mode 활성화에 매핑
이를 위해, QGroundcontrol에서 파라미터를 설정하고 RC_MAP_OFFB_SW 파라미터를 찾아서 offboard mode를 활성화시키는데 사용할 RC 채널을 할당할 수 있습니다. offboard mode 빠져나와 position control로 진입하는 방식으로 매핑하는 것이 유용할 수 있습니다.

MAVLink 메시지를 사용해서 offboard mode를 활성화시킬 수 있기 때문에 이 단계는 강제사항은 아닙니다. 더 안전하게 하기 위해서 이 방식을 고려하는 것입니다.

### 2. companion computer 인터페이스 유효
[SYS_COMPANION](https://pixhawk.org/firmware/parameters#system) 파라미터를 찾아서 921600 (추천) 혹은 57600으로 설정합니다. 이 파라미터는 데이터 스트림을 사용하는 Telem2 포트에서 MAVLink 스트림을 활성화시키며 여기에는 적절한 baud rate (921600 8N1 혹은 57600 8N1)의 onboard mode로 지정되어 있어야 합니다.

데이터 스트림에 관한 추가 정보는 [source code](https://github.com/PX4/Firmware/blob/master/src/modules/mavlink/mavlink_main.cpp)에서 "MAVLINK_MODE_ONBOARD"를 참고하세요.

## Hardware 셋업

보통 offboard 통신을 셋업하는데 3가지 방법이 있습니다.

### 1. 시리얼 라디오
1. 한쪽을 autopilot의 UART 포트에 연결
2. 한쪽을 ground station computer에 연결

라디오 예제
* [Lairdtech RM024](http://www.lairdtech.com/products/rm024)
* [Digi International XBee Pro](http://www.digi.com/products/xbee-rf-solutions/modules)

{% mermaid %}
graph TD;
  gnd[Ground Station] --MAVLink--> rad1[Ground Radio];
  rad1 --RadioProtocol--> rad2[Vehicle Radio];
  rad2 --MAVLink--> a[Autopilot];
{% endmermaid %}

### 2. On-board 프로세서
비행체에 마운트되어 있는 소형 컴퓨터가 UART-to-USB 아답터를 통해 autopilot에 연결되었습니다. 여기에 다양한 방식이 있을 수 있으며, 명령을 autopilot에 보내는 것 외에 추가로 원하는 on-board 프로세싱이 무엇이냐에 달려있다.

소형 저전력 경우 :
* [Odroid C1+](http://www.hardkernel.com/main/products/prdt_info.php?g_code=G143703355573) or [Odroid XU4](http://www.hardkernel.com/main/products/prdt_info.php?g_code=G143452239825)
* [Raspberry Pi](https://www.raspberrypi.org/)
* [Intel Edison](http://www.intel.com/content/www/us/en/do-it-yourself/edison.html)

크고 전력소모가 있는 경우 :
* [Intel NUC](http://www.intel.com/content/www/us/en/nuc/overview.html)
* [Gigabyte Brix](http://www.gigabyte.com/products/list.aspx?s=47&ck=104)
* [Nvidia Jetson TK1](https://developer.nvidia.com/jetson-tk1)

{% mermaid %}
graph TD;
  comp[Companion Computer] --MAVLink--> uart[UART Adapter];
  uart --MAVLink--> Autopilot;
{% endmermaid %}

### 3. On-board 프로세서와 ROS에 wifi 연결 (***추천***)
ROS를 실행하는 ground station에 WiFi로 연결을 가지면서 비행체에 마운트되어 있는 소형 컴퓨터가 UART-to-USB 아답터를 통해 autopilot에 연결되었습니다. WiFi 아답터를 가지고 결합된 위에서 언급한 컴퓨터의 형태가 될 수 있습니다. 예를 들자면, Intel NUC D34010WYB은 PCI Express Half-Mini 커넥터를 가지고 있으며 이는 [Intel Wifi Link 5000](http://www.intel.com/products/wireless/adapters/5000/) 아답터을 사용할 수 있습니다.


{% mermaid %}
	graph TD
	subgraph Ground  Station
	  gnd[ROS Enabled Computer] --- qgc[qGroundControl]
	end
	gnd --MAVLink/UDP--> w[WiFi];
	qgc --MAVLink--> w;
	subgraph Vehicle
	  comp[Companion Computer] --MAVLink--> uart[UART Adapter]
	uart --- Autopilot
	end
	w --- comp
{% endmermaid %}
