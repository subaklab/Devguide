# Bebop 2 - 고급

## FTDI 연결

다음 절차에 따라서 FTDI를 통해 Parrot Bebop 2에 연결합니다.
* 앞쪽 캡을 벗겨 내기 위해 2개 Torx 나사(T5)를 풉니다.
![](images/hardware/bebop_torx.JPG)
* 핀을 사용해서 ground/RX/TX에 연결하거나 커넥터에 케이블을 납떔합니다.
![](images/hardware/bebop_serial.JPG)
* FTDI 케이블을 연결하고 실행합니다.
```sh
screen /dev/ttyUSB0 115200
```
Bebop에 연결하기
![](images/hardware/bebop_ftdi.JPG)
