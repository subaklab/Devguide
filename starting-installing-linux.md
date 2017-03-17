# Linux에서 개발환경

Linux 배포판 중에 Debian / Ubuntu LTS를 표준으로 정했습니다. 하지만 Cent OS나 Arch Linux도 사용 가능합니다. 참고 : [boutique distribution instructions](starting-installing-linux-boutique.md)

## Permission 설정

> **경고** 'sudo'를 이용해서 permission 문제를 해결하지 마세요. 이로 인해 더 많은 permission 문제가 생길수도 있고 시스템을 다시 설치해야할수도 있습니다.

user는 "dialout" 그룹의 일원이 되어야 합니다. :

```sh
sudo usermod -a -G dialout $USER
```

다음으로 logout을 하고 다시 login합니다. 새로 login해야만 변경사항에 적용됩니다.

## 설치

패키지 목록을 업데이트하고 모든 PX4 빌드 타겟을 위해 필요한 의존하는 패키지를 설치합니다. PX4가 지원하는 4개 계열 :

* NuttX 기반 하드웨어: [Pixhawk](hardware-pixhawk.md), [Pixfalcon](hardware-pixfalcon.md),
  [Pixracer](hardware-pixracer.md), [Crazyflie](hardware-crazyflie2.md),
  [Intel Aero](hardware-intel-aero.md)
* Snapdragon Flight 하드웨어: [Snapdragon](hardware-snapdragon.md)
* Linux 기반 하드웨어: [Raspberry Pi 2/3](hardware-rpi.md), Parrot Bebop
* 호스트 simulation: [jMAVSim SITL](simulation-sitl.md)와 [Gazebo SITL](simulation-gazebo.md)

