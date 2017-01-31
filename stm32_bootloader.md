# STM32 Bootloader

PX4 bootloader 코드는 Github [Bootloader](https://github.com/px4/bootloader) 저장소에서 확인할 수 있습니다.

## 지원 보드

  * FMUv1 (PX4FMU, STM32F4)
  * FMUv2 (Pixhawk 1, STM32F4)
  * FMUv3 (Pixhawk 2, STM32F4)
  * FMUv4 (Pixracer 3 and Pixhawk 3 Pro, STM32F4)
  * FMUv5 (Pixhawk 4, STM32F7)
  * TAPv1 (TBA, STM32F4)
  * ASCv1 (TBA, STM32F4)

## Bootloader 빌드하기

```bash
git clone https://github.com/PX4/Bootloader.git
cd Bootloader
make
```

이 단계를 거치면 지원하는 모든 보드의 elf 파일이 Bootloader 디렉토리에 생깁니다.

## Bootloader 플래쉬

> 중요: 일부 보드에서는 JTAG / SWD 접근을 위해서 전원들어오는 순서가 중요합니다. 다음 설명한 순서를 따라 진행합니다. 아래 절차는 Blackmagic / Dronecode probe에서 잘 동작합니다. 다른 JTAG probe의 경우 설명과 다를 수 있습니다. bootloader를 플래쉬하려는 개발자는 사전지식이 있어야 합니다. 만약 여러분이 잘 알지 못하는 경우라면 다시 고려해 보기 바랍니다. bootloader 변경이 진짜 필요한지를 생각하고 진행합니다.

  * JTAG 케이블 연결해제
  * USB 전원 케이블 연결
  * JTAG 케이블 연결

### Black Magic / Dronecode Probe

#### 알맞는 시리얼 포트 사용하기

  * LINUX에서 : ```/dev/serial/by-id/usb-Black_Sphere_XXX-if00```
  * MAC OS에서 : tty.xxx 포트가 아니라 cu.xxx 포트를 사용: ```tar ext /dev/tty.usbmodemDDEasdf```

```bash
arm-none-eabi-gdb
  (gdb) tar ext /dev/serial/by-id/usb-Black_Sphere_XXX-if00
  (gdb) mon swdp_scan
  (gdb) attach 1
  (gdb) mon option erase
  (gdb) mon erase_mass
  (gdb) load tapv1_bl.elf
        ...
        Transfer rate: 17 KB/sec, 828 bytes/write.
  (gdb) kill
```

### J-Link

These instructions are for the [J-Link GDB server](https://www.segger.com/jlink-gdb-server.html).

#### 전제 조건

Segger 웹사이트에서 [J-Link software 다운받고](https://www.segger.com/downloads/jlink#) 지시에 따라 설치하기

#### JLink GDB server 실행하기

FMUv1:
```bash
JLinkGDBServer -select USB=0 -device STM32F405RG -if SWD-DP -speed 20000
```

AeroFC:
```bash
JLinkGDBServer -select USB=0 -device STM32F429AI -if SWD-DP -speed 20000
```

#### GDB 연결

```bash
arm-none-eabi-gdb
  (gdb) tar ext :2331
  (gdb) load aerofcv1_bl.elf
```

### 문제해결

위에 명령을 찾을 수 없다면, Blackmagic probe가 아니거나 소프트웨어가 이전 버전일 수 있습니다. 먼저 on-probe 소프트웨어를 업그레이드 합니다.

이런 에러 메시지가 발생하면:
```Error erasing flash with vFlashErase packet```

타겟(연결한 JTAG은 그대로 두고)의 연결을 끊고 실행

```bash
mon tpwr disable
swdp_scan
attach 1
load tapv1_bl.elf
```
이렇게 하면 타겟의 전원을 차단하고 다시 flash를 시도합니다.
