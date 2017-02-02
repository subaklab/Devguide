# System Startup

PX4 부팅은 [ROMFS/px4fmu_common/init.d](https://github.com/PX4/Firmware/tree/master/ROMFS/px4fmu_common/init.d) 폴더에 있는 쉘 스크립트가 제어하게 됩니다.

숫자와 밑줄로 시작하는 모든 파일은 airframe 설정과 관련되어 있습니다.(예 `10000_airplane`) 빌드할때 `airframes.xml`로 export되며 이 파일은 [QGroundControl](http://qgroundcontrol.com)에서 파싱되어 airframe 선택하는 UI에서 사용됩니다. 새로운 설정을 추가하는 방법은 [여기](airframes-adding-a-new-frame.md)를 참고하세요.

나머지 파일들은 일반적인 startup 로직의 일부로 사용되며 처음 실행되는 파일은 [rcS](https://github.com/PX4/Firmware/blob/master/ROMFS/px4fmu_common/init.d/rcS) 스크립트이며 이 스크립트에서 다른 스크립트를 호출하게 됩니다.

## 시스템 부팅 디버깅하기 (Debugging the System Boot)

소프트웨어 컴포넌트 드라이버 중에 하나라도 실패하면 부팅이 실패하게 됩니다.

<aside class="tip">
불완전하게 부팅되는 경우 ground control station에서 파라미터가 제대로 설정되지 않게 됩니다. 이는 실행되지 않은 application에서 파라미터를 초기화하지 않았기 때문입니다.
</aside>

부팅 순서를 디버깅하는 올바른 방법은 [system console](advanced-system-console.md)에 연결하여 보드에 전원이 들어오는 과정을 살펴봐야 합니다. 부팅 로그를 보면 부팅 순서에 대해서 상세한 정보를 얻을 수 있기에 왜 부팅에 문제가 있었는지 힌트를 얻을 수 있습니다.

### 일반적인 부팅 실패 원인

  * 필요한 센서의 구동 실패
  * 커스텀 application의 경우 : 시스템의 RAM 메모리가 부족한 경우. `free` 명령을 통해 남은 RAM 메모리 공간을 확인.
  * 소프트웨어 문제나 assertion의 경우 stack trace로 확인 가능

## 시스템 Startup 바꿔치기 (Replacing the System Startup)

일반적으로 기본 부팅에서 바꾸는 것이 좋은 방법입니다. 이에 관해서는 아래 문서를 참고하세요. 만약 완전히 부팅을 바꿔치기하고자 한다면, `/fs/microsd/etc/rc.txt`에 파일을 생성합니다. 이 파일은 마이크로SD 카드의 `etc` 폴더에 위치하고 있습니다. 이 파일이 존재하는 경우 시스템에서는 자동으로 시작되던 것들이 더이상 동작하지 않게 됩니다.

## 시스템 Startup 수정하기 (Customizing the System Startup)

시스템 startup을 수정하는 가장 좋은 방법은 [new airframe configuration](airframes-adding-a-new-frame.md)을 참고합니다. 몇 가지 수정을 하고 싶다면(1개 이상 application을 시작시킨다던가 다른 mixer를 사용하는 경우) 해당 startup에서 수정해서 사용할 수 있습니다.

<aside class="caution">
시스템 부팅 파일은 UNIX 포맷의 파일로 각 라인의 끝에는 UNIX LINED ENDING이 들어가야 합니다. 만약 윈도우에서 수정하는 경우 이를 지원하는 편집기를 이용하도록 합니다.
</aside>

3가지 주요 수정지점이 있습니다. microSD 카드의 루트 폴더는 `/fs/microsd`로 나타냅니다.

  * /fs/microsd/etc/config.txt
  * /fs/microsd/etc/extras.txt
  * /fs/microsd/mixers/NAME_OF_MIXER

### 설정 변경하기 (Customizing the Configuration) (config.txt)

`config.txt`파일은 main 시스템이 구성을 마치고 난 후에 로드됩니다. 그리고 부팅이 완료되기 *전* 으로 쉘 변수 수정이 가능합니다.

### 추가 application 구동시키기 (Starting additional applications)

`extras.txt`는 main 시스템이 부팅되고 나서 추가로 application을 구동시키는데 사용할 수 있습니다. 일반적으로 여기에 해당되는 것은 payload controller나 유사한 선택가능한 커스텀 컴포넌트들입니다.

### 수정한 mixer 구동시키기 (Starting a custom mixer)

시스템이 `/etc/mixers`에서 mixer를 로드하는 것이 기본입니다. 만약 동일한 이름의 파일이 `/fs/microsd/etc/mixers`에 존재한다면, 이 파일이 대신 로드될지도 모릅니다. 이렇게 하면 펌웨어를 재컴파일하지 않고 mixer 파일 수정이 가능합니다.
