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

모든 POSIX 호출
Not all POSIX calls are currently supported on QURT. Therefore, some custom ioctl are needed.

The APIs to set up and use the UART are described in [dspal](https://github.com/PX4/dspal/blob/master/include/dev_fs_lib_serial.h).

## Wifi-settings

<aside class="todo">These are notes for advanced developers.</aside>

Connect to the Linux shell (see [console instructions](advanced-system-console.html#snapdragon-flight-wiring-the-console)).

### Access point mode

If you want the Snapdragon to be a wifi access point (AP mode), edit the file: `/etc/hostapd.conf` and set:

```
ssid=EnterYourSSID
wpa_passphrase=EnterYourPassphrase
```

Then configure AP mode:

```
/usr/local/qr-linux/wificonfig.sh -s softap
reboot
```

### Station mode

If you want the Snapdragon to connect to your existing wifi, edit the file: `/etc/wpa_supplicant/wpa_supplicant.conf` and add your network settings:

```
network={
    ssid="my existing network ssid"
    psk="my existing password"
}
```

Then configure station mode:

```
/usr/local/qr-linux/wificonfig.sh -s station
reboot
```


## Troubleshooting

### adb does not work

- Check [permissions](#usb-permissions)
- Make sure you are using a working Micro-USB cable.
- Try a USB 2.0 port.
- Try front and back ports of your computer.


### USB permissions

1) Create a new permissions file

```
sudo -i gedit /etc/udev/rules.d/51-android.rules
```

paste this content, which enables most known devices for ADB access:

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

Set up the right permissions for the file:

```
sudo chmod a+r /etc/udev/rules.d/51-android.rules
```

Restart the deamon

```
sudo udevadm control --reload-rules
sudo service udev restart
sudo udevadm trigger
```

If it still doesn't work, check [this answer on StackOverflow](http://askubuntu.com/questions/461729/ubuntu-is-not-detecting-my-android-device#answer-644222).


### Board doesn't start / is boot-looping / is bricked

If you can still connect to the board using the serial console and get to a prompt such as:

```
root@linaro-developer:~#
```

You can get into fastboot (bootloader) mode by entering:

```
reboot2fastboot
```

If the serial console is not possible, you can try to connect the Micro USB cable, and enter:

```
adb wait-for-device && adb reboot bootloader
```

Then power cycle the board. If you're lucky, adb manages to connect briefly and can send the board into fastboot.

To check if it's in fastboot mode, use:

```
fastboot devices
```

Once you managed to get into fastboot mode, you can try [above teps](#upgradingreplacing-the-linux-image) to update the Android/Linux image.

If you happen to have a [P2 board](#do-i-have-a-p1-or-p2-board), you should be able to reset the Snapdragon to the recovery image by starting up the Snapdragon while shorting the two pins next to where J3 is written (The two rectangular pins in-between the corner hole and the MicroSD card slot almost at the edge of the board.

If everything fails, you probably need to request help from intrinsyc.


### No space left on device

Sometimes `make eagle_default upload` fails to upload:

```
failed to copy 'px4' to '/home/linaro/px4': No space left on device
```

This can happen if ramdumps fill up the disk. To clean up, do:

```
rm -rf /var/log/ramdump/*
```

Also, the logs might have filled the space. To delete them, do:

```
rm -rf /root/log/*
```

### Undefined PLT symbol

#### _FDtest

If you see the following output on mini-dm when trying to start the px4 program, it means that you need to [update the ADSP firmware](#updating-the-adsp-firmware):

```
[08500/03]  05:10.960  HAP:45:undefined PLT symbol _FDtest (689) /libpx4muorb_skel.so  0303  symbol.c
```

#### Something else

If you have changed the source, presumably added functions and you see `undefined PLT symbol ...` it means that the linking has failed.

- Do the declaration and definition of your function match one to one?
- Is your code actually getting compiled?
Is the module listed in the [cmake config](https://github.com/PX4/Firmware/blob/master/cmake/configs/qurt_eagle_default.cmake)?
- Is the (added) file included in the `CMakeLists.txt`?
- Try adding it to the POSIX build and running the compilation. The POSIX linker will inform you about linking errors at compile/linking time.

### krait update param XXX failed on startup

```
ERROR [platforms__posix__px4_layer] krait update param 297 failed
ERROR [platforms__posix__px4_layer] krait update param 646 failed

px4 starting.
ERROR [muorb] Initialize Error calling the uorb fastrpc initalize method..
ERROR [muorb] ERROR: FastRpcWrapper Not Initialized
```

If you get errors like the above when starting px4, try
- [upgrading the Linux image](#upgradingreplacing-the-linux-image)
- and [updating the ADSP firmware](#updating-the-adsp-firmware). Also try to do this from a native Linux installation instead of a virtual machine. There have been [reports](https://github.com/PX4/Firmware/issues/5303) where it didn't seem to work when done in a virtual machine.
- then [rebuild the px4 software](http://dev.px4.io/starting-building.html#building-px4-software), by first completely deleting your existing Firmware repo and then recloning it [as described here](http://dev.px4.io/starting-building.html#compiling-on-the-console)
- and finally [rebuild and re-run it](http://dev.px4.io/starting-building.html#qurt--snapdragon-based-boards)
- make sure the executable bit of `/usr/local/qr-linux/q6-admin.sh` is set:
  `adb shell chmod +x /usr/local/qr-linux/q6-admin.sh`

### ADSP restarts

If the mini-dm console suddently shows a whole lot of INIT output, the ADSP side has crashed. The reasons for it are not obvious, e.g. it can be some segmentation fault, null pointer exception, etc..

The mini-dm console output typically looks like this:

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

### Do I have a P1 or P2 board?

The silkscreen on the Snapdragon reads something like:

```
1DN14-25-
H9550-P1
REV A
QUALCOMM
```

If you see **H9550**, it means you have a P2 board!

**Please ignore that it says -P1.**

Presumably P1 boards don't have a factory partition/image and therefore can't be restored to factory state.
