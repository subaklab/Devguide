# PX4 Continuous Integration

PX4 빌드와 테스팅은 여러 CI 서비스들 위에서 동작합니다.

## [Travis-ci](https://travis-ci.org/PX4/Firmware)

Travis-ci는 [QGroundControl](http://qgroundcontrol.com/)을 통해 플래쉬 가능한 공식 stable/beta/development 바이너리를 책임지고 있습니다. 현재 docker 이미지 [px4io/px4-dev-base](https://hub.docker.com/r/px4io/px4-dev-base/)에 포함된 GCC 4.9.3을 사용하며 makefile target qgc_firmware로 px4fmu-{v1, v2, v4}, mindpx-v2, tap-v1을 컴파일 합니다.

Travis-ci는 테스팅을 포함하고 있는 MacOS에서 동작하는 posix sitl 빌드를 포함하고 있습니다.

## [Semaphore](https://semaphoreci.com/px4/firmware)

세마포어는 주로 Qualcomm Snapdragon 플랫폼에 대해서 변경 내용을 컴파일하는데 사용되지만, 동일한 [px4io/px4-dev-base](https://hub.docker.com/r/px4io/px4-dev-base/) docker 이미지를 사용해서 Travis-ci로 백업하는 역할을 수행합니다. Travis-ci로 펌웨어를 컴파일하는 일 외에, 세마포어는 stm32discovery, crazyflie를 빌드하며 단위 테스트와 코드 스타일을 검증하는 일을 수행합니다.

## [CircleCI](https://circleci.com/gh/PX4/Firmware)

CircleCI는 docker 이미지 [px4io/px4-dev-nuttx-gcc_next](https://hub.docker.com/r/px4io/px4-dev-nuttx-gcc_next/)를 이용해서 릴리즈할 펌웨어 버전을 위해 사용할 GCC의 차기 버전을 테스트하고 있습니다. px4fmu-v4_default와 posix_sitl_default를 컴파일에 makefile target quick_check를 사용하며 테스팅 수행 및 코드 스타일 검증을 수행합니다.
