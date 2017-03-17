# Snapdragon 고급

## Snapdragon에 연결하기

### FTDI 사용

Snapdragon 킷에 들어있는 작은 디버그 헤더와 FTDI 케이블 연결합니다.

리눅스에서 콘솔을 사용 :

```
screen /dev/ttyUSB0 115200
```

USB0을 알맞게 변경합니다. `/dev/`나 `/dev/serial/by-id`를 확인합니다.


### ADB(Android Debug Bridge) 사용

USB2.0으로 Snapdragon연결하고 power module을 사용해서 전원을 넣어줍니다.
Snapdragon이 실행될 때, LED는 청색으로 천천히 깜빡입니다.

adb를 사용하면 연결된 보드를 확인합니다. :

```
adb devices
```

장치를 확인하지 못하는 경우, USB 장치 permission과 관련된 문제일 수 있습니다. 아래 지시를 따라해 봅시다.

다음과 같이 쉘 얻기 :

```
adb shell
```

## Snapdragon 업그레이드

이 단계에서는 Intrynsic에서 Flight BSP zip 파일이 필요합니다. 보드 시리얼을 이용해서 등록해야 다운받기가 가능합니다.

### Linux 이미지 업그레이드/교체하기

<aside class="caution">Linux 이미지를 플래쉬하면 Snapdragon에 있는 모든 것이 삭제됩니다. 이 단계를 진핸하기 전에 작업한 자료를 백업합니다!</aside>

이제 adb를 사용해서 연결된 보드가 보이는지 확인합니다.:

```
adb devices
```

이제 fastboot bootloader로 리부팅합니다. :

```
adb reboot bootloader
```

fastboot을 이용해서 연결된 보드가 보이는지 확인합니다. :

```
fastboot devices
```

Intrinsyc에서 최신 BSP을 다운받습니다 :

```
unzip Flight_3.1.1_BSP_apq8074-00003.zip
cd BSP/binaries/Flight_BSP_4.0
./fastboot-all.sh
```

`recovery`, `update`, `factory`와 같은 partition은 동작하는 않는 것이 정상입니다.

### ADSP 펌웨어 업데이트하기

PX4 stack의 일부는 ADSP(Snapdragon 8074의 DSP부분)에서 동작하고 있습니다. 기반 OS는 QURT로 별도로 업데이트가 필요합니다.

<aside class="caution">ADSP 펌웨어 업데이트 중에 문제가 발생한다면, Snapdragon은 더이상 사용할 수 없게 될 수 있습니다! 따라서 이렇게 되지 않도록 아래 단계를 시도할때 각별한 주의가 필요합니다.</aside>

