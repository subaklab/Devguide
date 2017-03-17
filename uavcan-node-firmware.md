# UAVCAN 펌웨어 업그레이드

## Vectorcontrol ESC Codebase (Pixhawk ESC 1.6과 S2740VC)

ESC 코드 다운로드 :

<div class="host-code"></div>

```sh
git clone https://github.com/thiemar/vectorcontrol
cd vectorcontrol
```

### Flashing the UAVCAN Bootloader

UAVCAN을 통해 펌웨어를 업데이트를 진행하기 전에, Pixhawk ESC 1.6은 flash를 마친 UAVCAN bootloader가 필요합니다. bootloader를 빌드하기 위해 다음을 실행 :

<div class="host-code"></div>

```sh
make clean && BOARD=px4esc_1_6 make -j8
```

빌드를 마치고 bootloader 이미지가 `firmware/px4esc_1_6-bootloader.bin`에 생깁니다. OpenOCD 설정은 `openocd_px4esc_1_6.cfg`에 있습니다. bootloader를 ESC에 설치하기 위해 [these instructions](uavcan-bootloader-installation.md)을 따라합니다.

### 메인 바이너리 컴파일하기

<div class="host-code"></div>

```sh
BOARD=s2740vc_1_0 make && BOARD=px4esc_1_6 make
```

지원하는 ESC에 대해서 UAVCAN 노드 펌웨어를 빌드합니다. 이 펌웨어 이미지는 `com.thiemar.s2740vc-v1-1.0-1.0.<git hash>.bin`와 `org.pixhawk.px4esc-v1-1.6-1.0.<git hash>.binn`에 생깁니다.

## Sapog Codebase (Pixhawk ESC 1.4)

Sapog codebase 다운로드 :

<div class="host-code"></div>

```sh
git clone https://github.com/PX4/sapog
cd sapog
git submodule update --init --recursive
```

### Flashing the UAVCAN Bootloader

UAVCAN을 통해 펌웨어를 업데이트를 진행하기 전에, Pixhawk ESC 1.6은 flash를 마친 UAVCAN bootloader가 필요합니다. bootloader를 빌드하기 위해 다음을 실행 :

<div class="host-code"></div>

```sh
cd bootloader
make clean && make -j8
cd ..
```

bootloader 이미지가 `bootloader/firmware/bootloader.bin`에 생깁니다. OpenOCD 설정은 `openocd.cfg`에 있습니다. bootloader를 ESC에 설치하기 위해 [these instructions](uavcan-bootloader-installation.md)을 따라합니다.

### 메인 바이너리 컴파일하기

<div class="host-code"></div>

```sh
cd firmware
make sapog.image
```
펌웨어 이미지는 `firmware/build/org.pixhawk.sapog-v1-1.0.<xxxxxxxx>.bin`에 생성됩니다. `<xxxxxxxx>`는 숫자와 문자로된 임의의 순서를 나타냅니다.

## Zubax GNSS

펌웨어 빌드와 flash 방법은 [project page](https://github.com/Zubax/zubax_gnss) 사이트를 참고합니다.
Zubax GNSS는 UAVCAN 가능한 bootloader가 들어가 있으며, 이 펌웨어는 UAVCAN을 통해 아래 설명과 같이 동일한 방식으로 업데이트할 수 있습니다.

## Autopilot에 Firmware 설치

Pixhawk가 네트워크에 연결된 모든 UAVCAN 장치를 업데이트할려면 UAVCAN 노드 파일 이름의 규칙을 따라야 합니다. 위에 단게에서 생성된 펌웨어 파일은 SD 카드에 올바른 위치 혹은 업데이트할 장치를 위해 PX4 ROMFS에 복사해야만 합니다.

펌웨어 이미지 이름의 규칙은 :

  ```<uavcan name>-<hw version major>.<hw version minor>-<sw version major>.<sw version minor>.<version hash>.bin```

  예제 ```com.thiemar.s2740vc-v1-1.0-1.0.68e34de6.bin```

하지만 공간/성능 제약으로(28 글자를 넘지 않음) UAVCAN 펌웨어 업데이터는 파일들을 다음과 같은 디렉토리 구조에 분리해서 저장합니다. :

  ```/fs/microsd/fw/<node name>/<hw version major>.<hw version minor>/<hw name>-<sw version major>.<sw version minor>.<git hash>.bin```

 예제 ```s2740vc-v1-1.0.68e34de6.bin```

ROMFS 기반 업데이터는 패턴을 따르지만, 파일이름에 ```_```를 붙이기 때문에 다음과 같습니다. :

  ```/etc/uavcan/fw/<device name>/<hw version major>.<hw version minor>/_<hw name>-<sw version major>.<sw version minor>.<git hash>.bin```

## PX4 ROMFS에 바이너리 넣기

최종 파일의 위치는 :

  * S2740VC ESC: `ROMFS/px4fmu_common/uavcan/fw/com.thiemar.s2740vc-v1/1.0/_s2740vc-v1-1.0.<git hash>.bin`
  * Pixhawk ESC 1.6: `ROMFS/px4fmu_common/uavcan/fw/org.pixhawk.px4esc-v1/1.6/_px4esc-v1-1.6.<git hash>.bin`
  * Pixhawk ESC 1.4: `ROMFS/px4fmu_common/uavcan/fw/org.pixhawk.sapog-v1/1.4/_sapog-v1-1.4.<git hash>.bin``
  * Zubax GNSS v1: `ROMFS/px4fmu_common/uavcan/fw/com.zubax.gnss/1.0/gnss-1.0.<git has>.bin`
  * Zubax GNSS v2: `ROMFS/px4fmu_common/uavcan/fw/com.zubax.gnss/2.0/gnss-2.0.<git has>.bin`

ROMFS/px4fmu_common 디렉토리는 Pixhawk에 /etc에 마운트된다는 것을 명심하세요.

### 펌웨어 업그레이드 절차 시작하기

<aside class="note">
[PX4 Flight Stack](concept-flight-stack.md)를 사용할 때, 'Power Config' 섹션에서 UAVCAN을 활성화시키고 UAVCAN 펌웨어 업그레이드를 시도하기 전에 시스템을 reboot합니다.
</aside>

UAVCAN 펌웨어 업그레이드는 수동으로 NSH에서 다음과 같이 진행할 수도 있습니다. :

```sh
uavcan start
uavcan start fw
```
