# uLanding Radar

uLanding radar는 [Aerotenna](http://aerotenna.com/sensors/)의 제품이며 물체와의 거리를 측정하는데 사용할 수 있습니다.


## Enable the driver for your hardware
현재 이 radar 장치는 NuttX 운영체제가 돌아가는 하드웨어에서 동작하며 인터페이스로 시리얼 포트를 제공합니다. 일부 하드웨에에서 flash 공간이 작으면 여러분이 직접 타겟에 맞는 드라이버를 개발해야 합니다. 빌드하려는 대상에 따라서 cmake config 파일에 다음 라인을 추가하면 됩니다.:
```
drivers/ulanding
```

모든 config 파일은 [here.](https://github.com/PX4/Firmware/tree/master/cmake/configs)에 위치하고 있습니다.

## driver 구동시키기
시스템이 처음 시작할 때 radar의 driver를 구동시키라고 시스템에게 전달해야 합니다. 간단하게 SD카드에 있는 [extras.txt](advanced-system-startup.md) 파일에 다음 라인을 추가하면 됩니다.
```
ulanding_radar start /dev/serial_port
```

위 명령에서 마지막 인자를 하드웨어를 연결하는 시리얼 포트로 대체해야 합니다.
따로 지정을 하지 않으면 Pixhawk에 TELEM2 포트인 /dev/ttyS2를 driver가 사용합니다.

**경고**

radar 장치를 TELEM2하는 경우에 SYS_COMPANION parameter를 0으로 설정했는지 확인합니다. 그렇지 않으면 serial port는 다른 application에서 사용하게 되어 예상치 못한 동작을 하게 될 수 있습니다.
