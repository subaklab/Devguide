# Intel Aero

Aero는 UAV 개발 플랫폼입니다. 아래와 같이 Intel Aero Compute Board는 이 플랫폼의 일부입니다. 쿼드코어 CPU에서 Linux가 동작합니다. NuttX 기반의 PX4가 실행되는 FMU가 여기에 연결됩니다.


![](images/hardware/hardware-intel-aero.png)

## 소개

주요 문서는 https://github.com/intel-aero/meta-intel-aero/wiki 를 참고합니다. 보드 셋업 방법, 업데이트, 연결방법에 대해서 소개합니다. Linux에서 개발하는 방법도 소개하고 있습니다.

이제부터 flash하는 방법과 FMU에 연결하는 방법을 소개합니다.


## Flashing

PX4 개발 환경을 설정한 후에, PX4 소프트웨어를 FMU 보드에 심기 위해 다음을 따라합니다. :

1. Aero에 모든 소프트웨어를 최신으로 업데이트하기 (https://github.com/intel-aero/meta-intel-aero/wiki/Upgrade-To-Latest-Software-Release)

2. [Firmware](https://github.com/PX4/Firmware) 소스를 가지고 오기

3. `make aerofc-v1_default` 명령으로 컴파일하기

4. hostname을 설정하기 (다음 IP주소로 여러분이 WiFi로 연결했다고 가정) :
```
export AERO_HOSTNAME=192.168.1.1`
```
5. `make aerofc-v1_default upload` 명령으로 업로드하기


## 네트워크로 QGroundControl 연결하기

1. WiFi 기능의 보드나 USB 네트워크에 연결되었는지 확인

2. 보드에 ssh로 연결하고 mavlink가 제대로 보내지는지 확인. 기본적으로 부팅하면 자동으로 시작됨. 수동으로 시작시킬려면 다음과 같이 :
```
/etc/init.d/mavlink_bridge.sh start
```

3. QGroundControl를 시작시키면 자동으로 연결되어야 한다.

4. QGroundControl을 시작시키는 대신에 [NuttX shell](advanced-system-console.md#mavlink-shell) 을 다음 명령으로 실행 :
```
./Tools/mavlink_shell.py 0.0.0.0:14550
```
