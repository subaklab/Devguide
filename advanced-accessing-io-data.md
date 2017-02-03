# I/O 데이터 접근

DSPAL이라는 POSIX유사 API를 이용하는 경우 aDSP에서 실행되는 코드로 low level bus 데이터에 접근할 수 있습니다. 이 API에 대한 헤더 파일은  [github](https://github.com/ATLFlight/dspal)에서 유지관리되고 각 헤더 파일에서 Doxygen 포맷으로 설명하고 있습니다. 지원하는 API의 설명과 적용가능한 헤더 파일에 대한 링크는 아래와 같습니다.

## API 개요
* [Serial:](https://github.com/ATLFlight/dspal/blob/master/include/dev_fs_lib_serial.h)
* [I2C:](https://github.com/ATLFlight/dspal/blob/master/include/dev_fs_lib_i2c.h)
* [SPI:](https://github.com/ATLFlight/dspal/blob/master/include/dev_fs_lib_spi.h)
* [GPIO:](https://github.com/ATLFlight/dspal/blob/master/include/dev_fs_lib_gpio.h)
* Timers: [qurt_timer.h](https://developer.qualcomm.com/software/hexagon-dsp-sdk/tools)
* Power Control: [HAP_power.h](https://developer.qualcomm.com/software/hexagon-dsp-sdk/tools)

## 샘플 소스코드
각 DSPAL 함수를 검증하는 단위 테스트 코드를 보면 함수 호출을 어떻게 해야하는지 잘 보여주고 있습니다.
샘플 코드는 [github](https://github.com/ATLFlight/dspal/tree/master/test/dspal_tester)를 참고합니다.

### Serial Data Rate 설정하기
시리얼 API는 tcsetattr() 함수를 통해 data rate를 설정하는 termios 규칙이 확정된 것은 아닙니다. IOCTL 코드가 대신 사용되며 위에 링크한 헤더 파일에서 설명하고 있습니다.

### Timers
좀더 고급 aDSP 연산을 위해서 추가한 함수들은 qurt_ 접두어로 시작합니다. 예를 들자면 타이머 함수는 qurt_timer 접수어로 시작하며, [Hexagon SDK](https://developer.qualcomm.com/software/hexagon-dsp-sdk/tools)에 포함된 qurt_timer.h 헤더 파일에서 찾아볼 수 있습니다.

### 파워 레벨 설정 (Setting the Power Level)
Hexagon SDK가 제공하는 HAP 함수를 사용하는 경우, aDSP의 파워 레벨을 설정할 수 있습니다. 이런 경우 I/O 지연을 줄일 수 있습니다. 이 API에 관한 정보는 [Hexagon SDK](https://developer.qualcomm.com/software/hexagon-dsp-sdk/tools)에 있는 HAP_power.h 헤더 파일에서 찾을 수 있습니다.