우선 BSP 3.1.1이 준비되어 있지 않았다면, [Linux image 업그레이드](#upgradingreplacing-the-linux-image)를 합니다!

#### 벽돌로 되는 것 방지하기
ADSP 펌웨어에 문제가 생기면 부팅에서 계속 멈춰있게 되는 문제가 생기므로 이를 방지하기 위해서 업데이트하기 전에 다음과 같이 변경해 줍니다. :

`screen`나 `adb shell` 상에서 Snapdragon에서 직접 파일을 수정 :
```sh
vim /usr/local/qr-linux/q6-admin.sh
```

아니면 파일을 로컬로 로드하고 에디터를 이용해서 수정 :

이렇게 하기 위해서, 로컬로 파일을 로드합니다. :
```sh
adb pull /usr/local/qr-linux/q6-admin.sh
```

수정하기 :

```sh
gedit q6-admin.sh
```

다음으로 되돌리기 :

```sh
adb push q6-admin.sh /usr/local/qr-linux/q6-admin.sh
adb shell chmod +x /usr/local/qr-linux/q6-admin.sh
```

부팅 상태로 멈춰있게 하는 원인이 while loop을 커멘트 처리 :

```
# Wait for adsp.mdt to show up
#while [ ! -s /lib/firmware/adsp.mdt ]; do
#  sleep 0.1
#done
```

그리고:

```
# Don't leave until ADSP is up
#while [ "`cat /sys/kernel/debug/msm_subsys/adsp`" != "2" ]; do
#  sleep 0.1
#done
```

#### 최신 ADSP 펌웨어 파일 넣기

Intrinsync에서 파일 다운받기 [Flight_3.1.1a_qcom_flight_controller_hexagon_sdk_add_on.zip](http://support.intrinsyc.com/attachments/download/691/Flight_3.1.1a_qcom_flight_controller_hexagon_sdk_add_on.zip).

그리고 Snapdragon에 복사하기:

```
unzip Flight_3.1.1a_qcom_flight_controller_hexagon_sdk_add_on.zip
cd images/8074-eagle/normal/adsp_proc/obj/qdsp6v5_ReleaseG/LA/system/etc/firmware
adb push . /lib/firmware
```

다음으로 정상적으로 리부팅하면 펌웨어가 적용됩니다 :

```
adb reboot
```


## 시리얼 포트 (Serial ports)

### 시리얼 포트 사용하기

모든 POSIX를 QURT에서 지원하지는 않습니다. 따라서 일부 커스텀 ioctl이 필요합니다.

셋업을 위한 API와 UART를 사용하는 방법을 [dspal](https://github.com/PX4/dspal/blob/master/include/dev_fs_lib_serial.h)에서 설명합니다.

## Wifi 셋팅

<aside class="todo">이 부분은 고급 개발자를 위한 부분입니다.</aside>

Linux shell에 연결하기 ( [console instructions](advanced-system-console.html#snapdragon-flight-wiring-the-console) 참고).

### point mode 접근

만약에 Snapdragon이 wifi access point(AP mode)로 동작하기를 원한다면 `/etc/hostapd.conf` 파일을 다음과 같이 수정 :

```
ssid=EnterYourSSID
wpa_passphrase=EnterYourPassphrase
```

다음으로 AP mode 설정 :

```
/usr/local/qr-linux/wificonfig.sh -s softap
reboot
```

### Station mode

Snapdragon으로 기존 wifi에 연결하고자 한다면, `/etc/wpa_supplicant/wpa_supplicant.conf` 파일에 네트워크 셋팅을 추가 :

```
network={
    ssid="my existing network ssid"
    psk="my existing password"
}
```

다음으로 station mode 설정 :

```
/usr/local/qr-linux/wificonfig.sh -s station
reboot
```


## 문제해결

### adb가 동작하지 않을 때

- [permissions](#usb-permissions) 확인
- 마이크로 USB 케이블에 문제 없는지 확임
- USB 2.0 포트로 시도
- 컴퓨터의 앞/뒤 포트들에 연결


### USB permissions

1) 새로운 permission 파일을 생성

```
sudo -i gedit /etc/udev/rules.d/51-android.rules
```

이 내용을 붙이면 널리 알려진 장치를 ADB access로 활성화 시킨다.

```
SUBSYSTEM=="usb", ATTRS{idVendor}=="0bb4", MODE="0666", GROUP="plugdev"
SUBSYSTEM=="usb", ATTRS{idVendor}=="0e79", MODE="0666", GROUP="plugdev"
SUBSYSTEM=="usb", ATTRS{idVendor}=="0502", MODE="0666", GROUP="plugdev"
SUBSYSTEM=="usb", ATTRS{idVendor}=="0b05", MODE="0666", GROUP="plugdev"
SUBSYSTEM=="usb", ATTRS{idVendor}=="413c", MODE="0666", GROUP="plugdev"
SUBSYSTEM=="usb", ATTRS{idVendor}=="0489", MODE="0666", GROUP="plugdev"
SUBSYSTEM=="usb", ATTRS{idVendor}=="091e", MODE="0666", GROUP="plugdev"
SUBSYSTEM=="usb", ATTRS{idVendor}=="18d1", MODE="0666", GROUP="plugdev"
SUBSYSTEM=="usb", ATTRS{idVendor}=="0bb4", MODE="0666", GROUP="plugdev"
SUBSYSTEM=="usb", ATTRS{idVendor}=="12d1", MODE="0666", GROUP="plugdev"
SUBSYSTEM=="usb", ATTRS{idVendor}=="24e3", MODE="0666", GROUP="plugdev"
SUBSYSTEM=="usb", ATTRS{idVendor}=="2116", MODE="0666", GROUP="plugdev"
SUBSYSTEM=="usb", ATTRS{idVendor}=="0482", MODE="0666", GROUP="plugdev"
SUBSYSTEM=="usb", ATTRS{idVendor}=="17ef", MODE="0666", GROUP="plugdev"
SUBSYSTEM=="usb", ATTRS{idVendor}=="1004", MODE="0666", GROUP="plugdev"
SUBSYSTEM=="usb", ATTRS{idVendor}=="22b8", MODE="0666", GROUP="plugdev"
SUBSYSTEM=="usb", ATTRS{idVendor}=="0409", MODE="0666", GROUP="plugdev"
SUBSYSTEM=="usb", ATTRS{idVendor}=="2080", MODE="0666", GROUP="plugdev"
SUBSYSTEM=="usb", ATTRS{idVendor}=="0955", MODE="0666", GROUP="plugdev"
SUBSYSTEM=="usb", ATTRS{idVendor}=="2257", MODE="0666", GROUP="plugdev"
SUBSYSTEM=="usb", ATTRS{idVendor}=="10a9", MODE="0666", GROUP="plugdev"
SUBSYSTEM=="usb", ATTRS{idVendor}=="1d4d", MODE="0666", GROUP="plugdev"
SUBSYSTEM=="usb", ATTRS{idVendor}=="0471", MODE="0666", GROUP="plugdev"
SUBSYSTEM=="usb", ATTRS{idVendor}=="04da", MODE="0666", GROUP="plugdev"
SUBSYSTEM=="usb", ATTRS{idVendor}=="05c6", MODE="0666", GROUP="plugdev"
SUBSYSTEM=="usb", ATTRS{idVendor}=="1f53", MODE="0666", GROUP="plugdev"
SUBSYSTEM=="usb", ATTRS{idVendor}=="04e8", MODE="0666", GROUP="plugdev"
SUBSYSTEM=="usb", ATTRS{idVendor}=="04dd", MODE="0666", GROUP="plugdev"
SUBSYSTEM=="usb", ATTRS{idVendor}=="0fce", MODE="0666", GROUP="plugdev"
SUBSYSTEM=="usb", ATTRS{idVendor}=="0930", MODE="0666", GROUP="plugdev"
SUBSYSTEM=="usb", ATTRS{idVendor}=="19d2", MODE="0666", GROUP="plugdev"
```

해당 파일에 대해서 올바른 permission을 설정 :

```
sudo chmod a+r /etc/udev/rules.d/51-android.rules
```

daemon 재시작

```
sudo udevadm control --reload-rules
sudo service udev restart
sudo udevadm trigger
```

만약 여전히 동작하지 않는 경우라면 [this answer on StackOverflow](http://askubuntu.com/questions/461729/ubuntu-is-not-detecting-my-android-device#answer-644222)를 참조합니다.


### Board가 구동되지 않는 경우 / 무한부팅 / 멈춘 상태 유지

시리얼 콘솔을 사용해서 보드에 연결이 되고 다음과 같은 프롬프트가 보인다면 :

```
root@linaro-developer:~#
```

fastboot(bootloader) mode로 진입 :

```
reboot2fastboot
```

만약 시리얼 콘솔이 불가능하다면 마이크로 USB 케이블을 연결하여 진입 :

```
adb wait-for-device && adb reboot bootloader
```

다음으로 보드에 전원 공급합니다. 운이 좋다면 바로 adb로 연결되고 보드를 fastboot로 되게 합니다..

fastboot mode 상태인지 확인하는 방법 :

```
fastboot devices
```

일단 fastboot mode로 진입하면,  [above teps](#upgradingreplacing-the-linux-image)를 참고하여 Android/Linux 이미지를 업데이트합니다.

[P2 board](#do-i-have-a-p1-or-p2-board)가 있다면, Snapdragon을 구동시킬 때 복구 이미지로 Snapdragon을 리셋시킬 수 있습니다. J3라고 쓰여진 곳 옆에 2개 pin을 쇼트시킵니다. (코너에 있는 구멍과 MicroSD카드 슬롯 사이에 2개의 정사각형 pin)

모든 것이 다 실패라면 intrinsyc에 도움을 요청해야 합니다.


### 장치에 남은 공간이 없는 경우

가끔 `make eagle_default upload` 명령이 실패 :

```
failed to copy 'px4' to '/home/linaro/px4': No space left on device
```

ramdumps가 디스크 공간을 다 채우는 경우 발생합니다. 지우는 방법 :

```
rm -rf /var/log/ramdump/*
```

또한 log로 공간이 다찬 경우입니다. 삭제하는 방법 :

```
rm -rf /root/log/*
```

### Undefined PLT symbol

#### _FDtest

px4 프로그램을 구동시킬때, mini-dm에서 다음과 같은 출력을 보게 되는 경우 [update the ADSP firmware](#updating-the-adsp-firmware)와 같이 업그레이드가 필요하다는 뜻입니다 :

```
[08500/03]  05:10.960  HAP:45:undefined PLT symbol _FDtest (689) /libpx4muorb_skel.so  0303  symbol.c
```

#### 그 이외

소스를 변경한 경우, 함수가 추가되어 `undefined PLT symbol ...`를 보게 되는 경우가 있는데 이는 linking이 실패한 경우입니다.

- 여러분이 작성한 함수는 1:1로 선언과 정의가 매칭시켰는가?
- 여러분의 코드는 정말 컴파일이 되는가?
[cmake config](https://github.com/PX4/Firmware/blob/master/cmake/configs/qurt_eagle_default.cmake)에 찾을 수 있는 모듈인가?
- 추가한 파일이 `CMakeLists.txt`에 포함되어 있는가?
- POSIX 블드에 추가해보고 컴파일을 실행해 봅니다. POSIX linker가 compile/linking 할때 linking 에러를 알려줍니다.

### 구동시 에러 메시지 : krait update param XXX failed

```
ERROR [platforms__posix__px4_layer] krait update param 297 failed
ERROR [platforms__posix__px4_layer] krait update param 646 failed

px4 starting.
ERROR [muorb] Initialize Error calling the uorb fastrpc initalize method..
ERROR [muorb] ERROR: FastRpcWrapper Not Initialized
```

px4를 시작할 때 위와 같은 에러를 보게 된다면, 다음을 시도해 봅니다
- [Linux image 업그레이드](#upgradingreplacing-the-linux-image)
- 그리고 [ADSP 펌웨어 업데이트](#updating-the-adsp-firmware). 가상머신 대신에 실제 Linux가 설치된 상태에서 시도합니다. 가산머신을 이용하는 경우 동작하는 않는 [reports](https://github.com/PX4/Firmware/issues/5303)가 있습니다.
- 다음으로 [px4 소프트웨어를 재빌드](http://dev.px4.io/starting-building.html#building-px4-software)하는데, 처음에 완전히 기존 Firmware repo를 삭제하고 다시 clone합니다. [참조](http://dev.px4.io/starting-building.html#compiling-on-the-console)
- 마지막으로 [재빌드 및 재실행](http://dev.px4.io/starting-building.html#qurt--snapdragon-based-boards)
- `/usr/local/qr-linux/q6-admin.sh`의 실행부가 다음과 같이 설정되어 있는지 확인 :
  `adb shell chmod +x /usr/local/qr-linux/q6-admin.sh`

### ADSP 재시작 시키기

만약 mini-dm 콘솔이 갑자기 엄청나게 많은 INIT을 출력한다면 ADSP쪽에 문제가 생긴 것입니다. 이에 대한 이유는 명확하지는 않습니다. 세그먼테이션 폴트나 null 포인터 에러 등의 원인 때문일 수 있습니다.

mini-dm 콘솔의 출력은 일반적으로 다음과 같은 형태 :

```
[08500/02]  20:32.332  Process Sensor launched with ID=1   0130  main.c
[08500/02]  20:32.337  mmpm_register: MMPM client for USM ADSP core 12  0117  UltrasoundStreamMgr_Mmpm.cpp
[08500/02]  20:32.338  ADSP License DB: License validation function with id 164678 stored.  0280  adsp_license_db.cpp
[08500/02]  20:32.338  AvsCoreSvc: StartSvcHandler Enter  0518  AdspCoreSvc.cpp
[08500/02]  20:32.338  AdspCoreSvc: Started successfully  0534  AdspCoreSvc.cpp
[08500/02]  20:32.342  DSPS INIT  0191  sns_init_dsps.c
[08500/02]  20:32.342  INIT DONE  0224  sns_init_dsps.c
[08500/02]  20:32.342  Sensors Init : waiting(1)  0160  sns_init_dsps.c
[08500/02]  20:32.342  INIT DONE  0224  sns_init_dsps.c
[08500/02]  20:32.342  THRD CREATE: Thread=0x39 Name(Hex)= 53, 4e, 53, 5f, 53, 4d, 47, 52  0186  qurt_elite_thread.cpp
[08500/02]  20:32.343  THRD CREATE: Thread=0x38 Name(Hex)= 53, 4e, 53, 5f, 53, 41, 4d, 0  0186  qurt_elite_thread.cpp
[08500/02]  20:32.343  THRD CREATE: Thread=0x37 Name(Hex)= 53, 4e, 53, 5f, 53, 43, 4d, 0  0186  qurt_elite_thread.cpp
[08500/02]  20:32.343  THRD CREATE: Thread=0x35 Name(Hex)= 53, 4e, 53, 5f, 50, 4d, 0, 0  0186  qurt_elite_thread.cpp
[08500/02]  20:32.343  THRD CREATE: Thread=0x34 Name(Hex)= 53, 4e, 53, 5f, 53, 53, 4d, 0  0186  qurt_elite_thread.cpp
[08500/02]  20:32.343  THRD CREATE: Thread=0x33 Name(Hex)= 53, 4e, 53, 5f, 44, 45, 42, 55  0186  qurt_elite_thread.cpp
[08500/02]  20:32.343  Sensors Init : ///////////init once completed///////////  0169  sns_init_dsps.c
[08500/02]  20:32.342  loading BLSP configuration  0189  blsp_config.c
[08500/02]  20:32.343  Sensors DIAG F3 Trace Buffer Initialized  0260  sns_init_dsps.c
[08500/02]  20:32.345  INIT DONE  0224  sns_init_dsps.c
[00053/03]  20:32.345  Unsupported algorithm service id 0  0953  sns_scm_ext.c
[08500/02]  20:32.346  INIT DONE  0224  sns_init_dsps.c
[08500/02]  20:32.347  INIT DONE  0224  sns_init_dsps.c
[08500/02]  20:32.347  INIT DONE  0224  sns_init_dsps.c
[08500/02]  20:32.546  HAP:159:unable to open the specified file path  0167  file.c
[08500/04]  20:32.546  failed to open /usr/share/data/adsp/blsp.config  0204  blsp_config.c
[08500/04]  20:32.546  QDSP6 Main.c: blsp_config_load() failed  0261  main.c
[08500/02]  20:32.546  Loaded default UART-BAM mapping  0035  blsp_config.c
[08500/02]  20:32.546  UART tty-1: BAM-9  0043  blsp_config.c
[08500/02]  20:32.546  UART tty-2: BAM-6  0043  blsp_config.c
[08500/02]  20:32.546  UART tty-3: BAM-8  0043  blsp_config.c
[08500/02]  20:32.546  UART tty-4: BAM-2  0043  blsp_config.c
[08500/02]  20:32.546  UART tty-5: BAM N/A  0048  blsp_config.c
[08500/02]  20:32.546  UART tty-6: BAM N/A  0048  blsp_config.c
[08500/02]  20:32.547  HAP:111:cannot find /oemconfig.so  0141  load.c
[08500/03]  20:32.547  HAP:4211::error: -1: 0 == dynconfig_init(&conf, "security")   0696  sigverify.c
[08500/02]  20:32.548  HAP:76:cannot find /voiceproc_tx.so  0141  load.c
[08500/02]  20:32.550  HAP:76:cannot find /voiceproc_rx.so  0141  load.c
```

### P1이나 P2 보드를 가지고 있나?Do I have a P1 or P2 board?

Snapdragon에 실크스크린된 다음과 같은 부분을 읽어봅시다 :

```
1DN14-25-
H9550-P1
REV A
QUALCOMM
```

만약 **H9550**가 보이면 P2 보드입니다.

**-P1 부분은 무시해도 됩니다.**

Presumably P1 boards don't have a factory partition/image and therefore can't be restored to factory state.
