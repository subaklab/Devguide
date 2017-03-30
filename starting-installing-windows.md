# Windows 설치 방법

> **경고** Windows 툴체인이 있긴하지만, 공식적으로 지원되는 것이 아니라 사용을 추천하지 않습니다. 펌웨어를 컴파일에 드는 시간이 오래 걸리고 Snapdragon Flight 같은 새로운 보드에 대해서 지원하지 않습니다. 많은 개발자가 컴퓨터 비전과 네비게이션 프로토타입으로 사용하는 표준 ROS 패키지를 사용할 수 없습니다. Windows에서 개발하기 전에, [Ubuntu](http://ubuntu.com)로 듀얼부트되는 환경을 고려해 보세요.

## 개발환경 설치

시스템에 다운로드 및 설치 :

  * [Qt Creator IDE](http://www.qt.io/download-open-source/#section-6)
  * [PX4 Toolchain Installer v14 for Windows Download](http://firmware.diydrones.com/Tools/PX4-tools/px4_toolchain_installer_v14_win.exe) (32/64 bit systems, complete build system, drivers)
  * [PX4 USB Drivers](http://pixhawk.org/static/px4driver.msi) (32/64 bit systems)

자 이제 [처음 빌드](starting-building.md)를 해봅시다!

## 새소식! Windows에서 Bash

Linux 빌드 명령을 사용할 수 있는 Bash shell 사용할 수 있게 되면서 Windows 사용자에게 새로운 선택사항이 생겼습니다. [BashOnWindows](https://github.com/Microsoft/BashOnWindows)를 참고하세요. 이 환경에서 px4 빌드가 성공적으로 되는 것을 확인하였습니다. 아직 펌웨어를 flash할 수는 없지만 Mission Planner나 QGroundControl을 사용하면 Windows에서도 커스텀 펌웨어를 flash할 수 있습니다.