> **정보** Make보다 빌드를 빠르게 하려면 [Ninja Build System](http://dev.px4.io/starting-installing-linux-boutique.html#ninja-build-system)을 설치합니다. 설치되어 있다면 자동으로 선택해서 빌드합니다.

```sh
sudo add-apt-repository ppa:george-edison55/cmake-3.x -y
sudo apt-get update
sudo apt-get install python-argparse git-core wget zip \
    python-empy qtcreator cmake build-essential genromfs -y
# simulation tools
sudo apt-get install ant protobuf-compiler libeigen3-dev libopencv-dev openjdk-8-jdk openjdk-8-jre clang-3.5 lldb-3.5 -y
```

### NuttX 기반 하드웨어

Ubuntu에 기본으로 serial modem manager가 설치되어 있어서 로보틱스 관련해서 시리얼 포트나 \( USB 시리얼\) 을 사용하는 경우 방해가 됩니다. 따라서 효과적으로 삭제하는 방법은 :

```sh
sudo apt-get remove modemmanager
```

패키지 목록을 업데이트하고 다음과 같이 의존하는 패키지를 설치합니다. 특정 버전이 필요한 패키지는 해당 버전의 패키지를 설치해야만 합니다.

```sh
sudo apt-get install python-serial openocd \
    flex bison libncurses5-dev autoconf texinfo build-essential \
    libftdi-dev libtool zlib1g-dev \
    python-empy  -y
```

arm-none-eabi 툴체인을 추가하기 전에 이전에 설치된 것을 제거합니다.

```sh
sudo apt-get remove gcc-arm-none-eabi gdb-arm-none-eabi binutils-arm-none-eabi gcc-arm-embedded
sudo add-apt-repository --remove ppa:team-gcc-arm-embedded/ppa
```

4.9나 5.4 버전의 arm-none-eabi 툴체인을 설치할때 [toolchain installation instructions](http://dev.px4.io/starting-installing-linux-boutique.html#toolchain-installation)을 참고합니다.

### Snapdragon Flight

#### Toolchain 설치

```sh
sudo apt-get install android-tools-adb android-tools-fastboot fakechroot fakeroot unzip xz-utils wget python python-empy -y
```

```sh
git clone https://github.com/ATLFlight/cross_toolchain.git
```

QDN에서 Hexagon SDK 3.0 가져오기 : [https://developer.qualcomm.com/download/hexagon/hexagon-sdk-v3-linux.bin](https://developer.qualcomm.com/download/hexagon/hexagon-sdk-v3-linux.bin)

QDN 로그인이 필요합니다. 계정이 없다면 우선 등록해야 합니다.

이제 cross 툴체인 폴더로 옮깁니다. :

```sh
mv ~/Downloads/hexagon-sdk-v3-linux.bin cross_toolchain/downloads
```

툴체인과 SDK를 다음과 같이 설치 :

```sh
cd cross_toolchain
./installv3.sh
cd ..
```

다음은 환경설정을 위한 방법을 설명합니다. 기본 설치를 하고자 한다면 다음 환경설정을 따라서 실행하면 됩니다. 다시 실행하더라도 빠진 컴포넌트만 설치됩니다.

도구와 SDK는 "$HOME/Qualcomm/..."에 설치됩니다. ~/.bashrc 에 다음을 추가합니다 :

```sh
export HEXAGON_SDK_ROOT="${HOME}/Qualcomm/Hexagon_SDK/3.0"
export HEXAGON_TOOLS_ROOT="${HOME}/Qualcomm/HEXAGON_Tools/7.2.12/Tools"
export PATH="${HEXAGON_SDK_ROOT}/gcc-linaro-4.9-2014.11-x86_64_arm-linux-gnueabihf_linux/bin:$PATH"
```

새로운 설정을 로드합니다 :

```sh
source ~/.bashrc
```

#### sysroot 설치

Snapdragon Flight AP에 대해 application 개발시에 cross 컴파일에 필요한 라이브러리와 헤더 파일을 제공하는데 필요한 것이 바로 sysroot입니다.

qrlSDK sysroot은 카메라, GPU 등에 필요한 라이브러리와 헤더 파일을 제공합니다.

[Flight\_3.1.1\_qrlSDK.zip](http://support.intrinsyc.com/attachments/download/690/Flight_3.1.1_qrlSDK.zip) 파일을 다운받아서 `cross_toolchain/download/`에 저장합니다.

```sh
cd cross_toolchain
unset HEXAGON_ARM_SYSROOT
./qrlinux_sysroot.sh
```

다음을 ~/.bashrc 에 추가합니다. :

```sh
export HEXAGON_ARM_SYSROOT=${HOME}/Qualcomm/qrlinux_v3.1.1_sysroot
```

새로운 설정을 로드합니다. :

```sh
source ~/.bashrc
```

추가적인 sysroot 옵션은 [Sysroot Installation](https://github.com/ATLFlight/cross_toolchain/blob/sdk3/README.md#sysroot-installation)을 참고합니다.

#### ADSP 펌웨어 업데이트

빌드, flash, 코드실행 하기 전에, [ADSP firmware](advanced-snapdragon.html#updating-the-adsp-firmware)를 업데이트합니다.

#### 참고

Snapdragon Flight 툴체인과 SW 설정 그리고 검증에 관련된 자료 : [ATLFlightDocs](https://github.com/ATLFlight/ATLFlightDocs/blob/master/README.md)

DSP에서 보내는 메시지는 mini-dm을 이용해서 볼 수 있습니다.

```sh
${HEXAGON_SDK_ROOT}/tools/debug/mini-dm/Linux_Debug/mini-dm
```
노트: Mac에서는 [nano-dm](https://github.com/kevinmehall/nano-dm)를 사용할 수도 있습니다.

### Raspberry Pi 하드웨어

Raspberry Pi 하드웨어를 사용하는 개발자는 RPi Linux 툴체인을 아래 링크에서 다운받습니다. 설치 스크립트를 이용하면 자동으로 cross 컴파일러 툴체인이 설치됩니다. 직접 컴파일을 목적으로 _native_ Raspberry Pi 툴체인을 사용하고자 한다면  [here](http://dev.px4.io/hardware-pi2.html#native-builds-optional)를 참고하세요.

```sh
git clone https://github.com/pixhawk/rpi_toolchain.git
cd rpi_toolchain
./install_cross.sh
```

툴체인 설치를 위해 패스워드를 입력이 필요합니다.

`/opt/rpi_toolchain`의 기본 위치에 툴체인을 설치하지 않으려면 설치 스크립트에 다른 경로를 전달할 수 있습니다. `./install_cross.sh <PATH>`를 실행합니다. 인스톨러는 자동으로 환경변수를 설정합니다.

마지막으로 환경변수를 업데이트하기 위해 다음 명령을 실행합니다. :
```
source ~/.profile
```

### Parrot Bebop

Parrot Bebop으로 개발하는 경우 RPi Linux 툴체인을 설치해야만 합니다. [Raspberry Pi 하드웨어](raspberry-pi-hardware)에 있는 설명을 따라 진행합니다.

다음으로 ADB를 설치합니다.

``sh
sudo apt-get install android-tools-adb -y` ``

## 마무리

자 이제 [첫 빌드](starting-building.md)를 진행할 수 있습니다!
